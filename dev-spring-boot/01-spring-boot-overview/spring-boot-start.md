# Spring Boot Start

- Make it easier to get started with Spring development
  使Spring開發更容易
- Minimize the amount of manual configuration
  最大限度減少必須執行的手動配置
- Perform auto-configuration based on props files and JAR classpath
  根據屬性文件&JAR路徑執行的自動配置
- Help to resolve dependency conflicts (Maven or Gradle)
  解決依賴關係衝突
- Provide an embedded HTTP server so you can get started quickly
  提供嵌入式HTTP服務器

## spring initializr

https://start.spring.io/

1. Quickly create a starter Spring Boot project
2. Select your dependencies
3. Creates a Maven/Gradle project
4. Import the project into your IDE


# Maven

- Tell Maven the projects you are working with (dependencies)
- Maven will go out and download the JAR files for those projects for you
- And Maven will make those JAR files available during compile/run
- Think of Maven as your friendly helper

Benefits
- find JAR files for you
- Building and Running your Project
- Standard directory structure


## POM file

Project Meta File目標對象模型 - Configuration file for your project

|     Structure     |                   Using                   |
| :---------------: | :---------------------------------------: |
| projcet meta data |     Project name, version, file type      |
|   dependencies    |        list of projects depend on         |
|     plug ins      | additional custom tasks to run 額外自定義 |

### projcet meta data

Project Coordinates uniquely identify a project

|    Name     |                                       Description                                       |
| :---------: | :-------------------------------------------------------------------------------------: |
|  Group ID   |                         Name of company, group, or organization                         |
| Artifact ID |                                  Name for this project                                  |
|   Version   | A specific release version -> If project is under active development then: 1.0-SNAPSHOT |

### dependencies

- add given dependency project we need
- May see this referred to as: **GAV**

可以用來找Maven的dependencies
https://central.sonatype.com/?smo=true

### Spring Boot Starters

- A curated list of Maven dependencies
- Collection of dependencies grouped together
- It have been Tested and verified by the Spring Development team
- Reduces the amount of Maven configuration

### Full list
https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.build-systems.starters

### Spring Boot DevTool

- Automatically restarts your application when code is updated
- Simply add the dependency to your POM file

+ POM file
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

#### more details

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.devtools

### Spring Boot Actuator

+ Exposes endpoints to monitor and manage your application
+ Simply add the dependency to your POM file
+ Automatically exposes endpoints for metrics out-of-the-box
+ Endpoints are prefixed with: **/actuator**

- POM file
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- application.properties

expose all actuator endpoints
```
# Use wildcard "*" to expose all endpoints
# Can also expose individual endpoints with a comma-delimited list
#
management.endpoints.web.exposure.include=*
```

expose 特定 actuator endpoints
```
management.endpoints.web.exposure.include=health,info
management.info.env.enabled=true
```

|     Name     |                          Description                           |
| :----------: | :------------------------------------------------------------: |
|   /health    |           Health information about your application            |
|    /info     | gives information about your application **Default is empty**  |
| /auditevents |               Audit events for your application                |
|    /beans    | List of all beans registered in the Spring application context |
|  /mappings   |               List of all @RequestMapping paths                |


#### To expose `/info`

- application.properties
```
info.app.name=(My Super Cool App)
info.app.description=(A crazy and fun app, yoohoo!)
info.app.version=(1.0.0)
```

#### Full list
https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints

### Spring Boot Actuator - Security

+ You may NOT want to expose all of this information

- POM file
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

can override default user name and generated password
- application.properties
```
spring.security.user.name=...
spring.security.user.password=...
```

`/health` is still available. You can disable it if you want
- application.properties
```
# Exclude individual endpoints with a comma-delimited list
#
management.endpoints.web.exposure.exclude=health
```

#### more details

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#actuator


## Application Properties

### Costom 
- By default, Spring Boot reads information from a standard properties file
- define ANY custom properties in this file
- access properties using @Value

1. application properties
```
# Define custom properties
#
coach.name=(Mickey Mouse)
team.name=(The Mouse Club)
```
2. Inject Properties into app
```
@RestController
public class FunRestController {
    // inject properties for: coach.name and team.name
    
    @Value("${coach.name}")
    private String coachName;
    
    @Value("${team.name}")
    private String teamName;
}
```

### Spring Boot 

- set server port, context path, actuator, security etc

for example 1 - Logging

[logging](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)

```
# Log levels severity mapping
logging.level.org.springframework=DEBUG
logging.level.org.hibernate=TRACE
logging.level.com.luv2code=INFO

# Log file name 
logging.file.name=my-crazy-stuff.log
logging.file.path=c:/myapps/demo
```

Logging Levels ->TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

for example 2 - Web Properties

```
# HTTP server port
server.port=7070

# Context path of the application
server.servlet.context-path=/my-silly-app

# Default HTTP session time out
server.servlet.session.timeout=15m
```

for example 3 - Actuator Properties

```
# Endpoints to include by name or wildcard
management.endpoints.web.exposure.include=*

# Endpoints to exclude by name or wildcard
management.endpoints.web.exposure.exclude=beans,mapping

# Base path for actuator endpoints
management.endpoints.web.base-path=/actuator
```

for example 4 - Security Properties

```
# Default user name
spring.security.user.name=admin

# Password for default user
spring.security.user.password=topsecret
```

for example 5 - Data Properties
```

# JDBC URL of the database
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce

# Login username of the database
spring.datasource.username=scott

# Login password of the database
spring.datasource.password=tiger
```

### more details

https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties

## Static Content

- By default, Spring Boot will load static resources from "/static" directory

Examples of static resources HTML files, CSS, JavaScript, images, etc

**WARNING:**
Do not use the `src/main/webapp` directory if your application is packaged as a `JAR`.
Although this is a standard Maven directory, it works only with `WAR` packaging. 
It is silently ignored by most build tools if you generate a `JAR`

## Templates

Spring Boot includes auto-configuration for following template engines

- FreeMarker
- Thymeleaf
- Mustache

-----

## run app use in **CMD**

1. 先確認環境
```
echo %JAVA_HOME%
java --version
```

2. 開始打包資料

在*當前*資料夾打包
```
mvnw package
```

他會建立一個`target\testdemo-0.0.1-SNAPSHOT.jar`的資料

3. 啟動Spring

```
java -jar <資料夾名稱>

例:
java -jar target\testdemo-0.0.1-SNAPSHOT.jar
```

## run app use in **Spring Boot Maven Plugin**

```
mvnw spring-boot:run
```

