# **Spring對資料庫的支援**

## **Index 目錄**
* [DataSource](#datasource)
* [JdbcTemplate](#jdbctemplate)
* [對ORM框架的支援(Hibernate/JPA)](#對orm框架的支援hibernatejpa)
* [資料庫事務操作(Transaction)](#資料庫事務操作transaction)
* [Java組態設定範例](#java組態設定範例)

---

## **DataSource**
### **JNDI DataSource主要功能**
* 隱藏資料庫連線資訊：資料庫連線資訊設定在Server上，應用程式使用JNDI規格規定的方式搜尋連線資訊
* 隱藏背後的Connection Pool實作：Connection Pool隨著Server而改變，利用DataSource隱藏Connection Pool的實作，讓Web應用程式可以隨時更換Server而不需要改動程式
* 只能在Web應用程式運作：DataSource需要註冊到JNDI Server，只有Web應用程式內的JDBC程式碼才能使用DataSource(Java Application無法跑DataSource)

### **Spring對DataSource支援**
#### Spring透過xml設置DataSource
程式碼範例：
```xml
<!--內部配置連線池-->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    p:driverClassName="com.mysql.jdbc.Driver"
    p:url="jdbc:mysql://localhost:3306/testdb"
    p:username="root"
    p:password="password"
/>
```
```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/testdb
jdbc.username=root
jdbc.password=password
```
```xml
<!--引入外部文件配置連線池(以properties檔為例)-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    p:driverClassName="${jdbc.driverClassName}"
    p:url="${jdbc.url}"
    p:username="${jdbc.username}"
    p:password="${jdbc.password}"
/>
```

#### SpringBoot設置DataSource
* 如果引入以下兩個套件，SpringBoot會自動設定DataSource
    * spring-boot-starter-jdbc
    * spring-boot-starter-data-jpa
* 如果有宣告內嵌式資料庫，Spring會自動設定DataSource連接內嵌式資料庫
    * H2
    * HSQL
    * Derby
* 如果沒有宣告內嵌式資料庫，Spring會無法自動完成DataSource的自動設定而執行失敗，需修改自動設定機制預設值
* SpringBoot的DataSource自動設定機制支援三種Connection Pool
    * HikariCP connection pool(預設值，據說是最快的)
    * Tomcat dbcp connection pool
    * Apache Commons dbcp2
    * 如果要修改connection pool，需要將HikariCP connection pool函式庫移除，然後宣告其他connection pool函式庫

修改DataSource自動設定機制預設值
* application.properties設定，mysql資料庫為例
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/TestDB?serverTimezone=Asia/Taipei
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.drive-class-name=com.mysql.cj.jdbc.Driver
```
注：SpringBoot可通過url判斷driver-class-name，所以可省略driver-class-name設定
* application.yml設定，mysql資料庫為例
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/TestDb?serverTimezone=Asia/Taipei
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
```

---

## **JdbcTemplate**
Retrieved from https://medium.com/@steph.c/jdbctemplate-%E7%AF%84%E4%BE%8B-2c9a1f3718ba

JdbcTemplate 是 Spring 框架中的一個 class；提供一些 API 將 JDBC 的操作封裝起來，使用上類似 JDBC 但更簡單方便，也避免了一些 JDBC 操作上的常見錯誤。

以下是常用的方法：
* execute: 執行任何 SQL 都可以使用，常見使用於操作資料表相關的 SQL (create table, drop table…)
* update/ batchUpdate: 雖然這方法是 update， 但資料的新增/ 修改/ 刪除都可以用這個方法。 batchUpdate 跟 update 類似，但 update 是一筆一筆執行，會對資料庫進行頻繁的存取，batchUpdate 則是批次執行，累積一定量以後再對資料庫做存取。
* query/ queryForXXX: 用於查詢資料，ForXXX的部分是根據查詢語法返回的物件使用對應的方法。
(例如預期會查到一筆資料可以使用 queryForObject, 如果預期會返回多筆資料可以使用 queryForList)
* call: 常用於呼叫 Stored Procedure

程式碼範例：
* 建立資料表
```java
// create table by calling execute(String sql)
public void createTable() {
  String sql = "CREATE TABLE `student`(
  `id` bigint unsigned NOT NULL auto_increment,
  `name` varchar(255) NOT NULL,
  `age` int NULL default NULL,
  primary key(id),
  unique key id_UNIQUE(id))"
  jdbcTemplate.execute(sql); // execute SQL to create table
}
```
* 新增一筆資料
```java
public void addStudent(Student student) {
	String sql = "INSERT INTO student (name, age) VALUES (?, ?)";
	KeyHolder keyHolder = new GenerateKeyHolder();

	// public int update(final PreparedStatementCreator psc, final KeyHolder generatedKeyHolder)
	this.jdbcTemplate.update(connection -> {
		PreparedStatement ps = connection.preparedStatement(sql, Statement.RETURN_GENERATE_KEYS);
		ps.setString(1, student.getName()); // 1st question mark
		ps.setInt(2, student.getAge()); // 2nd question mark
		return ps;
	}, keyHolder);

	int id = keyHolder.getKey().intValue(); // get PK of the student that you just add
}
```
* 新增多筆資料，使用批次執行
```java
public int addOutboundShortNum_batch(List<Student> studentList) {
	String sql = "INSERT INTO student (name, age) VALUES (?, ?)"
	int[][] updateCounts = jdbcTemplate.batchUpdate(sql, studentList, studentList.size(),
			new ParameterizedPreparedStatementSetter<Student>() {
				public void setValues(PreparedStatement ps, Student student) throws SQLException {
					ps.setString(1, student.getName()); // 1st question mark
					ps.setInt(2, student.getAge()); // 2nd question mark
				};
			}
		);	
	return updateCounts.length;
}
```
* 修改資料
```java
public int updateStudentById(int id, String name) throws SQLException {
	String sql = "UPDATE student SET name = ? WHERE id = ?";
	int successCount = this.jdbcTemplate.update(sql, new Object[] { id, name });
	return successCount;
}
```
* 刪除資料
```java
public int deleteStudentById(int id) {
	String sql = "DELETE FROM student WHERE id = 2";
	int successCount = this.jdbcTemplate.update(sql, new Object[] { id });
	return successCount;
}
```
* 查詢資料
```java
// Demo 1, return Student instance
public Student getStudentById(int id) {
	String sql = "SELECT * FROM student WHERE id = ?";
	Student student = null;
	try {
		student = this.jdbcTemplate.queryForObject(sql, new Object[] { id }, new BeanPropertyRowMapper<Student>(Student.class)); // BeanPropertyRowMapper converts a row into a new instance of the specified mapped target class
	} catch (EmptyResultDataAccessException e) {
		e.printStackTrace();
	}
	return student;
}
```
想要查詢拿回整個 Student 物件回來時，可以使用 queryForObject 方法並透過 BeanPropertyRowMapper 來實現。BeanPropertyRowMapper 會將資料表中的欄位跟物件中的變數(variable)做映射(mapping)。

值得注意的是，如果想使用BeanPropertyRowMapper 幫忙做 mapping 的話，被 mapping 的 class 中的變數一定要有 setter 方法，否則會映射失敗。另外，資料表的欄位名稱與 class 中的變數名稱對應也有特定規則，變數名稱只可採用與欄位相同名稱或駝峰是命名

* 查詢資料，獲取某欄位值
```java
// demo 2, return name of the student
public Student getStudentNameById(int id) {
	String sql = "SELECT name FROM student WHERE id = ?";
	String name = "";
	try {
		name = this.jdbcTemplate.queryForObject(sql, new Object[] { id }, String.class);
	} catch (EmptyResultDataAccessException e) {
		e.printStackTrace();
	}
	return name;
}
```
跟想要獲取整個 Student 物件不同，如果只是要獲取某個欄位值，並不需要使用 BeanPropertyRowMapper 做映射，只需要使用欲獲取的欄位值的相應型態即可。

上面兩個查詢的例子都使用了 queryForObject 方法，但 query 系列尚有
1. queryForList
2. queryForMap
3. queryForRowSet
若從資料庫撈取的資料大於一筆時使用 queryForObject 會造成 Run time exception，此時就可以用上面的三種方法代替。queryForList 是三種裡面最直接的，因為他會直接幫做好映射然後返回指定類別的 List；不像 queryForMap 跟 queryForRowSet，需要自行遍歷再用對應名稱取得值。

---

## **對ORM框架的支援(Hibernate/JPA)**
Spring支援Hibernate、Java Persistence API等OR-Mapping技術
* Hibernate: 存取關聯式資料庫資料的ORM技術
* JPA: 存取關聯式資料庫資料的規格，底層是實作JPA規格的ORM技術，例如Hibernate, Eclipse Link等

### **Spring對Hibernate技術的支援**
Spring對Hibernate技術的支援非常多，此處將介紹以下兩種
* [Spring對SessionFactory的支援](#spring對sessionfactory的支援)
* [Spring對Transaction的支援](#spring對transaction的支援)

---

### **Spring對SessionFactory的支援**
Spring 可以與Hibernate結合使用，Hibernate的連結、事務管理等是由建立SessionFactory開始的，SessionFactory在 應用程式中通常只需存在一個實例，因而SessionFactory底層的DataSource可以使用Spring的 IoC注入，之後您再注入SessionFactory至相依的物件之中。

可將hibernate.cfg.xml組態檔刪除，因為這部份可以由Spring在Bean定義檔中撰寫DataSource設定及依賴注入來取代。<br>
Retrieved from https://openhome.cc/Gossip/SpringGossip/SessionFactoryInjection.html
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE beans PUBLIC "-//SPRING/DTD BEAN/EN" 
 "http://www.springframework.org/dtd/spring-beans.dtd"> 
<beans> 
    <bean id="dataSource" 
          class="org.springframework.jdbc.datasource.DriverManagerDataSource"> 
        <property name="driverClassName"> 
            <value>com.mysql.jdbc.Driver</value> 
        </property> 
        <property name="url"> 
            <value>jdbc:mysql://localhost:3306/demo</value> 
        </property> 
        <property name="username"> 
            <value>caterpillar</value> 
        </property> 
        <property name="password"> 
            <value>123456</value> 
        </property>  
    </bean> 
    
    <bean id="sessionFactory"  
          class="org.springframework.orm.hibernate3.LocalSessionFactoryBean" 
          destroy-method="destroy"> 
        <property name="dataSource"> 
            <ref bean="dataSource"/> 
        </property> 
        <property name="mappingResources"> 
            <list> 
                <value>onlyfun/caterpillar/User.hbm.xml</value> 
            </list> 
        </property> 
        <property name="hibernateProperties"> 
            <props> 
                <prop key="hibernate.dialect"> 
                    org.hibernate.dialect.MySQLDialect
                </prop> 
            </props> 
        </property> 
    </bean> 

    <bean id="userDAO" class="onlyfun.caterpillar.UserDAO"> 
        <property name="sessionFactory"> 
            <ref bean="sessionFactory"/> 
        </property> 
    </bean> 
</beans>
```
可以看到使用Spring整合Hibernate的好處，可以直接將DataSource注入至 org.springframework.orm.hibernate3.LocalSessionFactoryBean中，至於Hibernate所 需的相關設定，則可透過LocalSessionFactoryBean的相關屬性來設定，像是設定資料庫名稱、使用者名稱、密碼等， LocalSessionFactoryBean會建立SessionFactory的實例，並在執行依賴注入時將這個實例設定給UserDAO。

Hibernate的物件與關聯表格的映射文件之位置與名稱，則指定於"mappingResources"屬性中，如果自行提供Hibernate本身的設定檔（hibernate.cfg.xml），也可以使用 "configLocation"屬性來指定組態檔的位置，而這邊則使用"hibernateProperties"屬性在Spring的 Bean組態檔中直接指定，可以藉此減少對XML組態檔案的管理。

---

### **Spring對Transaction的支援**
OR-Mapping的架構很多(JDBC, Hibernate, JPA, JDO, JTA)，管理的transation的機制也不同。因此Spring提供transaction管理機制讓程式設計師可以使用相同方式管理不同OR-Mapping架構的transaction。

Spring的transaction管理可分為兩種：
* 程式設計式(Programming transaction demarcation)：適用於只有少量交易的情況，使用TransactionTemplate與PlatformTransaction Manager撰寫程式呼叫commit()、rollback()管理交易 (限制大，開發時不常使用)
* 宣告式(Declarative transaction demarcation)：適用於大量交易，使用xml或是annotation方式宣告transaction管理規則

#### **Spring宣告式transaction管理機制**
* Spring根據宣告在Service, Dao程式(通常為Service)插入transaction管理程式碼：企業邏輯程式與transaction管理程式相互分離
* Spring依賴AOP功能支援宣告式transaction管理機制，Spring AOP功能作用在方法，所以宣告式transaction管理機制作用在方法
* 設定transaction的相關transaction參數目的在描述transaction應用到各個方法的策略
* Spring transaction管理機制由PlatformTransactionManager類別與@Transactional提供
    * PlatformTransactionManager屬於org.springframework.transaction套件，定義各個OR-Mapping架構的Transaction Manager所必須提供的功能
    * 使用Spring宣告式Transaction管理機制，必須根據後端OR-Mapping架構在Spring組態設定檔設定PlatformTransactionManager

Spring組態配置TransactionManager程式碼範例：
```xml
<!--DataSource-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="url" value="jdbc:mysql://localhost:3306/TestDb">
    <property name="username" value="root">
    <property name="password" value="password">
    <property name="driverClassName" value="com.mysql.jdbc.Driver">
</bean>
<!--Transaction Manager-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!--Transaction Manager annotation-->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```
```java
@Service
@Transactional // 替這個類別的所有方法做事務管理
public class UserService {
    @Autowired
    private UserDao userDao;

    // 轉錢的方法
    public accountMoney() {
        // 轉帳扣錢的方法
        userDao.reduceMoney();

        // 轉帳加錢的方法
        userDao.addMoney();
    }

}
```

#### **@Transactional的重要屬性**
* transactionManager: transaction使用的PlatformTransactionManager的bean名稱
* value: transaction使用的PlatformTransactionManager的bean名稱
* readOnly: 唯讀提示，預設值false
* timeout: 逾時區間(單位:second)，預設值-1(底層資料庫的預設transaction timeout period)
* propagation: 傳遞行為，預設值Propagation.REQUIRED
* isolation: 隔離層級，預設值Isolation.DEFAULT(資料庫隔離設定)
* rollbackFor: 造成rollback的exception型別
* rollbackForClassName: 造成rollback的exception類別全名
* noRollbackFor: 不rollback的exception型別
* noRollbackForClassName: 不rollback的exception類別全名

---

## **資料庫事務操作(Transaction)**
資料庫事務(transaction)是訪問並可能操作各種資料項的一個資料庫操作序列，這些操作要麼全部執行,要麼全部不執行，是一個不可分割的工作單位。事務由事務開始與事務結束之間執行的全部資料庫操作組成。

### **事務的ACID原則**
* 原子性(Atomicity): 表示多個步驟中不能只發生其中一個動作，要馬全部成功，要馬全部失敗
* 一致性(Consistency): 針對一個事務操作前與操作後的狀態一致，例如雙方交易前和交易後的錢都不能小於 0，且雙方錢的總和不能改變，若是無法遵守，交易將會失敗
* 隔離性(Isolation): 事務的執行不受其他事務的干擾，且不能修改到同一個值，事務執行的中間結果對其他事務必須是透明的
* 永續性(Durability): 對於任意已提交事務，系統必須保證該事務對資料庫的改變不被丟失，即使資料庫出現故障

### **傳播行為（Propagation behavior）**
傳播行為定義了交易應用於方法上之邊界（Boundaries），它告知何時該開始一個新的交易，或何時交易該被暫停，或者方法是否要在交易中進行。

當客戶端本身不在交易中，而呼叫另一個方法時，該方法可能：
* 在非交易中進行
* 啟始新的交易並於其中執行
* 丟出例外

當客戶端本身在交易中，而呼叫另一個方法時，該方法可能：
* 在客戶端的交易中進行
* 啟始新的交易並於其中執行
* 暫停客戶端交易，於非交易環境中執行
* 丟出例外

Spring定義幾個傳播行為，可在TransactionDefinition的API文件說明上找到相對應的常數與說明:

| 交易區間策略 | 說明 |
| :---------| :-- |
| PROPAGATION_REQUIRED | 支援現在的交易，如果沒有的話就建立一個新的交易 |
| PROPAGATION_REQUIRES_NEW | 建立一個新的交易，如果現存一個交易的話就先暫停，並啟始一個新的交易來執行 |
| PROPAGATION_SUPPORTS | 支援現在的交易，如果沒有的話就以非交易的方式執行 |
| PROPAGATION_MANDATORY | 	方法必須在一個現存的交易中進行，否則丟出例外 |
| PROPAGATION_NOT_SUPPORTED | 指出不應在交易中進行，如果有的話就暫停現存的交易 |
| PROPAGATION_NEVER | 指出不應在交易中進行，如果有的話就丟出例外 |
| PROPAGATION_NESTED | 在一個巢狀的交易中進行，如果不是的話，則同PROPAGATION_REQUIRED |

若客戶端本身不在交易中，而呼叫另一個方法時，依該方法設定的策略，而會有的行為對應如下：
* 在非交易中進行（SUPPORTS、NOT_SUPPORTED、NEVER）
* 啟始新的交易並於其中執行（REQUIRED、REQUIRES_NEW）
* 丟出例外（MANDATORY）

若客戶端本身在交易中，而呼叫另一個方法時，依該方法設定的策略，而會有的行為對應如下：
* 在客戶端的交易中進行（REQUIRED、SUPPORTS、MANDATORY）
* 暫停客戶端交易，啟始新的交易並於其中執行（REQUIRED_NEW）
* 暫停客戶端交易，於非交易環境中執行（NOT_SUPPORTED）
* 暫停客戶端交易，於非交易環境中執行（NOT_SUPPORTED）
* 丟出例外（NEVER）

### **隔離層級（Isolation level）**
隔離性是交易的保證之一，表示交易與交易之間不互相干擾，好像同時間就只有自己的交易存在一樣，隔離性保證的基本方式是在資料庫層面，對資料庫或相關欄位鎖定，在同一時間內只允許一個交易進行更新或讀取。

隔離性可帶來的問題：
* 髒讀(Dirty read): 一個事務讀取了另外一個事務未提交的資料
* 不可重複讀(Nonrepeatable read): 一個事務先後讀取同一條記錄，而事務在兩次讀取之間該資料被其它事務所修改，則兩次讀取的資料不同
* 幻讀(Phantom read): 一個事務按相同的查詢條件重新讀取以前檢索過的資料，卻發現其他事務插入了滿足其查詢條件的新資料

Spring提供了幾種隔離層級設定，同樣的可以在TransactionDefinition的API文件說明上找到相對應的常數與說明:

| 隔離層級 | 說明 |
| :------ | :---|
| ISOLATION_DEFAULT | 使用底層資料庫預設的隔離層級 |
| ISOLATION_READ_COMMITTED | 允許交易讀取其它並行的交易已經送出（Commit）的資料欄位，可以防止Dirty read問題 |
| ISOLATION_READ_UNCOMMITTED | 允許交易讀取其它並行的交易還沒送出的資料，會發生Dirty、Nonrepeatable、Phantom read等問題 |
| ISOLATION_REPEATABLE_READ | 要求多次讀取的資料必須相同，除非交易本身更新資料，可防止Dirty、Nonrepeatable read問題 |
| ISOLATION_SERIALIZABLE | 完整的隔離層級，可防止Dirty、Nonrepeatable、Phantom read等問題，會鎖定對應的資料表格，因而有效能問題 |

* 唯讀提示（Read-only hints）
如果交易只進行讀取的動作，則可以利用底層資料庫在唯讀操 作時的一些最佳化動作，由於這個動作利用到資料庫在唯讀的交易操作最佳化，因而必須在交易中才有效，也就是說您要搭配傳播行為 PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、 PROPAGATION_NESTED來設置。

* 交易超時期間（The transaction timeout period）
有的交易操作可能延續一段很長的時間，交易本身可能關聯到資料表格的鎖定，因而長時間的交易操作會有效能上的問題，對於過長的交易操作，您要考慮回滾（Roll back）交易並要求重新操作，而不是無限時的等待交易完成。

可以設置交易超時期間，計時是從交易開始時，所以這個設置必須搭配傳播行為PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED來設置。

Retrieved from https://openhome.cc/Gossip/SpringGossip/TransactionAttribute.html

---

## **Java組態設定範例**
```java
@Configuration // 組態設定類別
@ComponentScan(basePackage = "com.ss871104") // 組件掃描
@EnableTransactionManagement // 啟用事務transaction
public class SpringConfig {

    // DataSource
    @Bean
    public BasicDataSource getBasicDataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/TestDb?serverTimezone=Asia/Taipei");
        dataSource.setUsername("root");
        dataSource.setPassword("password");

        return dataSource;
    }

    // JdbcTemplate
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        // 連接datasource
        jdbcTemplate.setDataSource(dataSource);

        return jdbcTemplate;
    }

    // TransactionManager
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager() {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        // 連接datasource
        transactionManager.setDataSource(dataSource);

        return transactionManager;
    }
}
```