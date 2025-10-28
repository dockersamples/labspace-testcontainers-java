# Step 7: Test the API

Now let's create a test for our API which will verify the business logic.

```java save-as=workshop/src/test/java/com/example/demo/RatingsControllerTest.java
package com.example.demo;

import com.example.demo.model.Rating;
import org.junit.jupiter.api.Test;

import static io.restassured.RestAssured.given;
import static org.awaitility.Awaitility.await;
import static org.hamcrest.Matchers.is;

public class RatingsControllerTest extends AbstractIntegrationTest {

    @Test
    public void testRatings() {
        String talkId = "testcontainers-integration-testing";

        given(requestSpecification)
                .body(new Rating(talkId, 5))
                .when()
                .post("/ratings")
                .then()
                .statusCode(202);

        await().untilAsserted(() -> {
            given(requestSpecification)
                    .queryParam("talkId", talkId)
                    .when()
                    .get("/ratings")
                    .then()
                    .body("5", is(1));
        });

        for (int i = 1; i <= 5; i++) {
            given(requestSpecification)
                    .body(new Rating(talkId, i))
                    .when()
                    .post("/ratings");
        }

        await().untilAsserted(() -> {
            given(requestSpecification)
                    .queryParam("talkId", talkId)
                    .when()
                    .get("/ratings")
                    .then()
                    .body("1", is(1))
                    .body("2", is(1))
                    .body("3", is(1))
                    .body("4", is(1))
                    .body("5", is(2));
        });
    }

    @Test
    public void testUnknownTalk() {
        String talkId = "cdi-the-great-parts";

        given(requestSpecification)
                .body(new Rating(talkId, 5))
                .when()
                .post("/ratings")
                .then()
                .statusCode(404);
    }
}
```

Run it
```bash
./mvnw clean test
```

Test failed. Why?

There is no Kafka!

Running Kafka in Docker is easy with Testcontainers.
There is a [Testcontainers Kafka module](https://java.testcontainers.org/modules/kafka/) providing integration with Kafka and the `KafkaContainer` abstraction for your code.

Just add it the same way as you added Redis and set the `spring.kafka.bootstrap-servers` system property.
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
import org.testcontainers.containers.KafkaContainer;
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
    static final KafkaContainer kafka = new KafkaContainer("7.7.6");

    @DynamicPropertySource
    public static void configureRedisAndKafka(DynamicPropertyRegistry registry) {
        redis.start();
        kafka.start();
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", redis::getFirstMappedPort);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
}
```

Run test one more time:
```bash
./mvnw clean test
```

## Hint 1:

Some containers expose helper methods. Check if there is one on `KafkaContainer` which might help you.
![KafkaContainer helpers](.labspace/images/kafka.png)


## Hint 2:

You can start several containers in parallel by doing:

```java
Stream.of(redis, kafka).parallel().forEach(GenericContainer::start);
```
