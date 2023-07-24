# UDP 日志发送

### 日志机端：

* crontab

  ```bash
  */1 * * * * /bin/bash /home/work/wcs_analyzer_log/start.sh >> /home/work/wcs_analyzer_log/crontab.log 2>&1
  0 12 * * * /bin/bash /home/work/wcs_analyzer_log/cleanExpirationLog.sh >> /home/work/wcs_analyzer_log/crontab.log 2>&1
  */1 * * * * /bin/bash /home/work/wcs_analyzer_log/cleanlog.sh >> /home/work/wcs_analyzer_log/crontab.log 2>&1
  ```

* Start.sh

  ```bash
  #!/bin/bash
  WORKDIR=`cd $(dirname $0);pwd -P`
  cd ${WORKDIR}
  BUILDERJAR=${WORKDIR}/UDPServer.jar
  ret=$(ps xuf | grep "${WORKDIR}/UDPServer.jar 15000" | grep -v grep | awk '{print $2}' )
  
  if [ "${ret}" = "" ];then
      source /etc/profile
      java -jar $BUILDERJAR 15000 utf-8 utf-8 &
  fi
  ```

* cleanExpirationLog.sh

  ```bash
  #!/bin/sh
  
  find  /home/work/wcs_analyzer_log/ -name "out.*" -mtime +3 | xargs -n1 -i bash -c "echo '[`date +"%Y-%m-%d %H:%M.%S"`] delete log :{}' >> /home/work/wcs_analyzer_log/crontab.log 2>&1; > {}; rm -rf {}"
  ```

* cleanlog.sh

  ```bash
  #!/bin/sh
  
  p=$(df /data03 | awk '{print $5}' | grep -Eo '[0-9]+')
  
  if [[ $p -gt 85 ]]; then
      for line in $(find /home/work/wcs_analyzer_log/ -type f -size +5G); do
          echo "[`date +"%Y-%m-%d %H:%M.%S"`] clean log file :$line"
          > $line
      done
  fi
  ```

* UDPServer.jar

  文件在当前文件夹中。

### 日志发送端：

* maven依赖

  ```xml
  <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-ip</artifactId>
      <version>${spring-integration-ip.version}</version>
  </dependency>
  ```

* UdpServiceClient

  ```java
  import exception.UdpSendException;
  import lombok.AccessLevel;
  import lombok.EqualsAndHashCode;
  import lombok.Getter;
  import lombok.NoArgsConstructor;
  import lombok.extern.slf4j.Slf4j;
  import org.springframework.integration.ip.udp.UnicastSendingMessageHandler;
  import org.springframework.integration.support.MessageBuilder;
  
  import java.util.ArrayList;
  import java.util.HashMap;
  import java.util.List;
  import java.util.Map;
  import java.util.concurrent.LinkedBlockingQueue;
  
  /**
   * @author baijiahao02 on 2023/4/24
   */
  @Slf4j
  public class UdpServiceClient {
  
      private static final String UDP_SERVER_HOST_MODEL = "host[^weight]:port,host[^weight]:port ····";
  
      @Getter
      private boolean running = false;
  
      private List<UdpServer> udpServerLists = null;
      private Map<UdpServer, LinkedBlockingQueue<String>> queueMapping = null;
  
      /**
       * 当前位置
       */
      private int pos = 0;
  
      /**
       * 初始化
       *
       * @param udpServerConfigStr udp服务器配置str
       * @param capacity           udp缓存区容量
       * @throws UdpSendException udp发送异常
       */
      public static UdpServiceClient init(String udpServerConfigStr, int capacity) {
          UdpServiceClient udpServiceClient = new UdpServiceClient();
  
          Map<UdpServer, Integer> udpServerConfigMap = parseUdpServerConfigStr(udpServerConfigStr, udpServiceClient);
  
          udpServiceClient.udpServerLists = new ArrayList<>();
          udpServiceClient.queueMapping = new HashMap<>(udpServerConfigMap.keySet().size());
          Map<UdpServer, Thread> demonThreadMapping = new HashMap<>(udpServerConfigMap.keySet().size());
  
          udpServerConfigMap.forEach((udpServer, weight) -> {
              udpServiceClient.queueMapping.put(udpServer, new LinkedBlockingQueue<>(capacity));
  
              Thread demonThread = new Thread(new UdpSendTask(udpServer, udpServiceClient), "AsyncUdpSend_daemonThread_" + udpServer);
              demonThread.setDaemon(true);
              demonThreadMapping.put(udpServer, demonThread);
  
              for (int i = 0; i < weight; i++) {
                  udpServiceClient.udpServerLists.add(udpServer);
              }
          });
  
          demonThreadMapping.forEach((udpServer, thread) -> thread.start());
          udpServiceClient.running = true;
  
          log.info("AsyncUdpSend.init 已初始化完成，capacity = " + capacity);
          return udpServiceClient;
      }
  
      /**
       * 将消息添加到发送队列
       *
       * @param msg 消息
       */
      public void offer(String msg) {
  
          if (!running) {
              log.error("UDP server is not running, please initialize it first.");
              return;
          }
  
          UdpServer selectedUdpServer;
  
          synchronized (this) {
              if(pos >= udpServerLists.size()){
                  pos = 0;
              }
              selectedUdpServer = udpServerLists.get(pos);
              pos++;
          }
  
          try {
              LinkedBlockingQueue<String> queue = queueMapping.get(selectedUdpServer);
              if (null == queue) {
                  log.error("QueueManager(" + selectedUdpServer + ") 尚未初始化");
                  return;
              }
              if (!queue.offer(msg)) {
                  log.error("AsyncUdpSend: queue is full,discard the msg:" + msg);
              }
          } catch (Exception e) {
              log.error(e.getMessage(), e);
          }
      }
  
      /**
       * 获得队列
       *
       * @param udpServer udp服务器
       * @return {@link LinkedBlockingQueue}<{@link String}>
       */
      public LinkedBlockingQueue<String> getQueue(UdpServer udpServer) {
          return queueMapping.get(udpServer);
      }
  
      /**
       * 解析udp服务器配置str
       *
       * @param udpServerConfigStr udp服务器配置str
       * @return {@link Map}<{@link UdpServer}, {@link Integer}>
       * @throws UdpSendException udp发送异常
       */
      private static Map<UdpServer, Integer> parseUdpServerConfigStr(String udpServerConfigStr, UdpServiceClient udpServiceClient) throws UdpSendException {
          final String[] udpServerHostArr = udpServerConfigStr.split(",");
          Map<UdpServer, Integer> udpServerWeightMap = new HashMap<>(udpServerHostArr.length);
          for (String udpServerHostStr : udpServerHostArr) {
              udpServerHostStr = udpServerHostStr.trim();
              if (udpServerHostStr.isEmpty()) {
                  continue;
              }
              final String[] udpServerConfigArr = udpServerHostStr.split(":");
              if (udpServerConfigArr.length != 2) {
                  udpServiceClient.running = false;
                  throw new UdpSendException("udpServerConfigStr(" + udpServerConfigStr + ") set error. eg: " + UDP_SERVER_HOST_MODEL);
              }
  
              final String[] udpServerHostWeight = udpServerConfigArr[0].split("\\^");
              if (udpServerHostWeight.length > 2) {
                  udpServiceClient.running = false;
                  throw new UdpSendException("udpServerConfigStr(" + udpServerConfigStr + ") set error. eg: " + UDP_SERVER_HOST_MODEL);
              }
              final UdpServer udpServer = new UdpServer(udpServerHostWeight[0], Integer.parseInt(udpServerConfigArr[1]));
              if (udpServerHostWeight.length > 1) {
                  udpServerWeightMap.put(udpServer, Integer.parseInt(udpServerHostWeight[1]));
              }else {
                  udpServerWeightMap.put(udpServer, 1);
              }
          }
          return udpServerWeightMap;
      }
  
      /**
       * udp发送任务
       *
       * @author baijiahao
       * @date 2023/02/16
       */
      @Slf4j
      @NoArgsConstructor(access = AccessLevel.PRIVATE)
      private static class UdpSendTask implements Runnable {
  
          private UdpServer udpServer;
          private UnicastSendingMessageHandler unicastSendingMessageHandler;
          private UdpServiceClient udpServiceClient;
  
          /**
           * udp客户端初始化
           */
          public UdpSendTask(UdpServer udpServer, UdpServiceClient udpServiceClient) {
              this.udpServer = udpServer;
              this.unicastSendingMessageHandler= new UnicastSendingMessageHandler(udpServer.getUdpServerHost(), udpServer.getPort());
              this.udpServiceClient = udpServiceClient;
              log.info("UnicastSendingMessageHandler init finish. ip:{} port:{}", this.udpServer.getUdpServerHost(), this.udpServer.getPort());
          }
  
          @Override
          public void run() {
              while (true) {
                  if (this.unicastSendingMessageHandler == null) {
                      log.error("UnicastSendingMessageHandler 尚未初始化。");
                      return;
                  }
  
                  if (udpServiceClient.getQueue(this.udpServer) == null) {
                      try {
                          Thread.sleep(10);
                      } catch (Exception e) {
                          log.error("Thread.sleep exec failed.");
                      }
                      continue;
                  }
  
                  try {
                      unicastSendingMessageHandler.handleMessage(MessageBuilder.withPayload(udpServiceClient.getQueue(this.udpServer).take()).build());
                  } catch (Exception e) {
                      log.error(e.getMessage(), e);
                  }
              }
          }
      }
  
      /**
       * udp服务器
       *
       * @author baijiahao
       * @date 2023/02/16
       */
      @NoArgsConstructor(access = AccessLevel.PRIVATE)
      @EqualsAndHashCode
      @Getter
      private static class UdpServer {
          /**
           * udp服务器主机IP
           */
          private String udpServerHost;
          /**
           * udp端口
           */
          private int port;
  
          public UdpServer(String udpServerHost, int port) {
              this.udpServerHost = udpServerHost;
              this.port = port;
          }
  
          @Override
          public String toString() {
              return this.udpServerHost + ":" + port;
          }
      }
  }
  ```
  
* AsyncUdpSendUtil

  ```java
  import lombok.AccessLevel;
  import lombok.NoArgsConstructor;
  import lombok.extern.slf4j.Slf4j;
  
  import java.net.InetAddress;
  import java.net.UnknownHostException;
  import java.util.*;
  
  /**
   * @author baijiahao02 on 2023/2/15
   */
  @Slf4j
  @NoArgsConstructor(access = AccessLevel.PRIVATE)
  public class AsyncUdpSendUtil {
  
      private static final String WCS_LOG_DNS_NAME = "wcslog.58dns.org";
  
      private static final String DEFAULT_UDP_NAME = "_default";
  
      private static final Integer DEFAULT_CAPACITY = 1000;
  
      private static final HashMap<String, UdpServiceClient> UDP_SERVER_LISTS = new HashMap<>(5);
  
      /**
       * init wcs日志dns
       *
       * @param udpPort udp端口
       * @throws UdpSendException udp发送异常
       */
      public static synchronized void initByWcsLogDns(int udpPort) throws UdpSendException {
          initByDns(WCS_LOG_DNS_NAME, udpPort, DEFAULT_CAPACITY);
      }
      public static synchronized void initByWcsLogDns(int udpPort, String udpName) throws UdpSendException {
          initByDns(WCS_LOG_DNS_NAME, udpPort, DEFAULT_CAPACITY, udpName);
      }
  
      /**
       * init wcs日志dns
       *
       * @param udpPort  udp端口
       * @param capacity 能力
       * @throws UdpSendException udp发送异常
       */
      public static synchronized void initByWcsLogDns(int udpPort, int capacity) throws UdpSendException {
          initByDns(WCS_LOG_DNS_NAME, udpPort, capacity);
      }
  
      public static synchronized void initByWcsLogDns(int udpPort, int capacity, String udpName) throws UdpSendException {
          initByDns(WCS_LOG_DNS_NAME, udpPort, capacity, udpName);
      }
  
      /**
       * initdns
       *
       * @param dnsName dns名称
       * @param udpPort udp端口
       * @throws UdpSendException udp发送异常
       */
      public static synchronized void initByDns(String dnsName, int udpPort) throws UdpSendException {
          initByDns(dnsName, udpPort, DEFAULT_CAPACITY);
      }
  
      public static synchronized void initByDns(String dnsName, int udpPort, String udpName) throws UdpSendException {
          initByDns(dnsName, udpPort, DEFAULT_CAPACITY, udpName);
      }
  
      /**
       * initdns
       *
       * @param dnsName  dns名称
       * @param udpPort  udp端口
       * @param capacity 能力
       * @throws UdpSendException udp发送异常
       */
      public static synchronized void initByDns(String dnsName, int udpPort, int capacity) throws UdpSendException {
          initByDns(dnsName, udpPort, capacity, DEFAULT_UDP_NAME);
      }
      public static synchronized void initByDns(String dnsName, int udpPort, int capacity, String udpName) throws UdpSendException {
          final InetAddress[] dnsNodeIpArr;
          try {
              dnsNodeIpArr = InetAddress.getAllByName(dnsName);
          } catch (UnknownHostException e) {
              log.error("dnsName(" + dnsName + ") is not a valid DNS address.");
              throw new UdpSendException(e);
          }
  
          if (dnsNodeIpArr.length == 0) {
              log.error("The number of dnsNodeIp(" + dnsNodeIpArr.length + ") is illegal value.");
          }
  
          StringBuilder udpServerConfigStrBuilder = new StringBuilder();
  
          for (InetAddress inetAddress : dnsNodeIpArr) {
              udpServerConfigStrBuilder.append(inetAddress.getHostAddress()).append(":").append(udpPort).append(",");
          }
  
          udpServerConfigStrBuilder.deleteCharAt(udpServerConfigStrBuilder.length() - 1);
  
          init(udpServerConfigStrBuilder.toString(), capacity, udpName);
      }
  
      /**
       * 初始化
       *
       * @param udpServerConfigStr udp服务器配置str
       * @throws UdpSendException udp发送异常
       */
      public static synchronized void init(String udpServerConfigStr) throws UdpSendException {
          init(udpServerConfigStr, DEFAULT_CAPACITY);
      }
  
      public static synchronized void init(String udpServerConfigStr, String udpName) throws UdpSendException {
          init(udpServerConfigStr, DEFAULT_CAPACITY, udpName);
      }
  
      /**
       * 初始化
       *
       * @param udpServerConfigStr udp服务器配置str
       * @param capacity           udp缓存区容量
       * @throws UdpSendException udp发送异常
       */
      public static synchronized void init(String udpServerConfigStr, int capacity) throws UdpSendException {
          init(udpServerConfigStr, capacity, DEFAULT_UDP_NAME);
      }
  
      public static synchronized void init(String udpServerConfigStr, int capacity, String udpName) throws UdpSendException {
          if (UDP_SERVER_LISTS.containsKey(udpName) && UDP_SERVER_LISTS.get(udpName) != null && UDP_SERVER_LISTS.get(udpName).isRunning()) {
              throw new UdpSendException("'" + udpName + "' UDP server is running. Do not repeat initialization.");
          }
          UDP_SERVER_LISTS.put(udpName, UdpServiceClient.init(udpServerConfigStr, capacity));
      }
  
      /**
       * 将消息添加到发送队列
       *
       * @param msg 消息
       */
      public static void offer(String msg) {
          offer(msg, DEFAULT_UDP_NAME);
      }
  
      public static void offer(String msg, String udpName) {
          if (UDP_SERVER_LISTS.containsKey(udpName) && UDP_SERVER_LISTS.get(udpName) != null) {
              UDP_SERVER_LISTS.get(udpName).offer(msg);
              return;
          }
          log.error("'" + udpName + "' UDP server is not exist. Please first initialized.");
      }
  
  }
  ```