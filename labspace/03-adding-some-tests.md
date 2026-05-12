# Step 3: Adding some tests

The app doesn't have any tests yet.

But before you write your first test, you'll create an abstract test class for the common things used by all tests.

## Abstract class

1. Define the `AbstractIntegrationTest` class to `src/test/java` sourceset, in the `com.example.demo` package.

    It will be an abstract class with standard Spring Boot's testing framework annotations on it:

    ```java save-as=src/test/java/com/example/demo/AbstractIntegrationTest.java
    package com.example.demo;

    import org.springframework.boot.test.context.SpringBootTest;

    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    public class AbstractIntegrationTest {
        
    }
    ```

## Your very first test

With the abstract base defined, you are ready to create your first test.


1. Create `DemoApplicationTest`, extending the `AbstractIntegrationTest` base class, adding a simple test:

    ```java save-as=src/test/java/com/example/demo/DemoApplicationTest.java
    package com.example.demo;

    import org.junit.jupiter.api.Test;

    public class DemoApplicationTest extends AbstractIntegrationTest{
        @Test
        public void contextLoads()  {
        }
    }
    ```

2. Run it and verify that the application starts and the test passes.

    ```bash
    ./mvnw clean test
    ```

    You should get output similar to the following:

    ```console no-copy-button no-run-button
    [INFO] Results:
    [INFO] 
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
    [INFO] 
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  5.009 s
    [INFO] Finished at: 2026-05-12T20:02:41Z
    [INFO] ------------------------------------------------------------------------
    ```

This is already a useful smoke test since it ensures, that Spring Boot is able to initialize the application context successfully.

## Populate the database

The Spring context starts. However, the database needs to be populated with some data before to run additional tests.

1. Create a `schema.sql` test resource file with the following content:

    ```sql save-as=src/test/resources/schema.sql
    CREATE TABLE IF NOT EXISTS talks(
      id    VARCHAR(64)  NOT NULL,
      title VARCHAR(255) NOT NULL,
      PRIMARY KEY (id)
    );

    INSERT
      INTO talks (id, title)
      VALUES ('testcontainers-integration-testing', 'Modern Integration Testing with Testcontainers')
      ON CONFLICT do nothing;

    INSERT
      INTO talks (id, title)
      VALUES ('flight-of-the-flux', 'A look at Reactor execution model')
      ON CONFLICT do nothing;
    ```

2. Now run the test again. 

    ```bash
    ./mvnw clean test
    ```

    Oh no, it fails!

    ```plaintext no-copy-button
    ...
    Caused by: org.h2.jdbc.JdbcSQLException: Syntax error in SQL statement "INSERT INTO TALKS (ID, TITLE) VALUES ('testcontainers-integration-testing', 'Modern Integration Testing with Testcontainers') ON[*] CONFLICT DO NOTHING";
    ...
    ```

    It seems that H2 does not support the PostgreSQL SQL syntax, at least not by default.

> [!IMPORTANT]
> Spring, by default, will use a H2 (in-memory) database when running tests. While this works in many scenarios, it varies from what your application will actually be using.
>
> To increase confidence in your application testing, you want to use the same database technology whenever possible.