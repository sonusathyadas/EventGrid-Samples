## Azure EventGrid - An overview to event based applications

### Communication strategies in application programming
1. **REST Services**
    This is one of the commonly used communication method in applications. A service can be exposed as an HTTP endpoint and one or more clients can send requests to that `HTTP endpoint`. The REST servcie accepts the reqeust and send the response to the client.

    ![REST communication](https://www.studytonight.com/rest-web-service/images/REST-message.png)

    The following are the features of REST based communications:
    + `Sender` and `receiver` must be online. If receiver is offline it send an error response to the client. 
    + One to one communication.
    + Synchronous by default. Optionally, methods can be called as asynchronous methods also. 
    + Sender always gets a response from the receiver. (Success response or error response).
    + Sender is well aware, how the request is going to be processed and what kind of response is can be expected. 
    + Communication is real-time.
    + Optionally, we can use an API Gateway to do advanced request-response processing such as SSL offloading, authentication, routing and caching etc.

    ![REST with API Gateway](https://docs.microsoft.com/en-us/azure/architecture/microservices/images/gateway.png)

2. **Message based communication**
    In a message based communication we use an intermediary component called `Message Broker`(eg:RabbitMQ, Azure Service Bus). They commonly follows the `AMQP (Advanced Message Queuing Protocol)` to provide interoperability between all messaging middleware. When using messaging, processes communicate by exchanging messages asynchronously. A client makes a command or a request to a service by sending it a message. If the service needs to reply, it sends a different message back to the client. Since it's a message-based communication, the client assumes that the reply won't be received immediately, and that there might be no response at all. 

    ![Messaging pattern](https://www.cloudamqp.com/img/blog/rabbitmq-beginners-updated.png)

    Following are the features of message based communication:
    + Asynchromous communication.
    + Receiver can be offline. When it comes online, it can consume messages from the queue.    
    + Typically, not response expected from the consumer. If required consumer can use a `reply queue` to send responses with `correlation id`.
    + Not a real-time communication.
    + Messages are simple text contents.
    + Sender can have an expectation, how the messages will be processed by the consumer.
    + Messages can be sent to all consumers using `fanout` fashion or to single receiver using `direct` or  only to a group of receivers by using `topic`.

    ![Message exchange patterns](https://i2.wp.com/blog.knoldus.com/wp-content/uploads/2018/12/exchanges-topic-fanout-direct.png)

3. **Event-driven architecture**
    In a distributed microservices based applications, you may need to notify the changes in one service or data to one or more other services. It should be a real-time communication but it should be asynchronous too. The sender (event generator) needs to inform other components or services about the latest changes but not expecting a response from them. REST based communication is real time but not asynchronous by default. The client needs to wait for the response from the service. Also it is one-to-one communication. You may need to make multiple HTTP requests to notify the more than one receivers. A messaging system can be asynchrous but not real time.  
    
    An event-driven architecture is best suited for such scenarios. The event-driven    architecture pattern is a popular distributed asynchronous architecture pattern used to produce highly scalable applications. An `event` is an object that represents what had happened in the event source. An `event source` is a component that generates stream of events. An `event consumers` is a service or application that consumes the events and react to it. An `event source` application uses `topics` to publish events. A `topic` is an end point for publishing an event. `Event consumers` uses `event subscriptions` to filter and consume events. An `event subscription` is an endpoint to consume routed events.

    ![Event ingestion system](https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/images/event-driven.svg)
    
    Events are delivered in near real time, so consumers can respond immediately to events as they occur. Producers are decoupled from consumers — a producer doesn't know which consumers are listening. Consumers are also decoupled from each other, and every consumer sees all of the events. 
    
    An event driven architecture can use a `pub/sub model` or an `event stream model`.
    **Pub/sub**: The messaging infrastructure keeps track of subscriptions. When an event is published, it sends the event to each subscriber. After an event is received, it cannot be replayed, and new subscribers do not see the event.
    **Event streaming**: Events are written to a log. Events are strictly ordered and durable. Clients don't subscribe to the stream, instead a client can read from any part of the stream. The client is responsible for advancing its position in the stream. That means a client can join at any time, and can replay events.
    
    On the consumer side, there are some common variations:
    **Simple event processing**: An event immediately triggers an action in the consumer.For example, you could use Azure Functions with a Service Bus trigger.
    **Complex event processing**: A consumer processes a series of events, looking for patterns in the event data, using a technology such as Azure Stream Analytics or Apache Storm.
    **Event stream processing**: Use a data streaming platform, such as Azure IoT Hub or Apache Kafka, as a pipeline to ingest events and feed them to stream processors. The stream processors act to process or transform the stream. There may be multiple stream processors for different subsystems of the application. This approach is a good fit for IoT workloads.