文章转自[VUE+Axios请求加安全级别（加密、签名、时间戳）与java服务端解析](https://my.oschina.net/Lady/blog/1814825)

@[TOC]
# 前端
1. 封装加密工具,这个是引用了crypto.js，通过npm安装
```
npm i --save crypto-js
```
2. 通过crypto-js实现 加解密工具
```javascript
/**
* 通过crypto-js实现 加解密工具
* AES、HASH(MD5、SHA256)、base64
*/
import CryptoJS from 'crypto-js';
const KP = {
  key: '1234567812345678', // 秘钥 16*n:
  iv: '1234567812345678'  // 偏移量
};
function getAesString(data, key, iv) { // 加密
    key = CryptoJS.enc.Utf8.parse(key);
    // alert(key）;
    iv = CryptoJS.enc.Utf8.parse(iv);
    let encrypted = CryptoJS.AES.encrypt(data, key,
        {
            iv,
            mode: CryptoJS.mode.CBC,
            padding: CryptoJS.pad.Pkcs7
        });
    return encrypted.toString();    // 返回的是base64格式的密文
}
function getDAesString(encrypted, key, iv) { // 解密
    key = CryptoJS.enc.Utf8.parse(key);
    iv = CryptoJS.enc.Utf8.parse(iv);
    let decrypted = CryptoJS.AES.decrypt(encrypted, key,
        {
            iv,
            mode: CryptoJS.mode.CBC,
            padding: CryptoJS.pad.Pkcs7
        });
    return decrypted.toString(CryptoJS.enc.Utf8);      //
}
// AES 对称秘钥加密
const aes = {
  en: (data) => getAesString(data, KP.key, KP.iv),
  de: (data) => getDAesString(data, KP.key, KP.iv)
};
// BASE64
const base64 = {
    en: (data) => CryptoJS.enc.Base64.stringify(CryptoJS.enc.Utf8.parse(data)),
    de: (data) => CryptoJS.enc.Base64.parse(data).toString(CryptoJS.enc.Utf8)
};
// SHA256
const sha256 = (data) => {
    return CryptoJS.SHA256(data).toString();
};
// MD5
const md5 = (data) => {
  return CryptoJS.MD5(data).toString();
};

/**
* 签名
* @param token 身份令牌
* @param timestamp 签名时间戳
* @param data 签名数据
*/
const sign = (token, timestamp, data) => {
  // 签名格式： timestamp + token + data(字典升序)
  let ret = [];
  for (let it in data) {
    let val = data[it];
    if (typeof val === 'object' && //
        (!(val instanceof Array) || (val.length > 0 && (typeof val[0] === 'object')))) {
      val = JSON.stringify(val);
    }
    ret.push(it + val);
  }
  // 字典升序
  ret.sort();
  let signsrc = timestamp + token + ret.join('');
  return md5(signsrc);
};
export {
  aes,
  md5,
  sha256,
  base64,
  sign
};
```
3. 封装axios，对外提供统一调用口。
```javascript
/**
* axios.js提供request请求封装
* 包括 get、post、delete、put等方式
*/
import axios from 'axios';
import store from '@/vuex/store';
import {aes, sign} from '@/common/js/crypto';
import { Message } from 'element-ui';

const ajax = axios.create({
  baseURL: store.getters.serviceHost, // url前缀
  timeout: 10000,                     // 超时毫秒数
  withCredentials: true               // 携带认证信息cookie
});

/**
* get方式请求，url传参
* @param url 请求url
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const get = (url, params, level) => ajax(getConfig(url, 'get', true, params, level)).then(res => successback(res)).catch(error => errback(error));
/**
* post方式请求 json方式传参
* @param url 请求url
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const postJson = (url, params, level) => ajax(getConfig(url, 'post', true, params, level)).then(res => successback(res)).catch(error => errback(error));
/**
* post方式请求 表单传参
* @param url 请求url
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const post = (url, params, level) => ajax(getConfig(url, 'post', false, params, level)).then(res => successback(res)).catch(error => errback(error));
/**
* delete方式请求 url传参
* @param url 请求url
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const del = (url, params, level) => ajax(getConfig(url, 'delete', true, params, level)).then(res => successback(res)).catch(error => errback(error));
/**
* put方式请求 json传参
* @param url 请求url
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const putJson = (url, params, level) => ajax(getConfig(url, 'put', true, params, level)).then(res => successback(res)).catch(error => errback(error));
/**
* put方式请求 表单传参
* @param url 请求url
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const put = (url, params, level) => ajax(getConfig(url, 'put', false, params, level)).then(res => successback(res)).catch(error => errback(error));


// 参数转换
const param2String = data => {
  console.log('data', data);
  if (typeof data === 'string') {
    return data;
  }
  let ret = '';
  for (let it in data) {
    let val = data[it];
    if (typeof val === 'object' && //
        (!(val instanceof Array) || (val.length > 0 && (typeof val[0] === 'object')))) {
      val = JSON.stringify(val);
    }
    ret += it + '=' + encodeURIComponent(val) + '&';
  }
  if (ret.length > 0) {
    ret = ret.substring(0, ret.length - 1);
  }
  return ret;
};

// 错误回调函数
const errback = error => {
  if ('code' in error) {
    // 未登录
    if (error.code === 30001) {
      sessionStorage.clear();
      window.location.href = '/';
      return;
    }
    return Promise.reject(error);
  }
  // 网络异常,或链接超时
  Message({
    message: error.message,
    type: 'error'
  });
  return Promise.reject({data: error.message});
};
// 成功回调函数
const successback = res => {
  if (res.status === 200 && res.data.code !== 20000) {
      let errMsg = {'30002': '对不起无权限', '30003': '验签失败'};
      let msg = errMsg[res.data.code];
      if (msg) {
        Message({
          message: errMsg[res.data.code],
          type: 'error'
        });
      }
      return Promise.reject(res.data);
  }
  return res.data;
};

/**
* @param url 请求url
* @param method get,post,put,delete
* @param isjson 是否json提交参数
* @param params 参数
* @param level 0:无加密，1：参数加密，2: 签名+时间戳； 默认0
*/
const getConfig = (url, method, isjson, params, level = 0) => {
  let config_ = {
    url,
    method,
    // params, data,
    headers: {
      level
    }
  };
  // 时间戳
  if (level === 1) { // 加密
      params = {encrypt: aes.en(JSON.stringify(params))};
  } else if (level === 2) { // 签名
    let timestamp = new Date().getTime();
    // 获取token
    let token = store.state.token;
    if (!token) {
        token = JSON.parse(sessionStorage.getItem('user')).token;
        store.state.token = token;
    }
    // 签名串
    let signstr = sign(token, timestamp, params);
    console.log('token', token);
    console.log('signstr', signstr);
    config_.headers = {
       level,
       timestamp,
       signstr
    };
  }
   // 表单提交参数
  if (!isjson) {
    config_.headers['Content-Type'] = 'application/x-www-form-urlencoded';
    config_.responseType = 'text';
    config_.transformRequest = [function (data) {
      return param2String(data);
   }];
  }
  // 设置参数
  if (method in {'get': true, 'delete': true}) {
    config_.params = params;
  } else if (method in {'post': true, 'put': true}) {
    config_.data = params;
  }
  return config_;
};

// 统一方法输出口
export {
  ajax,
  get,
  postJson,
  post,
  del,
  putJson,
  put
};
```
# 后端
1. 添加注解@CryptoType，需求是哪些controller类或方法需要加密或者签名加上对应注解即可实现。
```java
package com.nja.baseweb.crypto;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 秘密类型
 *
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CryptoType {

	Type value() default Type.NONE;
	
	enum Type {
		/**
		 * 无
		 */
		NONE,
		/**
		 * 签名
		 */
		SIGN, 
		/**
		 * 加密
		 */
		CRYPTO
	}
}
```
2. 添加签名验签拦截器，处理带签名请求方法
```java
package com.nja.baseweb.crypto;

import java.io.IOException;
import java.net.URLDecoder;
import java.util.Enumeration;
import java.util.Map;
import java.util.TreeMap;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import com.alibaba.fastjson.JSON;
import com.nja.base.common.bean.Result;
import com.nja.baseweb.crypto.CryptoType.Type;

import liquibase.util.MD5Util;

/**
 * 请求签名验证拦截器 <br>
 * 加签名验签的方法，需要在方法或者类上添加注解@CryptoType(CryptoType.SIGN)<br>
 * 签名方式：md5(timestamp + token + data（字典升序）)
 *
 */
public class RequestSignVerifyInterceptor extends HandlerInterceptorAdapter {

	private static Logger logger = LoggerFactory.getLogger(RequestSignVerifyInterceptor.class);

	/** 时间戳超时设置60s*/
	private static final long TIMEOUT = 60 * 1000L;
	
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		logger.debug("RequestSignVerifyInterceptor -> preHandle");
		if (handler instanceof HandlerMethod) {
			HandlerMethod method = (HandlerMethod) handler;
			// 方法上注解
			CryptoType type = method.getMethodAnnotation(CryptoType.class);
			if (null == type) {
				// 类上是否有注解
				type = method.getBeanType().getAnnotation(CryptoType.class);
				if (null == type) {
					return true;
				}
			}
			if (type.value() != Type.SIGN) {
				// 不是签名验证跳过
				return true;
			}
		}
		logger.debug("Start verifySign ->");
		if (!verifySign(request)) {
			// 不通过
			logger.debug("VerifySign fail");
			response.setContentType("application/json; charset=UTF-8");
			response.getWriter().write(JSON.toJSONString(Result.failure(Result.FAILURE_VERIFY_SIGN, "验签失败")));
			return false;
		}
		// 验签通过
		return true;
	}

	/**
	 * 验证签名是否一致
	 * 
	 * @param request
	 */
	private boolean verifySign(HttpServletRequest request) throws IOException {
		String timestamp = request.getHeader("timestamp");
		String signstr = request.getHeader("signstr");
		String level = request.getHeader("level");
		logger.debug("VerifySign params -> timestamp:{}, signstr:{}, level:{}", timestamp, signstr, level);
		if (StringUtils.isEmpty(timestamp) || StringUtils.isEmpty(signstr)) {
			logger.error("VerifySign head param timestamp or signstr is empty");
			return false;
		}
		// 时间戳是否过期 （60s）
		if (System.currentTimeMillis() - Long.parseLong(timestamp) > TIMEOUT) {
			// 超时
			logger.error("VerifySign timestamp timeout");
			return false;
		}
		// FIXME 从session获取id 作token，session需要做redis共享，不然集群部署每个服务获取sessionID不一致
		String token = request.getSession().getId();
		TreeMap<String, String> map = new TreeMap<>();
		Enumeration<String> names = request.getParameterNames();
		while (names.hasMoreElements()) {
			String name = names.nextElement();
			map.put(name, URLDecoder.decode(request.getParameter(name), "UTF-8"));
		}
		return signstr.equals(sign(token, timestamp, map));
	}

	/**
	 * 签名
	 * 
	 * @param token
	 * @param timestamp
	 * @param params
	 */
	private String sign(String token, String timestamp, TreeMap<String, String> params) {
		StringBuilder paramValues = new StringBuilder();
		paramValues.append(timestamp).append(token);

		for (Map.Entry<String, String> entry : params.entrySet()) {
			paramValues.append(entry.getKey()).append(entry.getValue());
		}
		return MD5Util.computeMD5(paramValues.toString());
	}
}
```
3. 添加解密过滤器，处理前端加密请求方法，重写request参数 
```java
package com.nja.baseweb.crypto;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

import javax.servlet.FilterChain;
import javax.servlet.ReadListener;
import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import javax.servlet.http.HttpServletResponse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactoryUtils;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.core.annotation.AnnotationAwareOrderComparator;
import org.springframework.util.Assert;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerExecutionChain;
import org.springframework.web.servlet.HandlerMapping;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import com.nja.base.common.bean.Result;
import com.nja.baseweb.crypto.CryptoType.Type;

import cn.hutool.crypto.Mode;
import cn.hutool.crypto.Padding;
import cn.hutool.crypto.symmetric.AES;

/**
 * 请求参数解密过滤器<br>
 * 需要解密的方法，在对应方法或者类上添加注解@CryptoType(CryptoType.CRYPTO)<br>
 * 前端加密会把参数通过AES方式加密成base64，“encrypt”： 加密后内容<br>
 * FIXME 当前只做请求参数加密，返回值未做加密
 * 
 * @author ldy
 *
 */
@WebFilter(urlPatterns = { "/*" }, filterName = "requestDecryptFilter")
public class RequestDecryptFilter extends OncePerRequestFilter implements ApplicationContextAware {

	private static Logger logger = LoggerFactory.getLogger(RequestDecryptFilter.class);

	/** 方法映射集 */
	private List<HandlerMapping> handlerMappings;

	/** AES加解密 */
	protected static AES aes = new AES(Mode.CBC, Padding.PKCS5Padding, "1234567812345678".getBytes(),
			"1234567812345678".getBytes());

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		try {
			Object handler = getHandler(request).getHandler();
			if (handler instanceof HandlerMethod) {
				HandlerMethod method = (HandlerMethod) handler;
				// 方法上注解
				CryptoType type = method.getMethodAnnotation(CryptoType.class);
				if (null == type) {
					// 类上是否有注解
					type = method.getBeanType().getAnnotation(CryptoType.class);
					if (null == type) {
						filterChain.doFilter(request, response);
						return;
					}
				}
				if (type.value() != Type.CRYPTO) {
					// 不是解密跳过
					filterChain.doFilter(request, response);
					return;
				}
			}
		} catch (Exception e) {
			logger.error("", e);
			filterChain.doFilter(request, response);
			return;
		}

		try {
			// 调用自定义request解析参数
			filterChain.doFilter(new DecryptRequest(request), response);
		} catch (IOException e) {
			// 异常处理
			logger.debug("Decrypt fail");
			response.setContentType("application/json; charset=UTF-8");
			response.getWriter().write(JSON.toJSONString(Result.failure(Result.FAILURE_VERIFY_SIGN, "验签失败")));
		}
	}

	@Override
	public void setApplicationContext(ApplicationContext context) throws BeansException {
		Map<String, HandlerMapping> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context,
				HandlerMapping.class, true, false);
		if (!matchingBeans.isEmpty()) {
			this.handlerMappings = new ArrayList<>(matchingBeans.values());
			// We keep HandlerMappings in sorted order.
			AnnotationAwareOrderComparator.sort(this.handlerMappings);
		}
	}

	/**
	 * 获取访问目标方法
	 * 
	 * @param request
	 * @return HandlerExecutionChain
	 * @throws Exception
	 */
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping hm : this.handlerMappings) {
				if (logger.isTraceEnabled()) {
					logger.trace("Testing handler map [" + hm + "] in DispatcherServlet with name ''");
				}
				HandlerExecutionChain handler = hm.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}

	/**
	 * 解密request封装
	 * 
	 * @author ldy
	 *
	 */
	private class DecryptRequest extends HttpServletRequestWrapper {

		private static final String APPLICATION_JSON = "application/json";
		/** 所有参数的Map集合 */
		private Map<String, String[]> parameterMap;
		/** 输入流 */
		private InputStream inputStream;

		public DecryptRequest(HttpServletRequest request) throws IOException {
			super(request);
			String contentType = request.getHeader("Content-Type");
			logger.debug("DecryptRequest -> contentType:{}", contentType);
			String encrypt = null;
			if (null != contentType && contentType.contains(APPLICATION_JSON)) {
				// json
				ServletInputStream io = request.getInputStream();
				ByteArrayOutputStream os = new ByteArrayOutputStream();
				byte[] buffer = new byte[1024];
				int length;
				while ((length = io.read(buffer)) != -1) {
					os.write(buffer, 0, length);
				}
				byte[] bytes = os.toByteArray();
				encrypt = (String) JSON.parseObject(new String(bytes)).get("encrypt");
			} else {
				// url
				encrypt = request.getParameter("encrypt");
			}
			logger.debug("DecryptRequest -> encrypt:{}", encrypt);
			// 解密
			String params = decrypt(encrypt);

			if (null != contentType && contentType.contains(APPLICATION_JSON)) {
				if (this.inputStream == null) {
					this.inputStream = new DecryptInputStream(new ByteArrayInputStream(params.getBytes()));
				}
			}
			parameterMap = buildParams(params);
		}

		private String decrypt(String encrypt) throws IOException {
			try {
				// 解密
				return aes.decryptStrFromBase64(encrypt);
			} catch (Exception e) {
				logger.error("", e);
				throw new IOException(e.getMessage());
			}
		}

		private Map<String, String[]> buildParams(String src) throws UnsupportedEncodingException {
			Map<String, String[]> map = new HashMap<>();
			Map<String, String> params = JSONObject.parseObject(src, new TypeReference<Map<String, String>>() {
			});
			for (String key : params.keySet()) {
				map.put(key, new String[] { params.get(key) });
			}
			return map;
		}

		@Override
		public String getParameter(String name) {
			String[] values = getParameterMap().get(name);
			if (values != null) {
				return (values.length > 0 ? values[0] : null);
			}
			return super.getParameter(name);
		}

		@Override
		public String[] getParameterValues(String name) {
			String[] values = getParameterMap().get(name);
			if (values != null) {
				return values;
			}
			return super.getParameterValues(name);
		}

		@Override
		public Enumeration<String> getParameterNames() {
			Map<String, String[]> multipartParameters = getParameterMap();
			if (multipartParameters.isEmpty()) {
				return super.getParameterNames();
			}

			Set<String> paramNames = new LinkedHashSet<String>();
			Enumeration<String> paramEnum = super.getParameterNames();
			while (paramEnum.hasMoreElements()) {
				paramNames.add(paramEnum.nextElement());
			}
			paramNames.addAll(multipartParameters.keySet());
			return Collections.enumeration(paramNames);
		}

		@Override
		public Map<String, String[]> getParameterMap() {
			return null == parameterMap ? super.getParameterMap() : parameterMap;
		}

		@Override
		public ServletInputStream getInputStream() throws IOException {
			return this.inputStream == null ? super.getInputStream() : (ServletInputStream) this.inputStream;
		}
	}

	/**
	 * 自定义ServletInputStream
	 * 
	 * @author ldy
	 *
	 */
	private class DecryptInputStream extends ServletInputStream {

		private final InputStream sourceStream;

		/**
		 * Create a DelegatingServletInputStream for the given source stream.
		 * 
		 * @param sourceStream
		 *            the source stream (never {@code null})
		 */
		public DecryptInputStream(InputStream sourceStream) {
			Assert.notNull(sourceStream, "Source InputStream must not be null");
			this.sourceStream = sourceStream;
		}

		@Override
		public int read() throws IOException {
			return this.sourceStream.read();
		}

		@Override
		public void close() throws IOException {
			super.close();
			this.sourceStream.close();
		}

		@Override
		public boolean isFinished() {
			return false;
		}

		@Override
		public boolean isReady() {
			return false;
		}

		@Override
		public void setReadListener(ReadListener readListener) {

		}
	}

}

```
# 使用
1、后端需要签名或者加密的请求方法或类上加上注解

@CryptoType(Type.SIGN)  // 签名

@CryptoType(Type.CRYPTO) // 加密

2、前端在请求方式传参数类型，

post(`url`, params, 2);  // 签名

post('url', params, 1); // 加密
