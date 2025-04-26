import akka.actor.ActorSystem

import akka.http.scaladsl.Http

import akka.http.scaladsl.model._

import akka.http.scaladsl.server.Directives._

import akka.stream.ActorMaterializer

import spray.json._

import org.apache.kafka.clients.producer.{KafkaProducer, ProducerRecord}

import com.rabbitmq.client.ConnectionFactory


case class ContainerRoutingRequest(containers: List[Int], routes: List[List[Int]])

case class ContainerRoutingResponse(optimalRoute: List[Int])


object ContainerRoutingAPI {
  
  implicit val system = ActorSystem("container-routing-api")
  
  implicit val materializer = ActorMaterializer()
  
  implicit val executionContext = system.dispatcher

  // Kafka producer
 
  val props = new java.util.Properties()
  
  props.put("bootstrap.servers", "localhost:9092")
 
  props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer")
  
  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer")
  
  val kafkaProducer = new KafkaProducer[String, String](props)

  // RabbitMQ connection
 
  val factory = new ConnectionFactory()
  
  factory.setHost("localhost")
 
  val rabbitMQConnection = factory.newConnection()
  
  val channel = rabbitMQConnection.createChannel()

  val route = path("optimize") {
   
    post {
     
      entity(as[String]) { request =>
        
        val containerRoutingRequest = request.parseJson.convertTo[ContainerRoutingRequest]
        
        val optimalRoute = optimize(containerRoutingRequest.containers, containerRoutingRequest.routes)
        
        val response = ContainerRoutingResponse(optimalRoute)

        // Send response to Kafka topic
        kafkaProducer.send(new ProducerRecord[String, String]("container-routing-response", response.toJson.compactPrint))

        // Send response to RabbitMQ queue
        channel.basicPublish("", "container-routing-response", null, response.toJson.compactPrint.getBytes)

        complete(response.toJson.compactPrint)
      }
    }
  }

  def optimize(containers: List[Int], routes: List[List[Int]]): List[Int] = {
   
    // Implement the mathematical formulas for container-routing optimization here
    // For example:
   
    val numContainers = containers.size
   
    val numRoutes = routes.size
    
    val costMatrix = Array.ofDim[Int](numContainers, numRoutes)
    // ...
    // Calculate the optimal route using the cost matrix and other variables
    // ...
    List(1, 2, 3) // Return a sample optimal route
  }

  def main(args: Array[String]) {
   
    val bindingFuture = Http().bindAndHandle(route, "localhost", 8080)
   
    println(s"Server online at http://localhost:8080/\nPress RETURN to stop...")
    
    scala.io.StdIn.readLine()
    
    bindingFuture
      
      .flatMap(_.unbind())
      
      .onComplete(_ => system.terminate())
  }
}


build.sbt

name := "container-routing-api"

version := "0.1"

scalaVersion := "2.13.8"

libraryDependencies ++= Seq(
 
  "com.typesafe.akka" %% "akka-http" % "10.2.9",
  
  "com.typesafe.akka" %% "akka-stream" % "2.6.19",
  
  "io.spray" %% "spray-json" % "1.3.6",
 
  "org.apache.kafka" %% "kafka" % "3.1.0",
  
  "com.rabbitmq" % "amqp-client" % "5.14.2"
)


README.md

## Overview
This API provides a container routing optimization solution using Akka HTTP, Kafka, and RabbitMQ.


## Request/Response Example
Request:
json
{
  "containers": [1, 2, 3],
  "routes": [[1, 2], [2, 3], [1, 3]]
}


Response:

{
  "optimalRoute": [1, 2, 3]
}

Explination of the code
1. import akka.actor.ActorSystem: This line imports the ActorSystem class from the Akka library, which is used to create actors for concurrent programming.

2. import akka.http.scaladsl.Http: This imports the Http object, which is used to create HTTP servers and clients.

3. import akka.http.scaladsl.model._: This imports various models related to HTTP, such as requests and responses.

4. import akka.http.scaladsl.server.Directives._: This imports directives that are used to define routes for the HTTP server.

5. import akka.stream.ActorMaterializer: This imports the ActorMaterializer, which is required for materializing streams.

6. import spray.json._: This imports the Spray JSON library, which is used for JSON serialization and deserialization.

7. import org.apache.kafka.clients.producer.{KafkaProducer, ProducerRecord}: This imports classes needed to create a Kafka producer, which is used for sending messages to Kafka topics.

8. import com.rabbitmq.client.ConnectionFactory: This imports the ConnectionFactory class from RabbitMQ, which is used to create connections to RabbitMQ servers.

9. case class ContainerRoutingRequest(containers: List[Int], routes: List[List[Int]]): This defines a case class for the incoming request, which contains a list of container IDs and their routes.

10. case class ContainerRoutingResponse(optimalRoute: List[Int]): This defines a case class for the response, which contains the optimal route as a list of integers.

11. object ContainerRoutingAPI: This defines a singleton object that will hold the API functionality.

12. implicit val system = ActorSystem("container-routing-api"): This creates an implicit ActorSystem named "container-routing-api", which is used throughout the application.

13. implicit val materializer = ActorMaterializer(): This creates an implicit ActorMaterializer that is necessary for handling streams.

14. implicit val executionContext = system.dispatcher: This sets the execution context to the dispatcher of the ActorSystem, which is used for executing futures.

15. val props = new java.util.Properties(): This creates a new Properties object to configure the Kafka producer.

16. props.put("bootstrap.servers", "localhost:9092"): This sets the address of the Kafka broker.

17. props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer"): This specifies the serializer for the keys of the messages sent to Kafka.

18. props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer"): This specifies the serializer for the values of the messages sent to Kafka.

19. val kafkaProducer = new KafkaProducer[String, String](props): This creates a new Kafka producer with the specified properties.

20. val factory = new ConnectionFactory(): This creates a new ConnectionFactory for RabbitMQ.

21. factory.setHost("localhost"): This sets the RabbitMQ server host to localhost.

22. val rabbitMQConnection = factory.newConnection(): This establishes a new connection to the RabbitMQ server.

23. val channel = rabbitMQConnection.createChannel(): This creates a new channel for communication with RabbitMQ.

24. val route = path("optimize") {: This defines a route for the API that listens for POST requests at the path "/optimize".

25. post {: This specifies that the following directives apply to POST requests.

26. entity(as[String]) { request =>: This extracts the request body as a string.

27. val containerRoutingRequest = request.parseJson.convertTo[ContainerRoutingRequest]: This parses the JSON request body into a ContainerRoutingRequest object.

28. val optimalRoute = optimize(containers, routes): This calls a method optimize (not shown in the snippet) to calculate the optimal route based on the containers and routes provided in the request.
