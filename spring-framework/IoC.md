# **IoC Container 控制反轉容器**

## **Index 目錄**
* [IoC 底層原理](#ioc-底層管理)
* [IoC 介面 (BeanFactory)](#ioc-介面-beanfactory)
* [IoC 操作Bean管理](#ioc-操作bean管理)
    * [xml組態檔實作](#ioc-操作bean管理-基於-xml)
    * [FactoryBean (工廠Bean - 自定義Bean)](#ioc-操作bean管理---工廠bean-factorybean)
    * [Bean Scope (Bean的作用域)](#ioc-操作bean管理---bean的作用域-bean-scope)
    * [Bean Lifecycle (Bean的生命週期)](#ioc-操作bean管理---bean的生命週期-bean-lifecycle)
    * [自動配置](#ioc-操作bean管理---自動配置)
    * [配置外部屬性資料](#ioc-操作bean管理---配置外部屬性資料)
    * [annotation標籤實作](#ioc-操作bean管理-基於註解-anotation)

## **IoC 底層管理**
### **什麼是控制反轉？**
* 將物件建立和物件之間呼叫的過程，交給Spring進行管理<br>
Spring creates the objects, configures and assembles their dependencies, manages their entire life cycle.
* 使用IoC的目的：為了降低耦合度<br>
The pupose of IoC is to achieve loose-coupling between Objects dependencies.

### **控制反轉底層原理**
* xml解析
* 工廠方法模式 (Factory Method Pattern)
* 反射 (Reflection)

IoC過程 - 程式碼範例<br>
```xml
<!-- 第一步，xml組態檔，配置建立的物件 -->
<!-- First step, xml configuration file -->
<bean id="userDaoImpl" class="com.ss871104.spring.user.UserDaoImpl"></bean>
```
```java
// 第二步，建立工廠類別
// Second step, create a factory class
class UserFactory {
    public static UserDao getDao() {
        // xml解析
        String classValue = "com.ss871104.spring.user.UserDaoImpl";
        // 通過反射建立物件
        Class class = Class.forName(classValue);
        return (UserDao)class.newInstance();
    }
}
```

## **IoC 介面 (BeanFactory)**
### **實現控制反轉的兩大介面**
1. BeanFactory: IoC Container的基本實作，是Spring內部的使用介面，不提供開發人員使用。
     * 載入組態檔的時候不會建立物件，在獲取/使用物件才去建立物件
2. ApplicationContext: BeanFactory介面的子介面，提供更多強大的功能，一般由開發人員使用。
    * 載入組態檔的時候就會把在組態檔內的物件進行建立
    * ApplicationContext 介面有實作類別
        * FileSystemXmlApplicationContext
        * ClassPathXmlApplicationContext

測試程式<br>
```java
public class TestSpring {

    @Test
    public void testAdd() {
        // 載入spring組態檔
        ApplicationContext context = 
            new ClassPathXmlApplicationContext("bean.xml");

        // 獲取配置建立的物件
        UserDaoImpl userDao = context.getBean("userDaoImpl", UserDaoImpl.class);

        userDao.add();
    }
}
```

## **IoC 操作Bean管理**
### 什麼是Bean管理？
1. Spring 建立物件
2. Spring 注入元件

### Bean管理操作的兩種方式
* [xml 組態檔方式實作](#ioc-操作bean管理-基於-xml)
* [註解(Annotation)方式實作](#ioc-操作bean管理-基於註解-anotation)

### Bean管理
* [FactoryBean (工廠Bean - 自定義Bean)](#ioc-操作bean管理---工廠bean-factorybean)
* [Bean Scope (Bean的作用域)](#ioc-操作bean管理---bean的作用域-bean-scope)
* [Bean Lifecycle (Bean的生命週期)](#ioc-操作bean管理---bean的生命週期-bean-lifecycle)
* [自動配置](#ioc-操作bean管理---自動配置)
* [配置外部屬性資料](#ioc-操作bean管理---配置外部屬性資料)

---

### **IoC 操作Bean管理 (基於 xml)**
**建立物件**
```xml
<!-- 一般配置Bean -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置UserDao物件建立 -->
    <bean id="userDaoImpl" class="com.ss871104.spring.user.UserDaoImpl"></bean>

</beans>
```
1. 在xml組態檔中，使用bean標籤，標籤裡面添加物件屬性，就可以實作物件建立
2. bean標籤有很多屬性(property)：
    * id屬性：唯一標示
    * class屬性：類別全路徑
    * name屬性：和id一樣，可用特殊符號，起源是為了strut設計的，現在很少用
3. 建立物件時，預設是執行無參數建構子完成物件建立

**注入屬性**
1. DI (Dependency Injection): 依賴注入，就是注入屬性(property)
    * [Setter方法注入 (Setter Injection)](#setter方法注入-setter-injection---xml)
    * [建構式注入 (Constructor Injection)](#建構式注入-constructor-injection---xml)

---

#### **Setter方法注入 (Setter Injection) - xml**
優點：
* 完全符合單一功能原則，因為每一個Setter只針對一個物件

缺點：
* 無法注入一個不可變的物件 (final修飾的物件)
* 注入的物件可能隨時被修改

```xml
<!-- UserDao userDao = new UserDao(); -->
<bean id="userDaoImpl" class="com.ss871104.spring.user.UserDaoImpl"></bean>

<!-- UserService userService = new UserService(); -->
<bean id="UserService" class="com.ss871104.spring.user.UserService">
    <property name="userDao" ref="userDaoImpl"></property>
    <!-- name為UserService內的屬性名字 userDao; -->
</bean>
```
```java
public class UserService {
    // Setter 注入
    private UserDao userDao;

    public UserService() {

    }

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

Setter方法注入 - p 命名空間注入 (Spring 3.0 引入的功能)
* 要有無參數建構子
```xml
<!-- 一般配置Bean><-->
<!-- 加入 xmlns:p="http://www.springframework.org/schema/p" -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- UserDao userDao = new UserDao(); -->
    <bean id="userDaoImpl" class="com.ss871104.spring.user.UserDao"></bean>

    <!-- UserService userService = new UserService(); -->
    <bean id="UserService" class="com.ss871104.spring.user.UserService" p:userDao-ref="userDaoImpl">
    </bean>

</beans>
```

Setter方法注入 - 集合類別屬性注入
```java
public class CollectionTest {
    private String[] array;
    private List<String> list;
    private Map<String, String> map;
    private Set<String> set;
    private List<UserBean> userList;

    public void setArray(String[] array) {
        this.array = array;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMap(Map<String, String> map) {
        this.map = map;
    }
    public void setSet(Set<String> set) {
        this.set = set;
    }
    public void setUserList(List<UserBean> userList) {
        this.userList = userList;
    }
}
```

```xml
<bean id="collectionTest" class="com.ss871104.spring.collection.CollectionTest">
    <property name="array">
        <array>
            <value>陣列1</value>
            <value>陣列2</value>
        </array>
    </property>
    <property name="list">
        <list>
            <value>陣列1</value>
            <value>陣列2</value>
        </list>
    </property>
    <property name="map">
        <map>
            <entry key="鑰匙1" value="值1"></entry>
            <entry key="鑰匙2" value="值2"></entry>
        </map>
    </property>
    <property name="set">
        <set>
            <value>集合1</value>
            <value>集合2</value>
        </set>
    </property>
    <property name="userList">
        <list>
            <ref bean="userBean1"></ref>
            <ref bean="userBean2"></ref>
        </list>
    </property>
</bean>

<!-- 建立多個UserBean物件 -->
<bean id="userBean1" class="com.ss871104.spring.user.UserBean">
    <property name="name" value="andy"></property>
</bean>
<bean id="userBean2" class="com.ss871104.spring.user.UserBean">
    <property name="name" value="wayne"></property>
</bean>
```

Setter方法注入 - util 名稱空間注入
* 必須與p命名空間一起使用
```java
public class UserBean {
    private List<String> list;

    public void setList(:ist<String> list) {
        this.list = list;
    }
}
```

```xml
<!-- 一般配置Bean><-->
<!-- 加入 xmlns:util="http://www.springframework.org/schema/util" -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/util
    http://www.springframework.org/schema/util/spring-util.xsd">

    <util:list id="userList">
        <value>andy</value>
        <value>wayne</value>
        <value>eric</value>
    </util:list>

    <bean id="userBean" class="com.ss871104.spring.user.UserBean">
        <property name="list" ref="userList"></property>
    </bean>

</beans>
```

---

#### **建構式注入 (Constructor Injection) - xml**
優點：
* 可以注入不可變得物件 (final修飾的物件)
* 注入物件不會被修改，因為建構式注入在建立物件時只會執行一次
* 完全初始化，因為建構子是在物件被建立時執行
* 通用性更好，適用於IoC框架和非IoC框架

```xml
<!-- UserDao userDao = new UserDao(); -->
<bean id="userDaoImpl" class="com.ss871104.spring.user.UserDaoImpl"></bean>

<!-- UserService userService = new UserService(userdao); -->
<bean id="UserService" class="com.ss871104.spring.user.UserService">
    <constructor-arg ref="userDaoImpl"></constructor-arg>
</bean>
```
```java
public class UserService {
    // 建構式注入
    private UserDao userDao;

    public UserService() {

    }

    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

建構式注入 - 多參數處理
```xml
<!-- UserBean userBean = new UserBean(id, name); -->
<bean id="userBean" class="com.ss871104.spring.user.UserBean">
    <constructor-arg name="id" value="1"></constructor-arg>
    <constructor-arg name="name" value="andy"></constructor-arg>
</bean>
```
```java
public class UserBean {
    // 建構式注入 - 多參數處理
    private Integer id;
    private String name;

    public UserBean(Integer id, String name) {
        this.id = id;
        this.name = name;
    }

    public UserBean() {
    // TODO Auto-generated constructor stub
    }   
}
```

建構式注入 - c 命名空間注入 (Spring 3.0 引入的功能)
* 需有參數建構子
```xml
<!-- 一般配置Bean -->
<!-- 加入 xmlns:c="http://www.springframework.org/schema/c" -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- UserDao userDao = new UserDao(); -->
    <bean id="userDaoImpl" class="com.ss871104.spring.user.UserDaoImpl"></bean>

    <!-- UserService userService = new UserService(userDao); -->
    <bean id="UserService" class="com.ss871104.spring.user.UserService" c:userDao-ref="userDaoImpl">
    </bean>

</beans>
```

---

### **IoC 操作Bean管理 - 工廠Bean FactoryBean**
Spring 有兩種 bean：

* Bean: 在組態檔中定義bean型態就是回傳型態

* FactoryBean: 在組態檔中定義bean型態可以和回傳型態不一樣，其目的是為了讓開發者自定義Bean的創建過程
    1. 創建類別，實作介面 FactoryBean
    2. 實作FactoryBean介面的方法，在實作的方法中定義回傳的bean型態
        * getObject() - 用來建立物件
        * getObjectType() - 用來返回物件的資料型態
        * isSingleton() - 用來指定建立的物件是否為單例

```java
public class MyBean implements FactoryBean {

    @Override
    public Object getObject() throws Exception {
        UserBean user = new UserBean();
        user.setName("andy");
        return user;
    }

    @Override
    public Class<?> getObjectType() {
        return UserBean.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }

}
```
```xml
<bean id="myBean" class="com.ss871104.spring.factorybean.MyBean">
</bean>
```

---

### **IoC 操作Bean管理 - Bean的作用域 Bean Scope**
Bean的作用域：設定Bean元件的有效範圍，預設值為singleton單例，共有4個合法值
* singleton: 每次使用相同id呼叫getBean()方法都會取得相同物件
* prototype: 每次使用相同id呼叫getBean()方法都會取得全新物件
* request: 在相同HTTP Request之內，每次使用相同id呼叫getBean()方法都會取得相同物件，只能在Web應用使用。
* session: 在相同HTTP Session之內，每次使用相同id呼叫getBean()方法都會取得相同物件，只能在Web應用使用。

```xml
<bean id="user" class="com.ss871104.spring.user.UserBean" scope="prototype">
</bean>
```

singleton和prototype的區別
* singleton是單例，prototype是多例
* 設定scope為singleton時，載入spring組態檔時就會建立單例物件
* 設定scope為prototype時，不是在載入spring組態檔時建立物件，是在呼叫getBean()方法時建立多例物件

---

### **IoC 操作Bean管理 - Bean的生命週期 Bean Lifecycle**
![Bean的生命週期](/spring-framework/bean-lifecycle.png)<br>
* Bean的生命週期指的是從建立物件到物件銷毀的過程
* Bean的生命週期流程
    1. 通過建構子建立bean的物件(無參數建構子)
    2. 為bean的屬性(property)設定值和對其他bean的引用(呼叫setter方法)
    3. 呼叫bean的初始化方法(需要配置初始化的方法)
    4. bean可以使用了(物件獲取到了)
    5. 當容器關閉時，呼叫bean的銷毀方法(需要配置銷毀的方法)

**程式碼範例**
```xml
<bean id="orders" class="com.ss871104.spring.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
    <property name="oname" value="手機"></property>
</bean>
```
```java
public class Orders() {

    private String oname;

    // 無參數建構子
    public Orders() {
        System.out.println("第一步，執行無參數建構子建立bean");
    }

    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步，呼叫setter方法設定屬性值");
    }
    // 初始化方法
    public void initMethod() {
        System.out.println("第三步，執行初始化的方法");
    }
    // 銷毀方法
    public void destroyMethod() {
        System.out.println("第五步，執行銷毀的方法");
    }
}
```
```java
@Test
public void testBean() {
    // ApplicationContext context = 
    //     new ClassPathXmlApplicationContext("bean.xml");
    ClassPathXmlApplicationContext context = 
        new ClassPathXmlApplicationContext("bean.xml");
    Orders orders = context.getBean("orders", Order.class);
    System.out.println("第四步，獲取創建的bean的物件");
    System.out.println(orders);

    // 銷毀方法
    context.close();
}
```
---

#### **BeanPostProcessor 介面**
* 在bean初始化前後執行方法
* 實作BeanPostProcessor介面會將 bean 的生命週期 +2步
* Bean的生命週期流程 implements BeanPostProcessor
    1. 通過建構子建立bean的物件(無參數建構子)
    2. 為bean的屬性(property)設定值和對其他bean的引用(呼叫setter方法)
    3. **把bean的物件傳給BeanPostProcessor的方法 (postProcessBeforeInitialization 方法)**
    4. 呼叫bean的初始化方法(需要配置初始化的方法)
    5. **把bean的物件傳給BeanPostProcessor的方法 (postProcessAfterInitialization 方法)**
    6. bean可以使用了(物件獲取到了)
    7. 當容器關閉時，呼叫bean的銷毀方法(需要配置銷毀的方法)

**程式碼範例**
```xml
<!--配置BeanPostProcessor-->
<bean id="myBeanPost" class="com.ss871104.spring.bean.MyBeanPost">
</bean>
```
```java
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之前的方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之後的方法");
        return bean;
    }
}
```

---

### **IoC 操作Bean管理 - 自動配置**
#### 什麼是自動配置
* 根據指定配置規則(屬性名稱或屬性類型)，Spring 自動將相應的屬性值進行注入
* 通常都是用標籤annotation去完成自動配置

自動配置過程
```xml
<!--bean標籤屬性autowire，自動配置
    autowire屬性常用的兩個值：
        byName根據屬性名稱注入，注入的bean的id值合類別屬性名稱一樣
        byType根據屬性類別注入-->
<bean id="emp" class="com.ss871104.spring.autowire.Emp" autowire="byName"></bean>
<bean id="dept" class="com.ss871104.spring.autowire.Dept"></bean>
```

---

### **IoC 操作Bean管理 - 配置外部屬性資料**
以配置資料庫為例：

**直接配置連線池**
```xml
<!--內部配置連線池-->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    p:driverClassName="com.mysql.jdbc.Driver"
    p:url="jdbc:mysql://localhost:3306/testdb"
    p:username="root"
    p:password="password"
/>
```

**引入外部文件**
```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/testdb
jdbc.username=root
jdbc.password=password
```
```xml
<!--引入外部屬性文件(以properties檔為例)-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    p:driverClassName="${jdbc.driverClassName}"
    p:url="${jdbc.url}"
    p:username="${jdbc.username}"
    p:password="${jdbc.password}"
/>
```

### **IoC 操作Bean管理 (基於註解 Anotation)**
**什麼是註解？**
* 註解是程式碼的特殊標記，格式：@Annotation(property="???")
* 註解可作用在類別、方法、屬性上
* 使用註解的目的：簡化xml配置

**Bean管理中建立物件的註解**
* @Component
* @Service - 商業邏輯層
* @Controller - 外部連結
* @Repository - Dao層

以上註解功能是一樣的，都可以用來建立bean物件

程式碼範例：<br>
開啟組件掃描
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!--開啟組件掃描 (掃描整包)-->
    <context:component-scan base-package="com.ss871104"></context:component-scan>

    <!--自己配置掃描的檔案，只掃描有@Controller的檔案-->
    <context:component-scan base-package="com.ss871104" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.streotype.Controller">
    </context:component-scan>

    <!--自己配置掃描的檔案，只不掃描有@Controller的檔案其他都掃描-->
    <context:component-scan base-package="com.ss871104">
        <context:exclude-filter type="annotation" expression="org.springframework.streotype.Controller">
    </context:component-scan>

</beans>
```
```java
// 註解理的value屬性可省略不寫，預設是首字母小寫
// UserService -> userService
@Component(value = "userService") // <bean id="userService " class="">
public class UserService {

    public void add() {
        System.out.println("service add");
    }
}
```

**注入屬性 (註解方式)**
* @Autowired<br>
    根據屬性類別進行資動裝配
* @Qualifier<br>
    根據屬性名稱進行注入 (要和＠Autowired一起使用)
* @Resource<br>
    可以根據類別注入也可以根據名稱注入，是javax.annotation套件底下的註解
* @Value<br>
    注入普通類別屬性

程式碼範例
* @Autowired
```java
@Service
public class UserService {
    // 定義dao類別屬性
    // 不需要添加setter方法
    // 添加注入屬性註解
    @Autowired
    private UserDao userDao;

    public void add() {
        System.out.println("service add");
        userDao.add();
    }
}
```
* @Qualifier
```java
 @Service
public class UserService {
    // 定義dao類別屬性
    // 不需要添加setter方法
    // 添加注入屬性註解
    @Autowired
    // 當有多個實作類別時可用
    @Qualifier(value = "userDaoImpl1")
    private UserDao userDao;

    public void add() {
        System.out.println("service add");
        userDao.add();
    }
}
```
* @Resource
```java
@Service
public class UserService {
    // 兩種皆可
    @Resource // 類別進行注入
    @Resoucre(value = "userDaoImpl1") // 名稱進行注入
    private UserDao userDao;

    public void add() {
        System.out.println("service add");
        userDao.add();
    }
}
```
* @Value
```java
@Service
public class UserService {
    
    @Value(value = "abc")
    private String name;

    @Autowire
    private UserDao userDao;

    public void add() {
        System.out.println("service add" + name);
        userDao.add();
    }

}
```

**Java 類別組態配置**<br>
由POJO 類別 + 特殊的annotation組成，可替代xml寫法
* @Configuration<br>
    定義在Java類別上，告訴Spring架構這個類別是專門做組態設定的特殊類別
* @Bean<br>
    定義在方法上，告訴Spring架構這是一個需要使用Spring IoC技術控管的Bean元件
* @ComponentScan<br>
    定義在Java類別上，告訴Spring架構需要搜尋的套件名稱

程式碼範例：
```java
// 作為配置類別，替代xml組態檔
@Configuration
// 需要Spring搜尋的套件名稱
@ComponentScan(basePackage={"com.ss871104"})
public class SpringConfig {

    @Bean
    public UserDao userDao() {
        return new UserDaoImpl();
    }
}
```
測試需改成以下形式
```java
public class TestSpring {

    @Test
    public void testAdd() {
        // 載入spring Java類別組態檔
        ApplicationContext context = 
            new AnnotationConfigApplicationContext("SpringConfig.class");

        // 獲取配置建立的物件
        UserDaoImpl userDao = context.getBean("userDaoImpl", UserDaoImpl.class);

        userDao.add();
    }
}
```