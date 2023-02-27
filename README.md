# Example Employee-Manager App

A sample Employee-Manager app to test Keploy integration capabilities using [SpringBoot](https://spring.io) and PostgreSQL.

## Pre-requisites

- [Java 8+](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.installing)

## Quick Installation

### **MacOS**

```shell
curl --silent --location "https://github.com/keploy/keploy/releases/latest/download/keploy_darwin_all.tar.gz" | tar xz -C /tmp

sudo mv /tmp/keploy /usr/local/bin && keploy
```

### **Linux**

<details>
<summary>Linux</summary>

```shell
curl --silent --location "https://github.com/keploy/keploy/releases/latest/download/keploy_linux_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/keploy /usr/local/bin && keploy
```

</details>

<details>
<summary>Linux ARM</summary>

```shell
curl --silent --location "https://github.com/keploy/keploy/releases/latest/download/keploy_linux_arm64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/keploy /usr/local/bin && keploy
```

The UI can be accessed at http://localhost:6789

</details>

### **Windows**

<details>
<summary>Windows</summary>

- Download the [Keploy Windows AMD64](https://github.com/keploy/keploy/releases/latest/download/keploy_windows_amd64.tar.gz), and extract the files from the zip folder.
- Run the `keploy.exe` file.

</details>

<details>
<summary>Windows ARM</summary>

- Download the [Keploy Windows ARM64](https://github.com/keploy/keploy/releases/latest/download/keploy_windows_arm64.tar.gz), and extract the files from the zip folder.
- Run the `keploy.exe` file.

</details>

## Build configuration

[Find the latest release](https://search.maven.org/artifact/io.keploy/keploy-sdk) of the Keploy Java SDK at maven
central.

Add *keploy-sdk* as a dependency to your *pom.xml*:

    `<dependency>`
      `<groupId>`io.keploy `</groupId>`
      `<artifactId>`keploy-sdk `</artifactId>`
      `<version>`N.N.N `</version>` (eg: 1.2.8)
    `</dependency>`

or to *build.gradle*:

    implementation 'io.keploy:keploy-sdk:N.N.N' (eg: 1.2.6)

## Usage

- **For Spring based application**

  Add `@Import(KeployMiddleware.class)` below `@SpringBootApplication`  in your main class.
- **Run along with agent to mock external calls of your API 🤩🔥**

  - Download the latest - Download the latest agent jar
    from [here](https://search.maven.org/remotecontent?filepath=io/keploy/agent/1.2.8/agent-1.2.8.jar)  (eg: 1.2.8)

    *Note: you can find all agent jars from [here](https://search.maven.org/artifact/io.keploy/agent/1.2.7/jar).*
  - Prefix `-javaagent:` with absolute classpath of agent jar (eg: `-javaagent:<your full path to agent jar>/agent-1.2.8.jar`) is possible through 3 ways:-

    1. **Using Intellij :** Go to Edit Configuration-> add VM options -> paste _java agent_ edited above.
    2. **Using Command Line :**

       ```
       export JAVA_OPTS="$JAVA_OPTS -javaagent:<your full path to agent jar>/agent-1.2.8.jar"
       ```
    3. **Running via Tomcat Server :**

       ```
       export CATALINA_OPTS="$CATALINA_OPTS -javaagent:<your full path to agent jar>/agent-1.2.8.jar"
       ```

## Setup Employee-Manager App

```bash
git clone https://github.com/keploy/samples-java 
```

### Start PostgreSQL instance

```bash
docker-compose up -d
```

### Maven clean install

```shell
mvn clean install 
```

### Set KEPLOY_MODE to record

```
To record testcases use `KEPLOY_MODE` env variable and set the same to `record` mode.
```

### Run the application

```shell
mvn spring-boot:run 
```

## Generate testcases

To generate testcases we just need to **make some API calls.** You can use [Postman](https://www.postman.com/), [Hoppscotch](https://hoppscotch.io/), or simply `curl`

### 1. Make an employee entry

```bash
curl --location --request POST 'http://localhost:8080/api/employees' \
--header 'Content-Type: application/json' \
--data-raw '{
    "firstName": "Myke",
    "lastName": "Tyson",
    "email": "mt@gmail.com",
    "timestamp":1
}'
```

this will return the response or an entry. The timestamp would automatically be ignored during testing because it'll always be different.

```
{
    "id": 1,
    "firstName": "Myke",
    "lastName": "Tyson",
    "email": "mt@gmail.com",
    "timestamp": 1661493301
}
```

### 2. Fetch recorded info about employees

```bash
curl --location --request GET 'http://localhost:8080/api/employees/1'
```

or by querying through the browser `http://localhost:8080/api/employees/1`

Now both these API calls were captured as **editable** testcases and written to `test/e2e/keploy-tests` folder. The keploy directory would also have `mocks` folder.

![testcases](https://i.imgur.com/rhNndcF.png)

Now, let's see the magic! 🪄💫

## Test mode

There are 2 ways to test the application with Keploy.

1. [Unit Test File](/README.md#testing-using-unit-test-file)
2. [KEPLOY_MODE environment variable](/README.md#testing-using-keploy_mode-env-variable)

### **Testing using Unit Test File**

Now that we have our testcase captured, run the unit test file (``SampleJavaApplication_Test.java`) already present in the sample app repo.

If not present, you can add ``SampleJavaApplication_Test.java`` in the test module of your sample application.

```java
    @Test
    public void TestKeploy() throws InterruptedException {
        CountDownLatch countDownLatch = HaltThread.getInstance().getCountDownLatch();
        mode.setTestMode();

        new Thread(() -> {
            SamplesJavaApplication.main(new String[]{""});
            countDownLatch.countDown();
        }).start();

        countDownLatch.await();
    }

```

1. To automatically download and run the captured test-cases. Let's run the test-file.
2. To get test coverage, in addition to above follow below instructions.
3. Add maven-surefire-plugin to your *pom.xml*.

```xml
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>

            <!-- <skipTests>true</skipTests> -->

                <systemPropertyVariables>
                    <jacoco-agent.destfile>target/jacoco.exec</jacoco-agent.destfile>
                </systemPropertyVariables>
            </configuration>
        </plugin>
```

4. Add Jacoco plugin to your *pom.xml*.

```xml
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.5</version>
            <executions>
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
                <execution>
                    <id>post-unit-test</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                    <configuration>
                        <!-- Sets the path to the file which contains the execution data. -->

                        <dataFile>target/jacoco.exec</dataFile>
                        <!-- Sets the output directory for the code coverage report. -->
                        <outputDirectory>target/my-reports</outputDirectory>
                    </configuration>
                </execution>
           </executions>
        </plugin>
```

5. Run your tests using command : `mvn test`.

It will create .html files as test-reports which can be found in your target folder !!

**_We got 75.3% without writing any testcases. 🎉_**

Go to the Keploy Console TestRuns Page to get deeper insights on what testcases ran, what failed.

![testruns](https://i.imgur.com/tg6OT0n.png "Summary")

### **Testing using **KEPLOY_MODE** Env Variable**

To test using `KEPLOY_MODE` env variable, set the same to `test` mode.

```
export KEPLOY_MODE=test
```

Now simply run the application either by ide or using command:

```shell
mvn test
```

Keploy will run all the captures test-cases, compare and show the results on the console.

```shell
10b3ddd5-42fa-48e7-b98a-b47257272e39 total tests: 3
2022-08-26 14:13:08.993  INFO 11560 --- [       Thread-4] io.keploy.service.GrpcService            : testing 1 of 3 testcase id: [ae4a6c91-712a-4566-bf0d-97d708f94b2d]
2022-08-26 14:13:08.994  INFO 11560 --- [       Thread-4] io.keploy.service.GrpcService            : testing 2 of 3 testcase id: [4843e03e-76a8-4194-99cb-f62740978d15]
2022-08-26 14:13:08.994  INFO 11560 --- [       Thread-4] io.keploy.service.GrpcService            : testing 3 of 3 testcase id: [e5231248-de1d-4c8b-8f15-8dcaf63f45c6]
2022-08-26 14:13:09.061  INFO 11560 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-08-26 14:13:09.061  INFO 11560 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-08-26 14:13:09.062  INFO 11560 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
Hibernate: insert into employees (email, first_name, last_name, timestamp) values (?, ?, ?, ?)
2022-08-26 14:13:09.247  INFO 11560 --- [pool-3-thread-1] io.keploy.service.GrpcService            : result : testcase id: [ae4a6c91-712a-4566-bf0d-97d708f94b2d]  passed: true
Hibernate: select employee0_.id as id1_0_0_, employee0_.email as email2_0_0_, employee0_.first_name as first_na3_0_0_, employee0_.last_name as last_nam4_0_0_, employee0_.timestamp as timestam5_0_0_ from employees employee0_ where employee0_.id=?
2022-08-26 14:13:09.291  INFO 11560 --- [pool-3-thread-1] io.keploy.service.GrpcService            : result : testcase id: [4843e03e-76a8-4194-99cb-f62740978d15]  passed: true
Hibernate: select employee0_.id as id1_0_, employee0_.email as email2_0_, employee0_.first_name as first_na3_0_, employee0_.last_name as last_nam4_0_, employee0_.timestamp as timestam5_0_ from employees employee0_
2022-08-26 14:13:09.383  INFO 11560 --- [pool-3-thread-1] io.keploy.service.GrpcService            : result : testcase id: [e5231248-de1d-4c8b-8f15-8dcaf63f45c6]  passed: true
2022-08-26 14:13:09.388  INFO 11560 --- [       Thread-4] io.keploy.service.GrpcService            : test run completed with run id [10b3ddd5-42fa-48e7-b98a-b47257272e39]
2022-08-26 14:13:09.388  INFO 11560 --- [       Thread-4] io.keploy.service.GrpcService            : || passed overall: TRUE ||
2022-08-26 14:13:19.408  INFO 11560 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-08-26 14:13:19.410  INFO 11560 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-08-26 14:13:19.414  INFO 11560 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

> **Note** : With this method coverage will not be calculated.

## Let's add a Bug in the App

Now let's introduce a bug! Let's try changing something like adding some extra headers in controllers  `./EmployeeController.java` on line 35 like :

```java
return ResponseEntity.ok().header("MyNewHeader","abc").body(employee);
```

Let's run the test-file to see if Keploy catches the regression introduced.

```shell
mvn test
```

You'll notice the failed test-case in the output.

```shell
2022-08-26 13:10:10.289 TRACE 70155 --- [nio-8080-exec-2] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
2022-08-26 13:10:10.307  INFO 70155 --- [pool-3-thread-1] io.keploy.service.GrpcService            : result : testcase id: [6aae7f37-798c-42d1-ac8a-c5446880fefc]  passed: false
2022-08-26 13:10:10.312  INFO 70155 --- [       Thread-1] io.keploy.service.GrpcService            : test run completed with run id [a443f2e9-58c9-4c86-8101-7b3e30ef79ff]
2022-08-26 13:10:10.312  INFO 70155 --- [       Thread-1] io.keploy.service.GrpcService            : || passed overall: FALSE ||
```

To deep dive the problem go to [test runs](http://localhost:6789/testruns)

![testruns](https://i.imgur.com/qwP8r4d.png "Recent testruns")

**In case of any query, refer to video below,**

[![java-sample](https://user-images.githubusercontent.com/74777863/217864311-94a3dc0c-90bc-4551-aca2-87e82e3d24cb.png)](https://youtu.be/Ssm4TnTkbLs)
