# **spring-framework**

## **Index 目錄**
* [Intro 簡介](#intro-簡介)
* [IoC Container 控制反轉容器](/spring-framework/IoC.md)
* [AOP 面向切面](/spring-framework/AOP.md)
* [JdbcTemplate](/spring-framework/JdbcTemplate.md)

## **Intro 簡介**

### **什麼是Spring框架？**

1. Spring 是輕量級的開源的JavaEE框架，不需要Java EE Application Server就可以執行

2. Spring 有兩大核心：IoC 和 AOP
    * **IoC (Inversion of Control) 控制反轉**: 把建立物件的過程交給Spring進行管理，目的在降低應用程式的元件與元件之間的相依關係，進而降低耦合度，讓每個元件的可重用性提高，讓應用程式更具延展性
    * **AOP (Aspect-Oriented Programming) 面向切面**: 不更改原始程式碼的情況下插入特定功能的程式碼，通常需要DI的輔助來實作

3. Spring 特點
    * 降低耦合度，方便開發 (loose coupling)
    * AOP 編寫支持
    * 方便程式碼測試
    * 方便合其他框架進行整合
    * 方便進行事務操作
    * 降低API開發難度

### **Spring Framework Runtime**
 
 ![spring-framework](https://docs.spring.io/spring-framework/docs/4.3.x/spring-framework-reference/htmlsingle/images/spring-overview.png)<br>
 Reference: Spring Framework Reference Documentation

#### **Spring Framework 的各個模組**
* Core Container: 提供Spring Framework的基礎功能，主要元件是BeanFactory, BeanFactory實作Factory design pattern, 可以管理物件的生成、釋放以及物件的依賴關係
* Context模組: 提供讀取Spring Framework組態設定檔的功能
* AOP模組: 提供Spring Framework的AOP功能
* JDBC模組: 提供JDBC相關功能，讓繁瑣的JDBC程式變得更容易管理，並且提供宣告式交易管理機制
* ORM模組: 提供整合OR mapping framework的功能，可以整合JDO, Hibernate, JPA等，Spring的transaction管理功能支援上述OR mapping framework與JDBC
* Web模組: 提供整合Web應用程式(Servlet)、及各種Web Framework(例如Struts)的功能
* Web MVC模組: Spring提供的Web應用程式MVC Framework，讓Web應用程式具備IoC功能，並且可以隨時抽換View與Controller元件
