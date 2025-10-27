# Step 2: Exploring the app

The app is a simple microservice based on Spring Boot for rating conference talks. It provides an API to track the ratings of the talks in real time.

## Storage

### SQL database with the talks

When a rating is submitted, we must verify that the talk for the given ID is present in our database.

Our database of choice is PostgreSQL, accessed with Spring JDBC.

Check :fileLink[TalksRepository]{path="workshop/src/main/java/com/example/demo/repository/TalksRepository.java"}.

### Redis

We store the ratings in Redis database with Spring Data Redis.

Check :fileLink[RatingsRepository]{path="workshop/src/main/java/com/example/demo/repository/RatingsRepository.java"}.

### Kafka

We use ES/CQRS to materialize the events into the state. Kafka acts as a broker and we use Spring Kafka.

Check :fileLink[RatingsListener]{path="workshop/src/main/java/com/example/demo/streams/RatingsListener.java"}.

## API

The API is a Spring Web REST controller :fileLink[RatingsController]{path="workshop/src/main/java/com/example/demo/api/RatingsController.java"} and exposes two endpoints:

* `POST /ratings { "talkId": ?, "value": 1-5 }` to add a rating for a talk
* `GET /ratings?talkId=?` to get the histogram of ratings of the given talk
