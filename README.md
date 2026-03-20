# Labspace - Introduction to Testcontainers - Java

Containers can be used for more than simply running your application. With Testcontainers, you can use containers to easily start the services needed by your application.

In this Labspace, you'll learn how to use Testcontainers in a Spring-based application.

## Learning objectives

By the end of this Labspace, you will have learned the following:

- Write end-to-end API tests using real services instead of mocks
- How to autowire Spring config to use containerized databases
- How to use GenericContainer to provide additional services and wire them with `@DynamicPropertySource`

## Launch the Labspace

To launch the Labspace, run the following command:

```bash
docker compose -f oci://dockersamples/labspace-testcontainers-java up -d
```

And then open your browser to http://localhost:3030.

### Using the Docker Desktop extension

If you have the Labspace extension installed (`docker extension install dockersamples/labspace-extension` if not), you can also [click this link](https://open.docker.com/dashboard/extension-tab?extensionId=dockersamples/labspace-extension&location=dockersamples/labspace-testcontainers-javaimages&title=Introduction%20to%20Testcontainers%20-%20Java) to launch the Labspace.


## Contributing

If you find something wrong or something that needs to be updated, feel free to submit a PR. If you want to make a larger change, feel free to fork the repo into your own repository.

**Important note:** If you fork it, you will need to update the GHA workflow to point to your own Hub repo.

1. Clone this repo

2. Start the Labspace in content development mode:

    ```bash
    # On Mac/Linux
    CONTENT_PATH=$PWD docker compose up --watch

    # On Windows with PowerShell
    $Env:CONTENT_PATH = (Get-Location).Path; docker compose up --watch
    ```

3. Open the Labspace at http://localhost:3030.

4. Make the necessary changes and validate they appear as you expect in the Labspace

    Be sure to check out the [docs](https://github.com/dockersamples/labspace-infra/tree/main/docs) for additional information and guidelines.
