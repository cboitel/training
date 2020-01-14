# Spring

## Pre-requisites
* [Java](java.md)
* [Maven](maven.d) (or Gradle)
* Optional : [VSCode](vscode.md) with Java extension pack + Spring Initializr + Maven for java extensions

## Objectives
* Build a sample Spring Cloud Gateway
* Modify its base configuration through configuration files and spring profiles
* Extend it with production ready features (Add Spring Boot Actuator)
* Extend it with Spring Cloud Config (Ability to download its configuration from a configuration central server)

## Build your initial Spring project
To build a Spring project, simply initialize one by using Spring's initializer:

* Through its dedicated web site
* or using Spring Initializr VS Code extension

### Option 1: using Spring Initializer Web site
 
Connect to [https://start.spring.io](https://start.spring.io) and choose:

* to build a maven project
* based on Java language
* using latest Spring boot release (avoid using SNAPSHOTs unless you know what u r doing)
* Project metadatas:
    * Group: fr.fdj.techlab.training
    * Artifact: demogw
* Packaging: leave JAR
* Java version : match it with your Java Release
* Dependencies: look for Gateway and add it
 
Then generate project 

* this will generate a zip files with project initialized in it
* Download it and install it in a working directory of your choice
 
> In the remaining of this document, we will assume chosen directory was %USERPROFILE%\java\training\demogw on a windows desktop or $HOME/java/training/demogw.
> If you chose a different directory, adapt it in provided commands.
 
### Option 2: using VSCode extension

Install Spring Initializr extension and access command palette (Shift+Ctrl+P or View/Command palette menus):

* Works similarly to web site
* at the end select/create directory in which generated project will be stored

## Build and run initial project

Open a windows command prompt and cd to directory where project was expanded: package the jar file and run it

```shell
    cd %USERPROFILE%\java\training\gwdemo && 
    mvn package &&
    FOR %J in (target\gwdemo*.jar) DO java -jar %J
```

## Configure your spring cloud gateway
Simply edit application.properties in src\main\ressources or create one under a config directory where you lanch your application.

### Option #1: update ressources inside your project
Spring Initializer has generated an empty default application.properties configuration file under src\main\resources.
 
We will be using a YAML configuration file so we will first delete the application.properties
```shell
cd %USERPROFILE%\java\training\gwdemo\src\main\resources && 
del application.properties
```

Copy/paster the following content inside application.yaml in the same directory:
```YAML
spring:
  cloud:
    gateway:
      routes:
      - id: demo
        uri: http://httpbin.org
        predicates:
        - Host={segment}.httpbin.org
        - Method=POST,GET
        filters:
        - SetRequestHeader=X-Configd-Demo,{segment}
        - RemoveRequestParameter=demo
```

This configuration will define a route:

* any request matching the predicate section
    * GET or POST HTTP requests
    * Host header in the form of _somename.httpbin.org_
* will be forwarded to httpbin.org web site:
    * any demo qery string parameter will be removed first 
    * HTTP header named "X-Config-Demo" will be added/set to _samename_ extracted from the host header original request 

Since our example requires access to Internet, you may have to configure your gateway to use your proxy configuration to be used by gateway by adding the following configuration:
```YAML
spring:
    cloud:
        ...
        httpclient:
            proxy:
                host: <i>*your_proxy_host*</i>
                port: <i>*your_proxy_port*</i>
                non-proxy-hosts-pattern: 127.0.0.1|localhost
```
Now, package again the application and run it

### Option #2: configure externally

Create a sub-directory named config and create a file named application.yaml containing same information as in option #1 above.
 
Run again your application jar from the location where your created your config directory and test
```
# will work and should report added X-Config-Demo header
curl --header "Host: somename.httpbin.org" http://127.0.0.1:8080/get
# will not work (does not match any route predicates)
curl --header "Host: somename.unknown.org" http://127.0.0.1:8080/get
```

# Extend with Spring's "production ready" features

Spring Actuator is the standard way and is very simple to implement

1. Add actuator dependency to your project's pom.xml file
    ```XML
    <dependencies>
        
        <!-- other dependencies -->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    ```
1. Update your configuration application properties to enable actuator endpoints 
    ```YAML
    spring:
        cloud:
            ...
            metrics:
                enabled: true
    
    management:
        server:
            address: 127.0.0.1
            port: 8079
        endpoint:
            shutdown:
                enabled: true
        endpoints:
            web:
                base-path: /actuator
                exposure:
                    include: "*"
    ```
    
    This will configure actuator to expose itself on local host IP on port 8079 and expose all endpoints including shutdown one

1. run and test
    ```shell
    # display exposed acturtor endpoints
    curl http://127.0.0.1/actuator/
    # get health status
    curl http://127.0.0.1/actuator/health
    # get info
    curl http://127.0.0.1/actuator/info
    # get gateway routes
    curl http://127.0.0.1/actuator/gateway/routes
    # gracefull application shutdown
    curl -X POST http://127.0.0.1/actuator/shutdown
    ``` 
    
Now let customize things a bit further:

1. Include some information from application's properties in actuator info endpoint
    1. add application information in application properties file and restart it 
        ```YAML
        info:
            somekey: somevalue we want to expose

        management:
            endpoint:
                ...
                health:
                    show-components: always
        ```

1. Automatically include build information with maven by updating project's pom.xml:
    ```XML
    <!-- .... -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                        <configuration>
                			<additionalProperties>
								<spring-cloud.version>${spring-cloud.version}</spring-cloud.version>
                    			<project.description>${project.description}</project.description>
                			</additionalProperties>
            			</configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <!-- ... -->
    ```

1. rebuild, restart and test again
    ```shell
    # get health
    curl http://127.0.0.1/actuator/health
    # get info
    curl http://127.0.0.1/actuator/info
    # httptrace
    curl http://127.0.0.1/actuator/httptrace
    ``` 

Now, have fun configuring logging, pid file generation following actuator documentation...

# To go further
* [Spring common properties](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)
* [Spring boot how-to](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html)
* [Spring Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html)
* Spring Cloud Gateway:
    * [Doc index](https://cloud.spring.io/spring-cloud-gateway/reference/html/index.html)
    * [Route Predicates](https://cloud.spring.io/spring-cloud-gateway/reference/html/index.html#gateway-request-predicates-factories)
    * [Spring Cloud Gateway Route Filters](https://cloud.spring.io/spring-cloud-gateway/reference/html/index.html#gatewayfilter-factories)
    * [Spring cloud Gateway properties](https://cloud.spring.io/spring-cloud-gateway/reference/html/appendix.html)
