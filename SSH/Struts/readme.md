# 一、`web.xml`
- `过滤器`
- `<session-config>会话时间`
- `欢迎页`
- `错误页`
- `监听器`
- `控制器`

 ## 1. `welcome-file-list` & `welcome-file`
 ## 2. `filter` & `filter-mapping`
	```

	<filter>
		<filter-name>struts2</filter-name><!--指定Filter的名字，不能为空-->
		<filter-class>org.apache.struts2.dispatcher.FilterDispatcher</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>struts2</filter-name><!--Filter 的名字，该名字必须是filter元素中已声明过的过滤器名字-->
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```
 ## 3. `error-page`
 ```
 <error-page>
 	<error-code>404</error-code>
 	<location>/error.jsp</location>
 </error-page>
 ```
