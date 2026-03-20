# Introduction

👋 Welcome to this Labspace focused on helping you learn how to use Testcontainers in a Spring-based application.

By the end, you will have learned the following:

- Write end-to-end API tests using real services instead of mocks
- How to autowire Spring config to use containerized databases
- How to use GenericContainer to provide additional services and wire them with `@DynamicPropertySource`

## Setup

1. Clone the workshop repository into the lab environment:

    ```bash
    git clone https://github.com/testcontainers/workshop.git
    ```

2. Build the project to download the dependencies

    Switch to the workshop folder:

    ```bash
    cd workshop
    ```

3. Build the project with Maven:

    ```bash
    ./mvnw verify
    ```
