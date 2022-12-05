# **spring-mvc**

## **Index 目錄**
* [Intro 簡介](#intro-簡介)
* [SpringMVC 工作流程](#springmvc-工作流程)

## **Intro 簡介**

### **什麼是Spring Web MVC？**
Spring Web MVC是一種基於Java實現Web MVC設計模式的請求驅動類型的輕量級Web框架，即使用了MVC架構模式的思想，將web層進行職責解耦，基於請求驅動指的就是使用請求-響應模型，框架的目的就是幫助我們簡化開發。
* Spring Web MVC功能建構在Servlet, JSP規格之上，必須Web Container的支援才能運作
* Spring Web MVC實作Front Controller Design Pattern，使用IoC機制，將Model, Controller, View之間的相依關係斷開，讓程式設計師可以隨時修改或是替換View, Controller與Model
* Spring Web MVC將Web應用程式中連接View元件與Controller元件的程式碼(View呼叫Controller, Controller導向View)抽離出來，提供Spring的制式化實作，讓程式設計師專注在企業邏輯上

---

## **SpringMVC 工作流程**