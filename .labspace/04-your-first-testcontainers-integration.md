# Step 4: Your first Testcontainers integration

From the Testcontainers website, we learn that there is a simple way of running different supported JDBC databases with Docker:  
[https://www.testcontainers.org/usage/database\_containers.html](https://www.testcontainers.org/usage/database_containers.html)

An especially interesting part are JDBC-URL based containers:  
[https://www.testcontainers.org/usage/database\_containers.html\#jdbc-url](https://www.testcontainers.org/usage/database_containers.html#jdbc-url)

It means that starting to use Testcontainers in our project \(once we add a dependency\) is as simple as changing a few properties in Spring Boot:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
 "spring.datasource.url=jdbc:tc:postgresql:16-alpine://testcontainers/workshop"
})
```
Let's apply it to the `AbstractIntegrationTest` class:
```java save-as=workshop/src/test/java/com/example/demo/AbstractIntegrationTest.java
package com.example.demo;

import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
        "spring.datasource.url=jdbc:tc:postgresql:16-alpine://testcontainers/workshop"
})
public class AbstractIntegrationTest {

}
```

If we split the magical JDBC url, we see:

* `jdbc:tc:` - this part says that we should use Testcontainers as JDBC provider
* `postgresql:16-alpine://` - we use a PostgreSQL database, and we select the correct PostgreSQL image from the Docker Hub as the image
* `testcontainers/workshop` - the host name \(can be anything\) is `testcontainers` and the database name is `workshop`. Your choice!

## Running tests
In order to run these test we'll need a Docker Engine available. To relax the system ;oad we'll use Testcontainers Cloud to spin up Testcontianers-based containers. 
To ebale it you need to:
1. Go to the [app.testcontainers.cloud](https://app.testcontainers.cloud/) and generate the TC_CLOUD_TOKEN.

2. Set the TC_CLOUD_TOKEN as environment variable:
::variableDefinition[tcc_token]{prompt="What is your TC_CLOUD_TOKEN value?"}

```bash
export TC_CLOUD_TOKEN=$$tcc_token$$
```
3. Start Testcontainers Cloud agent
```bash
sh -c "$(curl -fsSL https://get.testcontainers.cloud/bash)"
```

Now run the test again: 
```bash
./mvnw clean test
```
Test is green? Good!

Check the logs.

```text
2025-10-27T21:36:55.945Z  INFO 77211 --- [           main] o.t.d.DockerClientProviderStrategy       : Found Docker environment with Testcontainers Host with tc.host=tcp://127.0.0.1:43387
2025-10-27T21:36:55.946Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : Docker host IP address is 127.0.0.1
2025-10-27T21:36:56.055Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : Connected to docker: 
  Server Version: 28.3.3 (via Testcontainers Cloud Agent 1.22.0)
  API Version: 1.51
  Operating System: Ubuntu 22.04.5 LTS
  Total Memory: 31556 MB
2025-10-27T21:36:56.206Z  INFO 77211 --- [           main] tc.testcontainers/ryuk:0.8.1             : Creating container for image: testcontainers/ryuk:0.8.1
2025-10-27T21:36:56.396Z  INFO 77211 --- [           main] tc.testcontainers/ryuk:0.8.1             : Container testcontainers/ryuk:0.8.1 is starting: 779608b4dc49f2c37420ea0a39cc90951912fb767d7d7141c1b0ae1db1717989
2025-10-27T21:36:57.321Z  INFO 77211 --- [           main] tc.testcontainers/ryuk:0.8.1             : Container testcontainers/ryuk:0.8.1 started in PT1.114889292S
2025-10-27T21:36:57.521Z  INFO 77211 --- [           main] o.t.utility.RyukResourceReaper           : Ryuk started - will monitor and terminate Testcontainers containers on JVM exit
2025-10-27T21:36:57.523Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : Checking the system...
2025-10-27T21:36:57.530Z  INFO 77211 --- [           main] org.testcontainers.DockerClientFactory   : ✔︎ Docker server version should be at least 1.6.0
2025-10-27T21:36:57.532Z  INFO 77211 --- [           main] tc.postgres:17-alpine                    : Creating container for image: postgres:17-alpine
2025-10-27T21:36:57.679Z  INFO 77211 --- [           main] tc.postgres:17-alpine                    : Container postgres:17-alpine is starting: ed1a75d921ab911896763cde925724777aa6cea00700aec567d6b9a293b1e297
2025-10-27T21:36:58.939Z  INFO 77211 --- [           main] tc.postgres:17-alpine                    : Container postgres:17-alpine started in PT1.406803125S
2025-10-27T21:36:58.943Z  INFO 77211 --- [           main] tc.postgres:17-alpine                    : Container is started (JDBC URL: jdbc:postgresql://127.0.0.1:32771/workshop?loggerLevel=OFF)
```

As you can see, Testcontainers quickly discovered your environment and connected to Docker. 
It did some pre-flight checks as well to ensure that you have a valid environment.

## Hint:

Changing the PostgreSQL version is as simple as replacing `16-alpine` with, for example, `17-alpine`.
Try it, but don't forget that it will download the new image from the internet, if it's not already present on your computer.

```plaintext save-as=workshop/src/test/java/com/example/demo/AbstractIntegrationTest.java
package com.example.demo;

import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
        "spring.datasource.url=jdbc:tc:postgresql:17-alpine://testcontainers/workshop"
})
public class AbstractIntegrationTest {

}
```