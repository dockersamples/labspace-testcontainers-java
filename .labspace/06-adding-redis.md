# Step 6: Adding Redis

The simplest way to provide a Redis instance for your tests is to use `GenericContainer` with a Redis Docker image: [https://www.testcontainers.org/usage/generic\_containers.html](https://www.testcontainers.org/usage/generic_containers.html)
The integration between the tests code and Testcontainers is straightforward.  

## Rules? No thanks!

Testcontainers comes with first class support for JUnit, but in our app we want to have a single Redis instance shared between **all** tests. 
Luckily, there are the `.start()`/`.stop()` methods of `GenericContainer` to start or stop it manually.

Just update the `AbstractIntegrationTest` with the following code:

```java no-run-button no-copy-button
static final GenericContainer redis = new GenericContainer("redis:7-alpine")
                                            .withExposedPorts(6379);

@DynamicPropertySource
public static void configureRedis(DynamicPropertyRegistry registry) {
  redis.start();
  registry.add("spring.data.redis.host", redis::getHost);
  registry.add("spring.data.redis.port", redis::getFirstMappedPort);
}
```
The full `AbstractIntegrationTest` class implementation will look like:

```java save-as=workshop/src/test/java/com/example/demo/AbstractIntegrationTest.java
package com.example.demo;

import io.restassured.RestAssured;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.specification.RequestSpecification;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.http.MediaType;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.shaded.com.google.common.net.HttpHeaders;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = {
        "spring.datasource.url=jdbc:tc:postgresql:16-alpine://testcontainers/workshop"
})
public class AbstractIntegrationTest {
    protected RequestSpecification requestSpecification;

    @LocalServerPort
    protected int localServerPort;

    @BeforeEach
    void setUpAbstractIntegrationTest() {
        RestAssured.enableLoggingOfRequestAndResponseIfValidationFails();
        requestSpecification = new RequestSpecBuilder()
                .setPort(localServerPort)
                .addHeader(
                        HttpHeaders.CONTENT_TYPE,
                        MediaType.APPLICATION_JSON_VALUE
                )
                .build();
    }

    static final GenericContainer redis = new GenericContainer("redis:7-alpine")
            .withExposedPorts(6379);

    @DynamicPropertySource
    public static void configureRedis(DynamicPropertyRegistry registry) {
        redis.start();
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", redis::getFirstMappedPort);
    }
}
```

Simple and beautiful, huh?

Run the tests, now they should all pass.
```bash
./mvnw clean test
```