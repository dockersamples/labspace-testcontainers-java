# Introduction

👋 Welcome to this Labspace focused on helping you learn how to use Testcontainers in a Spring-based application.

By the end, you will have learned the following:

- Write end-to-end API tests using real services instead of mocks
- How to autowire Spring config to use containerized databases
- How to use GenericContainer to provide additional services and wire them with `@DynamicPropertySource`

## Setup

1. Run a build to download all of the dependencies and validate the application builds successfully:

    ```bash
    ./mvnw verify
    ```

    Note that it may take a few moments to download all of the Maven dependencies.

    If successful, you should see output similar to the following:

    ```console no-copy-button no-run-button
    [INFO] The original artifact has been renamed to /home/coder/project/target/demo-0.0.1-SNAPSHOT.jar.original
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  27.838 s
    [INFO] Finished at: 2026-05-12T15:41:38Z
    [INFO] ------------------------------------------------------------------------
    ```