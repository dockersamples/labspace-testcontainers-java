# Step 2: Exploring the app

The app is a simple microservice based on Spring Boot for rating conference talks. It provides an API to track the ratings of the talks in real time.

## Storage

### SQL database with the talks

When a rating is submitted, the app verifies the talk for the given ID is present in the database.

The :fileLink[TalksRepository]{path="src/main/java/com/example/demo/repository/TalksRepository.java"} class is responsible for interacting with the PostgreSQL database, accessed with Spring JDBC.

### Redis

The app stores the talk ratings in Redis database with Spring Data Redis, which is handled with the :fileLink[RatingsRepository]{path="src/main/java/com/example/demo/repository/RatingsRepository.java"} class.

### Kafka

The app uses ES/CQRS to materialize the events into the state, using Kafka acts as a broker and Spring Kafka to manage the connections. This is all handled in the :fileLink[RatingsListener]{path="src/main/java/com/example/demo/streams/RatingsListener.java"} class.

## API

The app exposes an API with a Spring Web REST controller :fileLink[RatingsController]{path="src/main/java/com/example/demo/api/RatingsController.java"}. This controller exposes two endpoints:

* `POST /ratings { "talkId": ?, "value": 1-5 }` - add a rating for a talk
* `GET /ratings?talkId=?` - get the histogram of ratings of the given talk
