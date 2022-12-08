# **spring-mvc**

## **Index 目錄**
* [Intro 簡介](#intro-簡介)
* [SpringMVC 工作流程](#springmvc-工作流程)
* [常用 annotation](#常用-annotation)
* [Spring Web MVC 多國語言(i18n)功能](#spring-web-mvc-多國語言i18n功能)

## **Intro 簡介**
Retrieved from 

https://www.readfog.com/a/1634442113787727872

https://juejin.cn/post/6844904031807094798#heading-20

### **什麼是 Spring Web MVC？**
Spring Web MVC 是一種基於 Java 實現 Web MVC 設計模式的請求驅動類型的輕量級 Web 框架，即使用了 MVC 架構模式的思想，將 web 層進行職責解耦，基於請求驅動指的就是使用請求來響應模型，框架的目的就是幫助我們簡化開發。
* Spring Web MVC 功能建構在 Servlet, JSP 規格之上，必須 Web Container 的支援才能運作
* Spring Web MVC 實作 Front Controller Design Pattern，使用 IoC 機制，將 Model, Controller, View 之間的相依關係斷開，讓程式設計師可以隨時修改或是替換 View,  Controller 與 Model
* Spring Web MVC 將 Web 應用程式中連接 View 元件與 Controller 元件的程式碼(View 呼叫 Controller, Controller 導向 View)抽離出來，提供 Spring 的制式化實作，讓程式設計師專注在企業邏輯上

---

## **SpringMVC 工作流程**
下圖是 Spring MVC 從收到 request 到後端伺服器到回傳 response 到 user 的過程：
![spring-mvc](/spring-mvc/spring-mvc.jpg)

* DispatcherServlet: Spring Web MVC 的流程控制中心，處理使用 Spring Web MVC 技術的 HTTP Request，其存在降低了元件之間的耦合性
* HandlerMapping: 處理 HTTP Request與Controller/Handler 元件之間對應的關係(1個 HTTP Request 應該呼叫哪個 Controller 來處理)
* ViewResolver: 處理 View 元件邏輯名稱與 View 元件實際 URL 之間對應關係的設定(讓 Controller 元件可以使用邏輯名稱呼叫 View)，藉由 logical view name 解析出 physical location of view，然後在藉由 Model 的 data 將 View 進行渲染，最後在回應到 Client 端

---

### **DispatcherServlet**
DispatcherServlet 是 Front Controller Pattern 的應用，提供 Spring Web MVC 的集中訪問點，並且負責職責分派，與 Spring IoC 容器相輔相成，主要負責流程控制，主要職責如下：
* 通過 HandlerMapping，將請求映射到 Handler (返回一個HandlerExecutionChain，其中包括一個 Handler 和多個 HandlerInterceptor 攔截器)
* 通過HandlerAdapter支持多種類型的Handler(HandlerExecutionChain的處理器)
* 通過 ViewResolver 解析 logical view name 得到 physical location of view
* 將 model 資料渲染到 view
* 如果執行過程遇到異常，將交給 HandlerExceptionResolver 來處理
* 文件上傳解析，如果請求類型是 multipart，將通過 MultipartResolver 進行文件上傳解析

#### **DispathcherServlet 設定**
* load-on-startup: 啟動容器時初始化該 servlet
* url-pattern: 表示哪些請求交給Spring Web MVC處理，"/"是用來定義預設 servlet 映射的
* contextConfigLocation: SpringMVC 組態文件的路徑
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
 <servlet>
          <servlet-name>Spring MVC</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <!-- 表示啟動容器時初始化該servlet -->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:Spring-servlet.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
     </servlet>
     <servlet-mapping>
          <servlet-name>Spring MVC</servlet-name>
          <url-pattern>/*</url-pattern>
     </servlet-mapping>

</web-app>
```

#### **Spring 和 SpringMVC 容器設定**
實際開發中，Spring 和 SpringMVC 是分開設定的，因此需要添加 Spring 相關組態設定，要將 Controller 以外的 Bean 註冊到 Spring Container 中，並將 Controller 註冊到 SpringMVC Container 中。
* Spring 容器通過 ContextLoaderListener 來加載，加載的 bean 如 Dao 和 Service
* SpringMVC 容器通過 DispatcherServlet 來加載，加載的 bean如Controller, HandlerMapping, HandlerAdapter 等等

程式碼範例：
* Spring 組態設定檔 - applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.javaboy" use-default-filters="true">
    <!--將 @Controller 排除在 Spring container 之外-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>
```
* 由於這個組態設定文件不會被自動加載，所以要在web.xml進行設定
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
* 在 SpringMVC 組態設定文件手動指定 Controller
```xml
<context:component-scan base-package="org.javaboy.helloworld" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
<!--這邊採用的 HandlerMapping 是 BeanNameUrlHandlerMapping，可根據此 bean 名稱查找對應的 Controller/Handler-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" id="handlerMapping">
    <property name="beanName" value="/hello"/>
</bean>
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" id="handlerAdapter"/>
<!--ViewResolver-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="viewResolver">
    <property name="prefix" value="/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```
* Controller
```java
@Controller("/hello")
public class MyController {
    @Autowired
    HelloService helloService;
    /**
     * 這是一個請求處理的api
     * @param req 前端發來的請求
     * @param resp 後端回給前端的回應
     * @return 返回值是 ModelAndView，Model 相當於是數據模型，View 是視圖
     * @throws Exception
     */
    public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        System.out.println(helloService.hello("java"));
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("name", "javaboy");
        return mv;
    }
}
```

---

### **HandlerMapping**
在 HandlerMapping 中，保存了特定的請求 url 應該被哪一個 Handler(也就是通常的 Controller) 所處理。HandlerMapping 根據映射策略的不同，大概有下面幾種映射查找方式：
1. org.springframework.web.servlet.handler.SimpleUrlHandlerMapping 通過配置請求路徑和 Controller 映射建立關係，找到相應的 Controller
2. org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping 通過 Controller 的類名找到請求的 Controller。
3. org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping 通過定義的 beanName 進行查找要請求的 Controller
4. org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping(spring 3.1之前)或RequestMappingHandlerMapping(spring 3.1) 通過註解 @RequestMapping(“/userlist”) 來查找對應的 Controller。

目前常用的方式是第四種，以 annotation 方式實現

#### **HandlerAdapter**
HandlerAdapter 負責對 Controller/Handler 進行執行，是Adapter Patterm 的應用，通過 Adapter 可以完成對更多類型的 Controller/Handler 進行執行，需根據 HandlerMapping 用的類別進行對應的設定

程式碼範例(xml 配置)：
* Spring3.1前的設定
```xml
<!--Spring3.1之前 HandlerMapping -->
<bean 
class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping" id="handlerMapping"/>

<!--Spring3.1之前 HandlerAdapter -->
<bean 
class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" id="handlerAdapter"/>
```
* Spring3.1設定
```xml
<!--Spring3.1開始 HandlerMapping -->
<bean 
class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" id="handlerMapping"/>
<!--Spring3.1開始 HandlerAdapter -->
<bean
class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" id="handlerAdapter"/>
```

#### **HandlerInterceptor**
HandlerInterceptor 負責將不合理的請求過濾，如請求是否為RESTful 規範，若不是，需要對請求進行攔截。和 Filter 的差別是差在 Filter 攔截的是 Servlet，HandlerInterceptor 攔截的是 Controller/Handler

HandlerInterceptor 有以下三個方法可供覆寫：
* preHandle(): 在 Controller 元件執行之前執行，回傳 true 代表繼續執行後續的 Controller，回傳 false 代表不執行後續元件
* postHandle(): 在 Controller 元件執行之後執行，只有在 preHandle() 回傳 true 的情況下才會執行，有能力修改 ModelAndView 內容，在呼叫 View 之前改變應用程式執行流程
* afterCompletion(): 在整個 HTTP Request 處理完成之後執行，只有在 preHandle() 回傳 true 的情況下才會執行，在處理完 View 元件之後給予 HTTP Request 處理流程 1 個後收尾的機會

HandlerInterceptor 可透過 WebMvcConfigurer 的 addInterceptors() 設定 HandlerInterceptor 的作用範圍。
* Java 配置
```java
@Configuration
public class SpringMVCJavaConfig implements WebMvcConfigurer {
    public void addInterceptors(InterceptorRegistry registry) {
        ...
    }
}
```
* xml 配置
```xml
<!-- 配置用於session驗證的攔截器 -->
<!-- 
    如果有多個攔截器滿足攔截處理的要求，則依據配置的先後順序來執行
 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 攔截所有的請求，這個必須寫在前面，也就是寫在【不攔截】的上面 -->
        <mvc:mapping path="/**" />
        <!-- 但是排除下面這些，也就是不攔截請求 -->
        <mvc:exclude-mapping path="/login.html" />
        <mvc:exclude-mapping path="/account/login" />
        <mvc:exclude-mapping path="/account/register" />
        <!-- 攔截器java代碼路徑 -->
        <bean class="com.ss871104.springmvc.interceptors.LogsInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```
```java
public class LogsInterceptor extends HandlerInterceptorAdapter {

    private static final Logger logger = LoggerFactory.getLogger(LogsInterceptor.class);
    
    private  NamedThreadLocal<String> logContext = new NamedThreadLocal<String>("log-id");

    @Autowired
    private TLogDao logDao;

    /**
     * preHandle方法是進行處理器攔截用的，顧名思義，該方法將在Controller處理之前進行調用，
     * SpringMVC中的Interceptor攔截器是鏈式的，可以同時存在多個Interceptor，
     * 然後SpringMVC會根據聲明的前後順序一個接一個的執行，
     * 而且所有的Interceptor中的preHandle方法都會在Controller方法調用之前調用。
     * SpringMVC的這種Interceptor鏈式結構也是可以進行中斷的，
     * 這種中斷方式是令preHandle的返回值為false，當preHandle的返回值為false的時候整個請求就結束了。
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String host = request.getRemoteHost();
        String url = request.getRequestURI();
        TLogEntity entity = new TLogEntity();
        entity.setCreateTime(new Timestamp(System.currentTimeMillis()));
        entity.setCreateUser("admin");
        entity.setIpAddress(host);
        entity.setLogUrl(url);
        entity.setIsSuccess("N");
        logDao.save(entity);
        logContext.set(entity.getLogId());

        logger.debug("IP為---->>> " + host + " <<<-----訪問了系統");
        return true;
    }

    /**
     * 這個方法只會在當前這個Interceptor的preHandle方法返回值為true的時候才會執行。
     * postHandle是進行處理器攔截用的，它的執行時間是在處理器進行處理之 後， 也就是在Controller的方法調用之後執行，
     * 但是它會在DispatcherServlet進行視圖的渲染之前執行，也就是說在這個方法中你可以對ModelAndView進行操作。
     * 這個方法的鏈式結構跟正常訪問的方向是相反的，也就是說先聲明的Interceptor攔截器該方法反而會後調用，
     * 這跟Struts2裏面的攔截器的執行過程有點像，
     * 只是Struts2裏面的intercept方法中要手動的調用ActionInvocation的invoke方法，
     * Struts2中調用ActionInvocation的invoke方法就是調用下一個Interceptor或者是調用action，
     * 然後要在Interceptor之前調用的內容都寫在調用invoke之前，要在Interceptor之後調用的內容都寫在調用invoke方法之後。
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }

    /**
     * 該方法也是需要當前對應的Interceptor的preHandle方法的返回值為true時才會執行。
     * 該方法將在整個請求完成之後，也就是DispatcherServlet渲染了視圖執行， 這個方法的主要作用是用於清理資源的，
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        String host = request.getRemoteHost();
        String logId = logContext.get();
        TLogEntity entity = logDao.findOne(logId);
        entity.setIsSuccess("Y");
        logDao.save(entity);

        logger.debug("IP為---->>> " + host + " <<<-----訪問成功");
    }

}
```

#### **HandlerExecutionChain**
HandlerMapping 返回給 DispatcherServlet 的不是可執行的 Controller/Handler，而是 HandlerExecutionChain 物件，他是若干的 HandlerInterceptor 與 Handler 的組合。這涉及到的是設計模式中的 Chain of Responsibility 設計模式，HandlerExecutionChain 將HandlerInterceptor 與 Handler 串成一個執行鏈的形式，首先請求會被第一個 HandlerInterceptor 攔截，如果返回 false, 那麼直接短路請求，如果返回 true, 那麼再交給第二個 HandlerInterceptor 處理，直到所有的 HandlerInterceptor 都檢查通過，請求才到達 Controller/Handler，交由 Handler 正式的處理請求。執行完成之後再逐層的返回。

#### **Controller/Handler 返回的值**
Controller/Handler 將業務邏輯層獲取的數據封裝到 ModelAndView 中，同時設置 ModelAndView 的 view 邏輯視圖名稱。從 ModelAndView 的名稱可以看出，它保存了 Handler 執行完成之後所需要發送到前端的數據，以及需要跳轉的路徑。除了回傳 ModelAndView，也可以回傳 void 和 String。
* return ModelAndView
```java
@Controller
@RequestMapping("/user")
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView hello() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("username", "javaboy");
        return mv;
    }
}
```
* return void
```java
@RequestMapping("/hello2")
public void hello2(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    req.getRequestDispatcher("/jsp/hello.jsp").forward(req,resp); // 通過request後端跳轉
}

@RequestMapping("/hello3")
public void hello3(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.sendRedirect("/hello.jsp"); // 通過response後端重定向
}

@RequestMapping("/hello3")
public void hello3(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.setStatus(302);
    resp.addHeader("Location", "/jsp/hello.jsp"); // 通過response手動指定實現重定向
}

@RequestMapping("/hello4")
public void hello4(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.setContentType("text/html;charset=utf-8");
    PrintWriter out = resp.getWriter();
    out.write("hello javaboy!");
    out.flush();
    out.close(); // 通過response做出回應
}
```
* return String
```java
@RequestMapping("/hello5")
public String hello5(Model model) {
    model.addAttribute("username", "javaboy"); // Model
    return "hello"; // 表示查找View
}

@RequestMapping("/hello6")
public String hello6() {
    return "forward:/jsp/hello.jsp";
}

@RequestMapping("/hello7")
public String hello7() {
    return "redirect:/user/hello";
}
```
* 若是要真的回傳 String，需要加上 @ResponseBody
```java
@RequestMapping("/hello8")
@ResponseBody
public String hello8() {
    return "redirect:/user/hello"; // 回傳此字串
}
```
---

### **ViewResolver**
ViewResolver 是 Spring MVC 將數據的獲取與數據的顯示渲染相分離的關鍵。DispatcherServlet 將 Controller/Handler 回傳的 ModelAndView，裡頭的響應結果數據(model)和邏輯視圖的路徑(logical view name)透過 ViewResolver 查找誰能把數據進行解析與渲染。

以下為尋找 View 可用的類別：
* BeanNameViewResolver : 將邏輯視圖名解析爲一個 Bean,Bean 的 id 等於邏輯視圖名。
* XmlViewResolver: 和 BeanNameViewResolver 類似，只不過目標視圖 Bean 對象定義在一個獨立的 XML 文件中，而非定義在 DispatcherServlet 上下文的主配置文件中
* InternalResourceViewResovlver: 將視圖名解析爲一個 URL 文件，一般使用該解析器將視圖名映射爲保存在 WEB-INF 目錄中的程序文件（如 JSP）
* XsltViewResolver: 將視圖名解析爲一個指定 XSLT 樣式表的 URL 文件
* JasperReportsViewResolver:JasperReports 是一個基於 java 的開源報表工具，該解析器將視圖名解析爲報表文件對應的 URL
* FreeMarkerViewResolver: 解析爲基於 FreeMarker 模板技術的模板文件
* VelocityViewResolver 和 VelocityLayoutViewResolver: 解析爲基於 Velocity 模板技術的模板文件

程式碼範例(InternalResourceViewResovlver為例)：
* application.properties
```properties
spring.mvc.view.prefix=/WEB-INF/views
spring.mvc.view.suffix=.jsp
```
* xml
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="viewResolver">
    <property name="prefix" value="/WEB-INF/views"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

---

### **View**
在根據邏輯視圖名藉助 ViewResolver 查找到對應的 View 實現類之後，DispatcherServlet 就會將 ModelAndView 中的數據交給 View 實現類來進行渲染，待該 View 渲染完成之後，會將渲染完成的數據交給 DispatcherServlet，這時候 DispatcherServlet 將其封裝到 Response 返回給前端顯示。

---

## **常用 annotation**
* @Controller: 宣告這是 SpringMVC Controller 物件，用於標記控制器層
* @RequestMapping: 標示請求的位址，若用在類別上表示所有回應請求的方法都是以該路徑作為父路徑，而它還有六個重要的參數。
    * value: 指定請求位址。
    * method : 指定請求的類型，如 GET、POST、PUT、DELETE 等。
    * consumes : 指定處理請求提交的內容類型(Content-Type)。
    * produces : 指定返回的內容類型，只有當請求頭中的Accept 類型中包含該指定類型才回傳。
    * headers : 指定請求中必須包含某些header 值。
    * params : 指定請求中必須包含某些參數值。
* @RequestParam: 將傳入變數的值綁定到與 @RequestParam 註釋指定名稱相同的參數上，若變數為非必須可以設定 required = false。
* @ModelAttribute : 將請求資訊封裝為指定物件。
* @SessionAttribute : 用於注入 Session 物件。

### **Spring3.0 支持 RESTful 風格後引入的 annotation:**
* @CookieValue： 將請求 request header 關於cookie的值綁定到方法的參數上
* @RequestHeader：將 request header 的值綁定到方法的參數上
* @RequestBody：常用來處理 application/json、application/xml 等 Content-Type 類型的資料，表示 HTTP 訊息是 JSON/XML 格式，需將其轉化為指定類型參數。
* @ResponseBody：透過適當的 HttpMessageConverter 將控制器中方法傳回的物件轉為指定格式(JSON/XML)後，寫入 Response 物件的 Body 資料區。
* @PathVariable：透過URL 中的範本變數 {PATH_PARAM} 綁定到與 @PathVariable 註釋指定名稱相同的參數上， @RequestMapping 可以定義動態的路徑。
* RestController: 作用相當於 @ResponseBody 加上 @Controller，用於回傳JSON、XML 等資料，但不能回傳HTML 頁面。
* GetMapping: @RequestMapping(method=RequestMethod.GET)的簡寫
* PostMapping: @RequestMapping(method=RequestMethod.POST)的簡寫
* PutMapping: @RequestMapping(method=RequestMethod.PUT)的簡寫
* DeleteMapping: @RequestMapping(method=RequestMethod.DELETE)的簡寫
* PatchMapping: @RequestMapping(method=RequestMethod.PATCH)的簡寫

---

## Spring Web MVC 多國語言(i18n)功能
* Spring MVC 延續 Spring 架構的多國語言(i18n)功能，主要參與元件是 LocalResolver
* DispatcherServlet 透過 LocalResolver 從 HTTP Request, Session, Cookie 等位置找出使用者的 Locale 資訊
* DispatcherServlet 將 Locale 資訊提供給處理 HTTP Request 的所有元件使用(例如：Controller, View 等)
* 取得 DispatcherServlet 提供的 Locale 資訊的語法：
```java
Locale locale = LocaleContextHolder.getLocale();
```

### **LocaleResolver**
org.springframework.web.servlet.LocaleResolver
* LocaleResolver 介面可被自定義實作 Locale 取得方式，但是 Spring MVC 內建實作類別足以應付一般 Web 應用程式的需求
* Spring Boot 自動設定機制預設定義 org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver
