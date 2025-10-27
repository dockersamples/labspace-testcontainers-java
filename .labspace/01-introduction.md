# Step 1: Getting Started

## Clone the following project from GitHub:
```bash
git clone https://github.com/testcontainers/workshop.git
```

## Build the project to download the dependencies
Switch to the workshop folder:
```bash
cd workshop
```
and build the project with Maven:
```bash
./mvnw verify
```

## \(optionally\) Pull the required images before doing the workshop

This might be helpful if the internet connection is somewhat slow.
```bash
docker pull postgres:16-alpine
docker pull redis:7-alpine
docker pull openjdk:8-jre-alpine
docker pull confluentinc/cp-kafka:7.5.0
```