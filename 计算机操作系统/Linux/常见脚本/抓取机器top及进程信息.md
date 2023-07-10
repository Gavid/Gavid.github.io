## 抓取机器top及进程信息

* watchps.sh

  ```bash
  #!/bin/sh
  for((i=0;$i<=600;i++))
    do
     echo `date '+%Y-%m-%d %H:%M:%S'` >> /opt/log/ps.log
     ps -ef >> /opt/log/ps.log
     sleep 1s
  
    done
  echo
  ```

* watchtop.sh

  ```bash
  #!/bin/sh
  for ((i=0; $i<=600; i++))
  do
      echo `date '+%Y-%m-%d %H:%M:%S'` >> /opt/log/top.log
      top -b -n 1 -c | head -n 31 >> /opt/log/top.log
      sleep 1
  done
  ```

  