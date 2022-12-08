# **spring-framework**

## **Index 目錄**
* [Intro 簡介](#intro-簡介)
* [IoC & Spring 組態配置](/spring-framework/IoC.md)
* [AOP 面向切面](/spring-framework/AOP.md)
* [Spring 對資料庫的支援](/spring-framework/spring-database.md)

## **Intro 簡介**

### **什麼是 Spring 框架？**

1. Spring 是輕量級的開源的JavaEE框架，不需要 Java EE Application Server 就可以執行

2. Spring 有兩大核心：IoC 和 AOP
    * **IoC (Inversion of Control) 控制反轉**: 把建立物件的過程交給 Spring 進行管理，目的在降低應用程式的元件與元件之間的相依關係，進而降低耦合度，讓每個元件的可重用性提高，讓應用程式更具延展性
    * **AOP (Aspect-Oriented Programming) 面向切面**: 不更改原始程式碼的情況下插入特定功能的程式碼，通常需要DI的輔助來實作

3. Spring 特點
    * 降低耦合度，方便開發 (loose coupling)
    * AOP 編寫支持
    * 方便程式碼測試
    * 方便合其他框架進行整合
    * 方便進行事務操作
    * 降低 API 開發難度

### **Spring Framework Runtime**
 
 ![spring-framework](https://docs.spring.io/spring-framework/docs/4.3.x/spring-framework-reference/htmlsingle/images/spring-overview.png)<br>
Retrieved from Spring Framework Reference Documentation

#### **Spring Framework 的各個模組**
* Core Container: 提供 Spring Framework 的基礎功能，主要元件是 BeanFactory, BeanFactory 實作 Factory design pattern, 可以管理物件的生成、釋放以及物件的依賴關係
* Context 模組: 提供讀取 Spring Framework 組態設定檔的功能
* AOP 模組: 提供 Spring Framework 的 AOP 功能
* JDBC 模組: 提供 JDBC 相關功能，讓繁瑣的 JDBC 程式變得更容易管理，並且提供宣告式交易管理機制
* ORM 模組: 提供整合 OR mapping framework 的功能，可以整合JDO, Hibernate, JPA等，Spring 的 transaction 管理功能支援上述 OR mapping framework 與 JDBC
* Web 模組: 提供整合 Web 應用程式(Servlet)、及各種 Web Framework(例如Struts)的功能
* Web MVC 模組: Spring 提供的 Web 應用程式 MVC Framework，讓 Web 應用程式具備 IoC 功能，並且可以隨時抽換 View 與 Controller 元件
