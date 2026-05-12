# Step 6: Adding Redis

The simplest way to provide a Redis instance for your tests is to use a `GenericContainer` with a Redis Docker image: [https://www.testcontainers.org/usage/generic\_containers.html](https://www.testcontainers.org/usage/generic_containers.html)

The integration between the tests code and Testcontainers is straightforward.

## Rules? No thanks!

Testcontainers comes with first class support for JUnit, but in our app we want to have a single Redis instance shared between **all** tests. 

Luckily, there are the `.start()`/`.stop()` methods of `GenericContainer` to start or stop it manually.

1. Update the :fileLink[`AbstractIntegrationTest`]{path="src/test/java/com/example/demo/AbstractIntegrationTest.java"} to create a redis container:

    ```java no-run-button
    static final GenericContainer redis = new GenericContainer("redis:8.6-alpine")
                                                .withExposedPorts(6379);
    ```

2. In that same class, add a method annotated with `@DynamicPropertySource` to provide the properties needed to connect to the Redis container:

    ```java
    @DynamicPropertySource
    public static void configureRedis(DynamicPropertyRegistry registry) {
        redis.start();
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", redis::getFirstMappedPort);
    }
    ```

    This will extract the host and randomly selected port the Redis container will be using and put those details into the Spring configuration.

    The full `AbstractIntegrationTest` class implementation should now look like the following:

    ```java save-as=src/test/java/com/example/demo/AbstractIntegrationTest.java
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
            "spring.datasource.url=jdbc:tc:postgresql:18-alpine://testcontainers/workshop"
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

3. Run the tests again to validate everything works now:

    ```bash
    ./mvnw clean test
    ```

    If successful, you should see output similar to the following:

    ```console no-copy-button no-run-button
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 6.688 s - in com.example.demo.DemoApplicationTest
    [INFO] 
    [INFO] Results:
    [INFO] 
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
    [INFO] 
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  8.092 s
    [INFO] Finished at: 2026-05-12T20:26:19Z
    [INFO] ------------------------------------------------------------------------
    ```