# **AOP (Aspect-Oriented Programming) 面向切面**

## **Index 目錄**
* [AOP 介紹](#aop-介紹)
* [AOP 底層](#aop底層)
* [AOP 動態代理流程](#aop-動態代理流程)
* [AspectJ annotation](#aspectj-annotation)

## **AOP 介紹**
AOP (Aspect Oriented Programming)，面向切面
* 在應用程式的企業邏輯流程中，常常需要執行與企業邏輯無關、純粹為了系統運作考量的程式邏輯(例如：Log, Security, Transaction等)，這些純粹系統運作考量的程式邏輯叫做Cross-cutting-concerns(橫跨所有企業邏輯流程的意思)
* 讓企業邏輯與純粹系統運作考量的程式邏輯(Cross-cutting-concerns)彼此混在相同的類別、甚至相同方法內，會造成程式難以維護
* 從重複利用(Reuse)的角度來看，將Cross-cutting-concerns獨立出來，在應用程式的企業邏輯開發完成後，再依據需求套用到企業邏輯流程是比較好的做法

AOP目的：
* 降低耦合度
* 提高重用性

---

## **AOP底層**
AOP的底層使用動態代理，有兩種情況代理
1. 有介面的情況，JDK動態代理(基於實作介面)
    * 創建介面的實作類別，透過實作實現AOP
2. 沒有介面的情況，CGLIB動態代理(基於繼承)
    * 對代理的目標類別創建子類別，透過繼承實現AOP

### **JDK動態代理**
使用Proxy類別裡的方法建立代理物件 (java.lang.reflect.Proxy)
* 呼叫newProxyInstance方法，內有三個參數
    * ClassLoader loader: 類別加載器
    * Class<?>[] interfaces: 縫合方法的實作類別的介面，支持多個介面
    * InvocationHandler h (是個介面): 實作InvocationHandler介面，建立代理物件，寫縫合的方法

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

AOP名詞解釋：
* AOP著重在Cross-cutting concerns(又稱Advice)的實作，以及將Aspect縫合(Weave)到企業邏輯流程的時機
* JoinPoint: 在企業邏輯流程中程式執行的某個時間點，例如：某個方法被呼叫、某個Exception被丟出
* Advice: 要在某個JoinPoint執行的程式碼，就是純粹系統運作考量程式邏輯(Cross-cutting concerns)的程式碼
* PointCut: Advice要在哪些時間點執行的相關設定，例如：在1個類別的所有方法執行之前執行
* Weave: Advice被應用至物件之上的過程稱之為縫合（Weave），在AOP中縫合的方式有幾個時間點：編譯時期（Compile time）、類別載入時期（Classload time）、執行時期（Runtime）。
* Aspect: Advice(程式) + PointCut(設定)，表示在應用程式的某些時間點要執行的動作

#### **Advice: Aspect的具體實作程式**
依據將Advice縫合(Weave)到主要企業邏輯流程的時機，Spring架構提供多個不同功能的Advice
* before advice: 在JoinPoint之前執行的Advice
* after returning advice: 在JoinPoint正常完成(例如方法正常結束沒有發生exception)之後執行的Advice
* after throwing advice: 在JoinPoint不正常完成(例如exception無法正常結束)時執行的Advice
* after(finally) advice: JoinPoint結束時(不論正常還是不正常結束)執行的Advice
* around advice: 包在JoinPoint前後，可控制是否執行JoinPoint的Advice

**around advice: 最強大的一種Advice**
* 可以在JoinPoint前後執行特定程式碼：在前半部分程式執行完畢之後可以選擇是否呼叫JoinPoint，在JoinPoint執行完畢之後再執行後半部分程式<br>
**注意：** 呼叫ProceedingJoinPoint的proceed()方法執行原本JoinPoint的程式(通常是一個方法)，proceed()方法的回傳值就是原本JoinPoint的程式的執行結果(方法回傳值)
* 可以自行決定回傳值：回傳JoinPoint的回傳值、回傳Advice自訂的回傳值、甚至丟出exception結束執行<br>
**注意：** before advice與after advice無法影響JoinPoint的執行結果，而around advice可以取代或是修改JoinPoint的執行結果

---

## **AspectJ annotation**
Spring framework一般是基於AspectJ來實現AOP
* AspectJ為獨立框架，和Spring框架一起使用實現AOP
* 可使用xml組態檔或annotation實作，通常是使用annotation

### **Spring對AOP的支援**
**annotation組態設定**
* @EnableAspectJAutoProxy: 使用在Java類別組態，要求Spring架構搜尋並註冊所有使用@Aspect宣告的類別
* @Aspect: 使用在類別上，註明此類別是一個Aspect
* @Pointcut: 使用在Aspect類別的方法上，設定各種advice的作用時機
    * @Pointcut方法: 回傳型別必須是void、方法名稱是pointcut名稱
    * @Pointcut的value屬性: 定義符合條件的advice作用時機(JoinPoint)，也就是設定哪個、或是哪幾個方法執行的時候要啟動AOP機制
    * @Pointcut的value屬性語法: ?代表非必要欄位

**Pointcut切面的value**<br>
execution(modifiers? returnType declareType?.name(params) throw?)
* modifiers: 方法修飾字
* returnType: 方法回傳型別
* declareType: 方法類別的全名(套件名稱.類別名稱)
* name: 方法名稱
* params: 方法所宣告的參數
* throws: 方法所宣告的exception
* 注意：使用*代表任意字元、使用..代表任意數量的參數

**用於宣告Advice的annotation**
* @Before: 用來宣告JoinPoint之前執行的Advice
* @After: 用來宣告JoinPoint之後執行的Advice
* @Around: 用來宣告JoinPoint之前和之後都執行的Advice
* @AfterReturning: 透過returning屬性宣告代表JoinPoint回傳值的參數名稱
* @AfterThrowing: 透過throwing屬性宣告代表JoinPoint發生例外的參數名稱

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
