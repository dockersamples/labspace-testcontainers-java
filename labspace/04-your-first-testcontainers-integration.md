# Step 4: Your first Testcontainers integration

The Testcontainers framework provides an easy mechanism to run databases in tests using containers. In fact, there are [many JDBC-supported databases](https://java.testcontainers.org/modules/databases/jdbc/) with full support.

One feature of Testcontainers is the ability to use a custom JDBC URL scheme to automatically launch the appropriate database in a container. 

For example, the following JDBC URL will automatically start a PostgreSQL database in a container:

```properties
jdbc:tc:postgresql:18.3:///databasename
```

It means that starting to use Testcontainers in our project \(once we add a dependency\) is as simple as changing a few properties in Spring Boot:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
 "spring.datasource.url=jdbc:tc:postgresql:18-alpine://testcontainers/workshop"
})
public class ...
```

1. Update the :fileLink[`AbstractIntegrationTest`]{path="src/test/java/com/example/demo/AbstractIntegrationTest.java"} class to add the `@SpringBootTest` annotation:

    ```java save-as=src/test/java/com/example/demo/AbstractIntegrationTest.java
    package com.example.demo;

    import org.springframework.boot.test.context.SpringBootTest;

    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
            "spring.datasource.url=jdbc:tc:postgresql:18-alpine://testcontainers/workshop"
    })
    public class AbstractIntegrationTest {

    }
    ```

    If we split the magical JDBC url, we see:

    * `jdbc:tc:` - this part says that we should use Testcontainers as a JDBC provider
    * `postgresql:18-alpine://` - we use a PostgreSQL database, and we select the correct PostgreSQL image from Docker Hub as the image
    * `testcontainers/workshop` - the host name \(can be anything\) is `testcontainers` and the database name is `workshop`. Your choice!

2. Run the tests with the following command:

    ```bash
    ./mvnw clean test
    ```

    When the tests run, you'll see log output indicating it is pulling the `postgresql:16-alpine` container image. It will then use this container to run the tests.

    Eventually, you will see output similar to the following indicating the tests ran successfully:

    ```console no-copy-button no-run-button
    [INFO] Results:
    [INFO] 
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
    [INFO] 
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  15.529 s
    [INFO] Finished at: 2026-05-12T20:12:35Z
    [INFO] ------------------------------------------------------------------------
    ```


## Running tests with Testcontainers Cloud

When the tests ran in the previous step, they used the Docker Engine on your machine. However, there are times in which you might not be able to easily run containers (such as in CI pipelines).

In these environments, Testcontainers Cloud can be used to spin up Testcontainers-based containers. To enable it you need to:

1. Go to the [app.testcontainers.cloud](https://app.testcontainers.cloud/) and generate the `TC_CLOUD_TOKEN`.

2. Set the `TC_CLOUD_TOKEN` as environment variable:

    ::variableDefinition[tcc_token]{prompt="What is your TC_CLOUD_TOKEN value?"}

    ```bash
    export TC_CLOUD_TOKEN=$$tcc_token$$
    ```

3. Start the Testcontainers Cloud agent:

    ```bash
    sh -c "$(curl -fsSL https://get.testcontainers.cloud/bash)"
    ```

4. Now run the test again: 

    ```bash
    ./mvnw clean test
    ```

    Test is green? Good!

5. Check the logs to see the output indicating Testcontainers Cloud is being used:

    ```text
    2026-03-20T21:36:55.945Z  INFO 77211 --- [           main] o.t.d.DockerClientProviderStrategy       : Found Docker environment with Testcontainers Host with tc.host=tcp://127.0.0.1:43387
    2026-03-20T21:36:55.946Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : Docker host IP address is 127.0.0.1
    2026-03-20T21:36:56.055Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : Connected to docker: 
    Server Version: 28.3.3 (via Testcontainers Cloud Agent 1.22.0)
    API Version: 1.51
    Operating System: Ubuntu 22.04.5 LTS
    Total Memory: 31556 MB
    2026-03-20T21:36:56.206Z  INFO 77211 --- [           main] tc.testcontainers/ryuk:0.8.1             : Creating container for image: testcontainers/ryuk:0.8.1
    2026-03-20T21:36:56.396Z  INFO 77211 --- [           main] tc.testcontainers/ryuk:0.8.1             : Container testcontainers/ryuk:0.8.1 is starting: 779608b4dc49f2c37420ea0a39cc90951912fb767d7d7141c1b0ae1db1717989
    2026-03-20T21:36:57.321Z  INFO 77211 --- [           main] tc.testcontainers/ryuk:0.8.1             : Container testcontainers/ryuk:0.8.1 started in PT1.114889292S
    2026-03-20T21:36:57.521Z  INFO 77211 --- [           main] o.t.utility.RyukResourceReaper           : Ryuk started - will monitor and terminate Testcontainers containers on JVM exit
    2026-03-20T21:36:57.523Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : Checking the system...
    2026-03-20T21:36:57.530Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : ✔︎ Docker server version should be at least 1.6.0
    2026-03-20T21:36:57.532Z  INFO 77211 --- [           main] tc.postgres:18-alpine                    : Creating container for image: postgres:18-alpine
    2026-03-20T21:36:57.679Z  INFO 77211 --- [           main] tc.postgres:18-alpine                    : Container postgres:18-alpine is starting: ed1a75d921ab911896763cde925724777aa6cea00700aec567d6b9a293b1e297
    2026-03-20T21:36:58.939Z  INFO 77211 --- [           main] tc.postgres:18-alpine                    : Container postgres:18-alpine started in PT1.406803125S
    2026-03-20T21:36:58.943Z  INFO 77211 --- [           main] tc.postgres:18-alpine                    : Container is started (JDBC URL: jdbc:postgresql://127.0.0.1:32771/workshop?loggerLevel=OFF)
    ```

As you can see, Testcontainers quickly discovered your environment and connected to Docker. 

It also performed some pre-flight checks to ensure the environment is valid.

## Trying other database versions

Changing the PostgreSQL version is as simple as replacing `18-alpine` with, for example, `17-alpine`. That's it!

1. Change the JDBC property value:

    ```java save-as=src/test/java/com/example/demo/AbstractIntegrationTest.java
    package com.example.demo;

    import org.springframework.boot.test.context.SpringBootTest;

    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
            "spring.datasource.url=jdbc:tc:postgresql:17-alpine://testcontainers/workshop"
    })
    public class AbstractIntegrationTest {

    }
    ```

2. And run the tests:

    ```bash
    ./mvnw clean test
    ```

    You'll see the container image for `postgres:17-alpine` download and then start.