| Header One     | Header Two     |
| :------------- | :------------- |
| `web.xml`      | 对Web应用中一些初始信息进行配置       |
| `struts.properties` | 管理struts2框架中定义的大量常量       |
| `struts.xml` | 配置Action和HTTP 请求的对应关系，以及配置逻辑视图和物理视图资源的对应关系 配置常量（开发时常用） 导入其它配置文件       |

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
 ## 4. `listener`
 ```
 <listener>
 	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
 ```
 ## 5. `session-config`
 ```

 ```

 ## 6. `init-param`
 ```

 ```
 ## 7. `session-config`
 ```

 ```

 ## 8. `HTTP 状态码`
 ```

 ```
--------------

# 二、 `struts.properties`

 ```
 struts.i18n.encoding=UTF-8
 struts.devMode=true
 struts.locale=en_US
 ```
 ## 1. 关键元素

---

# 三、 `struts.xml`

 ## 1. `struts.xml`关键元素分析
 + `package`
 ```
 <package name="A" extends="struts-default">
 	<action class="com.action.LoginAction">
		<result name="login_success">/login_success.jsp</result>
 	</action>
 </package>

 <package name="A" extends="struts-default">
 	<action class="com.action.LoginAction">
		<result name="login_success">/login_success.jsp</result>
 	</action>
 </package>

 <package name="A" extends="struts-default">
 	<action class="com.action.LoginAction">
		<result name="login_success">/login_success.jsp</result>
 	</action>
 </package>
 ```
 + `action`

 ```convert```????

 + `result`
 	- `redirect`结果类型的两种应用方式
	 	- 直接在`result`标签后添加`Action`名称
	 	- 运用`<param>`标签的方式俩指定`Action`
 + `include`
 + `global-results`
 + `default-action-ref`


 ## 2. `Action`的动态调用（DMI-Dynamic Method Invocation）
 >`Action`对象名称和方法之间用“!”隔开

 ```
 /user!change.action
 ```
 ## 3. `通配符`
 | 通配符 | 说明    |
 | :-------------: | :------------- |
 | `*`       | 匹配0个或多个字符除了“/”      |
 | `**`       | 匹配0个或多个字符包含“/”      |
 | `\character`       |      |
 ```
 <action name="*_*" class="com.coderdream.{2}Action" method="{1}">
  <result name="success">/{0}Suc.jsp</result>
 </action>
 ```

 常用方式
 ```
 <action>
  <result>{0}.jsp</result>
 </action>
 ```
 >避免用户直接访问系统的JSP 页面，而必须通过Struts2框架来处理请求

 ## 4. `常量配置`
 `Struts2`框架加载常量的顺序如下：
 + `struts-default.xml`
 + `struts-plugin.xml`
 + `struts.xml`
 + `properties.xml`
 + `web.xml`

-------------

# 三、`Action`类文件
```
public interface Action
{
  public static final String  ERROR= "error";
  public static final String  SUCCESS= "success";
  public static final String  INPUT= "input";
  public static final String  LOGIN= "login";
  public static final String  NONE= "now";
  public String execute() throws Exception;
}
```
 ## 1. `ActionSupport`
  >自定义的`Action`类继承了`ActionSupport`类，因此必须定义一个变量`serialVersionUID`

  ```
  private static final long serialVersionUID=1L;
  ```
  ## 2. `Servlet API`
  + 通过 `ActionContext`类访问
  | Header One     | Header Two     |
  | :------------- | :------------- |
  | void put(String key, Object value)       | Item Two       |
  | Map getApplication()      | Item Two       |
  | Map getSession       | Item Two       |


  + 通过特定接口访问(直接访问实例)
  | Header One     | Header Two     |
  | :------------- | :------------- |
  | `ServletRequestAware`    | Item Two       |
  | `ServletResponsetAware`    | Item Two       |
  | `ServletContextAware`    | Item Two       |

  + 通过`ServletActionContext`访问
  | Header One     | Header Two     |
  | :------------- | :------------- |
  | `HttpServletRequest`       | Item Two       |
  | `HttpServletResponse`       | Item Two       |
  | `ServletContext`       | Item Two       |

## 3. `ModelDriven`接口

  ```
  public class LoginAction extends ActionSupport implements ModelDriven<XxxModel>
  {
  	private static final long serialVersionUID = 1L;
  	// 创建XxxModel实例
  	private XxxModel x = new XxxModel();

  	// getter方法，必须实现
  	public XxxModel getModel()
  	{
  		// TODO Auto-generated method stub
  		return x;
  	}

  	// 重载execute方法
  	public String execute() throws Exception
  	{
  		return SUCCESS;
  	}
  }

  ```

  ## 4. 异常处理
  + `全局异常配置`
  + `局部异常配置`

  输出异常信息
  + 输出异常对象信息
  ```
<s:property value="exception.message"/>
  ```
  + 输出异常堆栈信息
  ```
  <s:property value="exceptionStack"/>
  ```

  ```
  <struts>  
    <package name="default" namespace="/" extends="struts-default">
        <global-results>
                       <result name="Exception">/Exception.jsp</result>
                       <result name="SQLException">/SQLException.jsp</result>
        </global-results>
        <global-exception-mappings>
            <exception-mapping exception="java.sql.SQLException" result="SQLException"/>
            <exception-mapping exception="java.lang.Exception" result="Exception"/>
        </global-exception-mappings>      
        <action name="user" class="com.action.UserAction">
            <exception-mapping exception="com.action.SecurityException" result="login"/>        
		   <result name="login" >/loginException.jsp </result>
            <result>/success.jsp</result>
        </action>    
    </package>    
</struts>
  ```
