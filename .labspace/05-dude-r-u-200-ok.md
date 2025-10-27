# Step 5: Hello, r u 200 OK?

One of the great features of Spring Boot is the Actuator and its health endpoint. 
It gives you an overview how healthy your app is.

The context starts, but what's about the health of the app?

## Configure Rest Assured

To check the health endpoint of our app, we will use the [RestAssured](http://rest-assured.io/) library.

However, before using it, we first need to configure it. 
Update your abstract test class with `setUpAbstractIntegrationTest` method since we will share it between all tests:

```java save-as=workshop/src/test/java/com/example/demo/AbstractIntegrationTest.java
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
}
```

Here we ask Spring Boot to inject the random port it received at the start of the app, so that we can pre-configure RestAssured's requestSpecification.

## Call the endpoint

Now let's check if the app is actually healthy.
Add the `healthy` test implementationn in the `DemoApplicationTest` class:

```java save-as=workshop/src/test/java/com/example/demo/DemoApplicationTest.java
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

Now let's run the test again: 
```bash
./mvnw clean test
```

Oh ow! it fails:

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

It seems that it couldn't find Redis and there is no autoconfigurable option for it.
