# Spring Core

# Spring Container

- XML configuration file (legacy)
- Java Annotations (modern) 註解定義
- Java Source Code (modern) Java類提供

# Ioc(Inversion of Control) 控制反轉

控制: 控制指的是控制外部資源的獲取、控制物件的生命週期

反轉: 剛開始流行的是由開發人員操縱一切，現在變了，由Spring框架來控制程式中的外部資源、控制物件的生命週期等。
所以取名`反轉`，即控制的權利由開發人員轉移到了Spring框架。由Spring框架幫我建立我們物件中依賴的物件，我們的物件只能被動的接受

Ioc的好處就是解耦，物件和物件之間的耦合度變低了，便於測試、便於功能複用

# DI(Dependency Injection)依賴注入

依賴: 我的A物件中引用了B物件，也就是說A物件依賴B物件。你要通過配置檔案告訴Spring你的物件之間的依賴關係。

注入: 你的物件已經交給Spring管理了，你也告訴Spring你的物件之間的依賴關係了，那麼在合適的時候，由Spring把你依賴的其他物件(或者資源、常量等)注入給你。

總結就是，把所有的**控制權交給Spring**，由Spring幫你建立物件、幫你維護物件之間的依賴關係


>建立new的控制權從我們反轉(交給)Spring，稱為控制反轉(IoC)
>在依賴實例的地方，Spring自動會new出來，並幫我們注入，稱依賴注入。並在不需要的時候，自動收回。
>**因此會常聽到，用DI來實現IoC**

## DI種類

- constructor injection建構子注入: 透過一個 `class` constructor 提供 dependencies
- setter injection設值方法注入: Client 開放一個 `setter method` 讓 injector 使用
- interface injection介面注入: Dependency 會提供一個 `injector method`， client 會將他傳進來， 然後他再 inject dependency
  Clients 一定會實作一個有公開 setter method 的 interface

DI 負責:
1. 建立 objects
2. 了解那些 classes 需要哪些 objects
3. 提供所有的 objects


# constructor injection 建構子注入

## Test demo
1. Define the dependency interface and class

```
//建立一個interface

package com.luv2code.springcoredemo;

public interface Coach {
    String getDailyWorkout();
}
```

2. Create Demo REST Controller

```
//建立一個class

package com.luv2code.springcoredemo;

import org.springframework.stereotype.Component;

@Component
public class CricketCoach implements Coach{
    @Override
    public String getDailyWorkout() {
        return "Practice 15 min.";
    }
}
```

3. Create a constructor in your class for injections
4. Add @GetMapping for /dailyworkout

```
@RestController
public class DemoController {
    
    //3.
    // define a private field for the dependency
    private Coach myCoach;

    // define a constructor for dependency injection
    @Autowired  // tells Spring to inject a dependency
    public DemoController(Coach theCoach) {
        myCoach = theCoach;
    }
    
    //4.
    @GetMapping("/dailyworkout")
    public String getDailyWorkout() {
        return myCoach.getDailyWorkout();
    }
}
```
## Constructor Injection Behind the Scenes

- Creates application context 
- registers all beans
- Starts the embedded server

`Spring Framework`
```
Coach theCoach = "new" CricketCoach();
DemoController demoController = new DemoController(theCoach);
```

## @Component

是一個註釋，它允許Spring自動檢測我們的自定義bean

- Automatically register the beans in the Spring containe
- marks the class as a Spring Bean(成為DI的候選對象)
- makes the bean available for dependency injection

`File: SpringcoredemoApplication.java`
```
package com.luv2code.springcoredemo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringcoredemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcoredemoApplication, args);
    }
}
```

`@SpringBootApplication` is composed of the following annotations

|        Annotation        |                                    Description                                     |
| :----------------------: | :--------------------------------------------------------------------------------: |
| @EnableAutoConfiguration |                  Enables Spring Boot's auto-configuration support                  |
|      @ComponentScan      | Enables component scanning of current package. Also recursively scans sub-packages |
|      @Configuration      |   Able to register extra beans with @Bean or import other configuration classes    |

- By default, scanning from **same package** as your main Spring Boot application (Also scans sub-packages)
- Allows you to leverage default component scanning, no need to explicitly reference the base package name
  定義了一個基本搜尋包，利用預設組件掃描，無需明確引用基礎套件名稱

![component-scanning](https://hackmd.io/_uploads/HJGrRLsSa.png)

### 如何讓Spring Boot找到其他package(紅色框框)

![component-scanning-outside](https://hackmd.io/_uploads/ryCwCUoBT.png)

`File: SpringcoredemoApplication.java`

```
@SpringBootApplication(
		scanBasePackages = {"com.luv2code.springcoredemo",
				     "com.luv2code.util",
                                    "增加外部資料夾來源"})
```

# setter injection 設值方法注入

Inject dependencies by `calling setter method(s)` on your clas

## Test demo
1. Create setter method(s) in your class for injections

`DemoController`
```
@RestController
public class DemoController {
    private Coach myCoach;
    
    public void setCoach(Coach theCoach) {
        myCoach = theCoach;
    }
}
```


2. Configure the dependency injection with @Autowired Annotation

`DemoController`
```
@RestController
public class DemoController {
    private Coach myCoach;
    
    @Autowired
    public void setCoach(Coach theCoach) {
        myCoach = theCoach;
    }
}
```

*behind the scenes -> Spring Framework*

```
Coach theCoach = new CricketCoach();
DemoController demoController = new DemoController();
demoController.setCoach(theCoach);
```

# Field Injection 屬性注入

Inject dependencies by setting field values on your class directly (even private fields)

## Test demo
**Accomplished by using Java Reflection**

`DemoController`
```
package com.luv2code.springcoredemo;
import org.springframework.beans.factory.annotation.Autowired;

@RestController
public class DemoController {

    @Autowired
    private Coach myCoach;
     
    // no need for constructors or setters
     
    @GetMapping("/dailyworkout")
    public String getDailyWorkout() {
        return myCoach.getDailyWorkout();
    }
}
```


# @AutoWiring

對class成員變量、方法及構造函數進行標註，完成自動裝配的工作

應用於`建構函數`

```
public class MovieRecommender {
 
    private final CustomerPreferenceDao customerPreferenceDao;
 
    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

}
```

應用於`setter 方法`

```
public class SimpleMovieLister {
 
    private MovieFinder movieFinder;
 
    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
 
}
```

# @Qualifier

- if we have multiple implementations，定義要用哪個bean
- Same name as class, first character **lower-case**
- has higher priority than @Primary

## constructor injection

```
@Autowired
public DemoController(@Qualifier("cricketCoach") Coach theCoach) {
    myCoach = theCoach;
}
```

## setter injection

```
@Autowired
public void setCoach(@Qualifier("cricketCoach") Coach theCoach) {
    myCoach = theCoach;
}
```

# @Primary

- @Qualifier 的 Alternate solution available替代方案
- If there are multiple coaches, then you figure it out
- **only one can use**

## Test demo

No need for @Qualifier, there is a @Primary coach

```
package com.luv2code.springcoredemo.common;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

@Component
@Primary
public class TrackCoach implements Coach {

    @Override
    public String getDailyWorkout() {
        return "Run a hard 5k!";
    }
}
```

# @Lazy 延遲初始化

- Instead of creating all beans up front
- A bean will only be initialized `in needed for dependency injection` or `it is explicitly requested`
- is disabled by default

*Advantages*
+ Only create objects as needed
+ May help with faster startup time if you have large number of components

*Disadvantages*
- If you have web related components like @RestController, not created until requested
- May not discover configuration issues until too late
- Need to make sure you have enough memory for all beans once created


## Global configuration
All beans are lazy, no beans are created until needed

`application.properties`

```
spring.main.lazy-initialization=true
```

For dependency resolution:
1. Spring creates CricketCoach first 
2. Then creates  DemoController 
3. injects the CricketCoach

```
In constructor: CricketCoach
In constructor: DemoController
```

## Private configuration

```
@Component
@Lazy
public class TrackCoach implements Coach{

    public TrackCoach(){
        System.out.println("In constructor: " + getClass().getSimpleName());
    }

    @Override
    public String getDailyWorkout() {
        return "Run a hard 5K!";
    }
}
```

# Bean Scopes

- Default scope is singleton 單例 
  Spring Container creates only one instance of the bean
- All dependency injections for the bean reference the **SAME** bean

## Additional Bean Scopes

|     Scope      |                          Description                          |
| :------------: | :-----------------------------------------------------------: |
|   singleton    | Create a single shared instance of the bean **Default scope** |
|   prototype    |    Creates a new bean instance for each container request     |
|    request     |     Scoped to an HTTP web request. Only used for web apps     |
|    session     |     Scoped to an HTTP web session. Only used for web apps     |
| global-session |  Scoped to a global HTTP web session. Only used for web apps  |

## singleton

- Both point to the same instance

```
@RestController
public class DemoController {
    private Coach myCoach;
    private Coach anotherCoach;
    
    @Autowired
    public DemoController(
                @Qualifier("cricketCoach") Coach theCoach, 
                @Qualifier("cricketCoach") Coach theAnotherCoach) {
        myCoach = theCoach;
        anotherCoach = theAnotherCoach;
    }
    
    @GetMapping("/check")
    public String check(){
        return "Comparing beans: myCoach == anotherCoach, " + (myCoach == anotherCoach);
    }
}

->瀏覽器會顯示Comparing beans: myCoach == anotherCoach, true

```

![singleton](https://hackmd.io/_uploads/By3_A_b8a.png)


## prototype

`CricketCoach`
```
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)

->瀏覽器會顯示Comparing beans: myCoach == anotherCoach, false
```

![prototype](https://hackmd.io/_uploads/SyOjAO-LT.png)

# Bean Lifecycle Methods

- You can add custom code during **bean initialization**, **bean destruction**

## bean initialization (Init)

`CricketCoach`
```
@PostConstruct
public void doMyStartupStuff(){
    System.out.println("In doMyStartupStuff():" + getClass().getSimpleName());
}

->執行程式流程
In constructor: BaseballCoach
In constructor: CricketCoach
In doMyStartupStuff():CricketCoach
In constructor: TennisCoach
In constructor: TrackCoach
In constructor: DemoController
```

## bean destruction (Destroy)

**For "prototype" scoped beans, Spring does not call the destroy method**

`CricketCoach`
```
public void doMyCleanupStuff(){
    System.out.println("In doMyCleanupStuff():" + getClass().getSimpleName());
}

-> 關閉程式後，銷毀生命週期
In doMyCleanupStuff():CricketCoach

Process finished with exit code 130
```

# Configuring Beans with Java Code

- inject it into other services of our application
- Make an existing third-party class available to Spring framework

假設SwimCoach沒有`@Component`，可以用`@Bean`替代

## demo

1. `SwimCoach`
```
public class SwimCoach implements Coach{

    public SwimCoach(){
        System.out.println("In constructor: " + getClass().getSimpleName());
    }
    
    @Override
    public String getDailyWorkout() {
        return "Swim 1000 meters as a warm up!";
    }
}
```
如果直接執行會報錯，找不到SwimCoach，因此需要以下步驟

2. 建立package -> springcoredemo.config
   加入`@Configuration`, `@Bean`
   
`SportConfig`
```
package com.luv2code.springcoredemo.config;

import com.luv2code.springcoredemo.common.Coach;
import com.luv2code.springcoredemo.common.SwimCoach;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SportConfig {

    @Bean
    public Coach swimCoach(){
        return new SwimCoach();
    }
}
```

-> 簡單的將SwimCoach用`@Bean`配置為springbean

## bean ID自定義

bean ID 預設為 class 檔名，可以用以下方法自定義

`SportConfig`
```
@Bean("自定義名稱擺放位置")

public Coach swimCoach(){
    return new SwimCoach();
    }
```