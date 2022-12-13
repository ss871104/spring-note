# **AOP (Aspect-Oriented Programming) 面向切面**

## **Index 目錄**
* [AOP 介紹](#aop-介紹)
* [AOP 底層](#aop-底層)
* [AOP 動態代理流程](#aop-動態代理流程)
* [AspectJ annotation](#aspectj-annotation)

## **AOP 介紹**
AOP (Aspect Oriented Programming)，面向切面
* 在應用程式的企業邏輯流程中，常常需要執行與企業邏輯無關、純粹為了系統運作考量的程式邏輯(例如：Log, Security, Transaction等)，這些純粹系統運作考量的程式邏輯叫做Cross-cutting-concerns(橫跨所有企業邏輯流程的意思)
* 讓企業邏輯與純粹系統運作考量的程式邏輯(Cross-cutting-concerns)彼此混在相同的類別、甚至相同方法內，會造成程式難以維護
* 從重複利用(Reuse)的角度來看，將 Cross-cutting-concerns 獨立出來，在應用程式的企業邏輯開發完成後，再依據需求套用到企業邏輯流程是比較好的做法

AOP目的：
* 降低耦合度
* 提高重用性

---

## **AOP 底層**
AOP 的底層使用動態代理，有兩種情況代理
1. 有介面的情況，JDK 動態代理(基於實作介面)
    * 創建介面的實作類別，透過實作實現 AOP
2. 沒有介面的情況，CGLIB 動態代理(基於繼承)
    * 對代理的目標類別創建子類別，透過繼承實現 AOP

### **JDK 動態代理**
使用Proxy類別裡的方法建立代理物件 (java.lang.reflect.Proxy)
* 呼叫 newProxyInstance 方法，內有三個參數
    * ClassLoader loader: 類別加載器
    * Class<?>[] interfaces: 縫合方法的實作類別的介面，支持多個介面
    * InvocationHandler h (是個介面): 實作 nvocationHandler 介面，建立代理物件，寫縫合的方法

程式碼範例：
* 介面
```java
public interface UserDao {
    public int add(int a, int b);

}
```
* 實作類別
```java
public class UserDaoImpl implements UserDao {
    @Override
    public int add(int a, int b) {
        System.out.println("add方法執行...");
        return a+b;
    }

}
```
* JDK代理類別
```java
public class JDKProxy {

    public static void main(String[] args) {
        // 創建介面實現類別的代理物件
        Class[] interfaces = {UserDao.class};

        UserDaoImpl userDao = new UserDaoImpl();
        UserDao dao = (UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));

        int result = dao.add(1, 2);
        System.out.println("result: " + result);
    }
}

// 創建實作InvocationHandler介面的類別
class UserDaoProxy implements InvocationHandler {

    // 透過有參數建構子把物件傳遞過來
    private Object obj;
    public UserDaoProxy(Object obj) {
        this.obj = obj;
    }

    // 縫合的邏輯
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        // 方法之前
        System.out.println("方法之前執行..." + method.getName() + ": 傳遞的參數" + Arrays.toString(args));

        // 被縫合的方法執行
        Object res = method.invoke(obj, args);

        // 方法之後
        System.out.println("方法之後執行...");

        return res;

    }
}
```
```
Output:
方法之前執行...add: 傳遞的參數[1, 2]
add方法執行...
方法之後執行...
result: 3
```

---

## **AOP 動態代理流程：**
![AOPConcept](/spring-framework/AOPConcept-3.jpg)<br>
Retrieved from https://openhome.cc/Gossip/SpringGossip/AOPConcept.html

AOP 名詞解釋：
* AOP 著重在 Cross-cutting concerns(又稱 Advice)的實作，以及將 Aspect 縫合(Weave)到企業邏輯流程的時機
* JoinPoint: 在企業邏輯流程中程式執行的某個時間點，例如：某個方法被呼叫、某個 Exception 被丟出
* Advice: 要在某個 JoinPoint 執行的程式碼，就是純粹系統運作考量程式邏輯(Cross-cutting concerns)的程式碼
* PointCut: Advice 要在哪些時間點執行的相關設定，例如：在 1 個類別的所有方法執行之前執行
* Weave: Advice 被應用至物件之上的過程稱之為縫合（Weave），在 AOP 中縫合的方式有幾個時間點：編譯時期（Compile time）、類別載入時期（Classload time）、執行時期（Runtime）。
* Aspect: Advice(程式) + PointCut(設定)，表示在應用程式的某些時間點要執行的動作

#### **Advice: Aspect 的具體實作程式**
依據將 Advice 縫合(Weave)到主要企業邏輯流程的時機，Spring 架構提供多個不同功能的 Advice
* before advice: 在 JoinPoint 之前執行的 Advice
* after returning advice: 在 JoinPoint 正常完成(例如方法正常結束沒有發生 exception)之後執行的 Advice
* after throwing advice: 在 JoinPoint 不正常完成(例如 exception 無法正常結束)時執行的 Advice
* after(finally) advice: JoinPoint 結束時(不論正常還是不正常結束)執行的 Advice
* around advice: 包在 JoinPoint 前後，可控制是否執行 JoinPoint 的 Advice

**around advice: 最強大的一種 Advice**
* 可以在 JoinPoint 前後執行特定程式碼：在前半部分程式執行完畢之後可以選擇是否呼叫 JoinPoint，在 JoinPoint 執行完畢之後再執行後半部分程式<br>
**注意：** 呼叫 ProceedingJoinPoint 的 proceed()方法執行原本 JoinPoint 的程式(通常是一個方法)，proceed()方法的回傳值就是原本 JoinPoint 的程式的執行結果(方法回傳值)
* 可以自行決定回傳值：回傳 JoinPoint 的回傳值、回傳 Advice 自訂的回傳值、甚至丟出 exception 結束執行<br>
**注意：** before advice 與 after advice 無法影響 JoinPoint 的執行結果，而 around advice 可以取代或是修改 JoinPoint 的執行結果

---

## **AspectJ annotation**
Spring framework 一般是基於 AspectJ 來實現 AOP
* AspectJ 為獨立框架，和 Spring 框架一起使用實現 AOP
* 可使用 xml 組態檔或 annotation 實作，通常是使用 annotation

### **Spring 對 AOP的支援**
**annotation 組態設定**
* @EnableAspectJAutoProxy: 使用在 Java 類別組態，要求 Spring 架構搜尋並註冊所有使用 @Aspect 宣告的類別
* @Aspect: 使用在類別上，註明此類別是一個 Aspect
* @Pointcut: 使用在 Aspect 類別的方法上，設定各種 advice 的作用時機
    * @Pointcut 方法: 回傳型別必須是 void、方法名稱是 pointcut 名稱
    * @Pointcut 的 value 屬性: 定義符合條件的 advice 作用時機(JoinPoint)，也就是設定哪個、或是哪幾個方法執行的時候要啟動 AOP 機制
    * @Pointcut 的 value 屬性語法: ?代表非必要欄位

**execution(modifiers? returnType declareType?.name(params) throw?)**
* modifiers: 方法修飾字
* returnType: 方法回傳型別
* declareType: 方法類別的全名(套件名稱.類別名稱)
* name: 方法名稱
* params: 方法所宣告的參數
* throws: 方法所宣告的 exception
* 注意：使用*代表任意字元、使用..代表任意數量的參數

**用於宣告 Advice 的 annotation**
* @Before: 用來宣告 JoinPoint 之前執行的 Advice
* @After: 用來宣告 JoinPoint 之後執行的 Advice
* @Around: 用來宣告 JoinPoint 之前和之後都執行的 Advice
* @AfterReturning: 透過 returning 屬性宣告代表 JoinPoint 回傳值的參數名稱
* @AfterThrowing: 透過 throwing 屬性宣告代表 JoinPoint 發生例外的參數名稱

程式碼範例(annotation)
* 建立類別
```java
public class User {
    public void add() {
        System.out.println("add...");
    }
}
```
* 建立切面
```java
@Component
@Aspect // 生成代理物件
@Order(1) // 若是有其他切面同時縫合同一個方法，可利用@Order註解排順序
public class UserProxy {

    // 設定切入點(pointcut)
    @Pointcut(value = "execution(* com.ss871104.spring.aop.User.add())")
    public void pointcut() {

    }

    // before advice
    @Before("pointcut()")
    public void before() {
        System.out.println("before...");
    }

    // after advice
    @After("pointcut()")
    public void after() {
        System.out.println("after...");
    }

    // after returning advice
    @AfterReturning("pointcut()")
    public void afterReturning() {
        System.out.println("after returning...");
    }

    // after throwing advice
    @AfterThrowing("pointcut()")
    public void afterThrowing() {
        System.out.println("after throwing...");
    }

    // around advice
    @Around("pointcut()")
    public void around(ProceedingJoinPoint proceedJoinPoint) throws Trowable{
        System.out.println("before around...");

        // 原方法執行
        proceedJoinPoint.proceed();

        System.out.println("after around...");
    }
}
```
```
Output:
before around...
before...
add...
after around...
after...
after returning...
```
