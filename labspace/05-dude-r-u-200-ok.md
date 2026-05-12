# Step 5: Hello, r u 200 OK?

One of the great features of Spring Boot is the Actuator and its health endpoint. With this endpoint, you can quickly get an overview of the health of your app.

While testing, this endpoint can also be used in a test to quickly validate the application starts up completely.

## Configure Rest Assured

To check the health endpoint of your app while testing, you can use the [RestAssured](http://rest-assured.io/) library.

1. Update your abstract test class with `setUpAbstractIntegrationTest` method since we will share it between all tests:

    ```java save-as=src/test/java/com/example/demo/AbstractIntegrationTest.java
    package com.example.demo;

    import io.restassured.RestAssured;
    import io.restassured.builder.RequestSpecBuilder;
    import io.restassured.specification.RequestSpecification;
    import org.junit.jupiter.api.BeforeEach;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.boot.test.web.server.LocalServerPort;
    import org.springframework.http.MediaType;
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
    }
    ```

    The addition of the `@LocalServerPort` annotation will instruct Spring Boot to inject the app's random port, which is loaded into the RestAssured configuration.

## Call the endpoint

You're now ready to check the health of your application!

1. Add the `healthy` test implementation in the `DemoApplicationTest` class:

    ```java save-as=src/test/java/com/example/demo/DemoApplicationTest.java
    package com.example.demo;
    
    import io.restassured.filter.log.LogDetail;
    import org.junit.jupiter.api.Test;
    import static io.restassured.RestAssured.given;
    
    public class DemoApplicationTest extends AbstractIntegrationTest{
        @Test
        void healthy() {
            given(requestSpecification)
                    .when()
                    .get("/actuator/health")
                    .then()
                    .statusCode(200)
                    .log().ifValidationFails(LogDetail.ALL);
        }
    }
    ```

2. Run the test again: 

    ```bash
    ./mvnw clean test
    ```

    Oh no! It fails:

    ```text
    ...
    HTTP/1.1 503 Service Unavailable
    transfer-encoding: chunked
    Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8

    {
        "status": "DOWN",
        "details": {
            "diskSpace": { ... },
            "redis": {
                "status": "DOWN",
                "details": {
                    "error": "org.springframework.data.redis.RedisConnectionFailureException: Unable to connect to Redis; nested exception is io.lettuce.core.RedisConnectionException: Unable to connect to localhost:6379"
                }
            },
            "db": {
                "status": "UP",
                "details": {
                    "database": "PostgreSQL",
                    "hello": 1
                }
            }
        }
    }
    ... 
    Expected status code <200> but was <503>.
    ```

It seems that it couldn't find Redis and there is no auto-configurable option for it.
