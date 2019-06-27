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
    + **Pub/sub**: The messaging infrastructure keeps track of subscriptions. When an event is published, it sends the event to each subscriber. After an event is received, it cannot be replayed, and new subscribers do not see the event.
    + **Event streaming**: Events are written to a log. Events are strictly ordered and durable. Clients don't subscribe to the stream, instead a client can read from any part of the stream. The client is responsible for advancing its position in the stream. That means a client can join at any time, and can replay events.
    
    On the consumer side, there are some common variations:
    + **Simple event processing**: An event immediately triggers an action in the consumer.For example, you could use Azure Functions with a Service Bus trigger.
    + **Complex event processing**: A consumer processes a series of events, looking for patterns in the event data, using a technology such as Azure Stream Analytics or Apache Storm.
    + **Event stream processing**: Use a data streaming platform, such as Azure IoT Hub or Apache Kafka, as a pipeline to ingest events and feed them to stream processors. The stream processors act to process or transform the stream. There may be multiple stream processors for different subsystems of the application. This approach is a good fit for IoT workloads.

    Event-driven architecture has the following features:
    + Real-time processing of request.
    + Asynchronous processing.
    + Receiver can be offline.
    + One-to-many communication.
    + Sender is not aware how the subscriber (receiver) is going to handle the event.

### Azure Event Grid
Azure `Event Grid` is a fully-managed intelligent event routing service that allows `event publishers` to send events to an event ingestion system and `event consumers` to subscribe for that. Event Grid routes the events to the appropriate subscribers. Publisher and subscriber can be Azure or non-Azure service.

`Event Grid` allows you to build applications with event based architectures. Some Azure services such as Resource Groups, Storage account, Service Bus, Event Hub, Media services are already integrated with Event Grid. Event Grid can also support custom events from your applications using `Custom Topics`. An event handler can be any Azure or non-azure service. You can use Azure Function, Logic Apps, Event Hubs, Storage Queues as an `event handler`. You can also use your custom application as an `event handler` using `Web Hooks`. 

![Event Grid](https://docs.microsoft.com/en-us/azure/event-grid/media/overview/functional-model.png)

Some of the key features of Azure Event Grid are:

+ *Simplicity* - Point and click to aim events from your Azure resource to any event handler or endpoint.
+ *Advanced filtering* - Filter on event type or event publish path to make sure event handlers only receive relevant events.
+ *Fan-out* - Subscribe several endpoints to the same event to send copies of the event to as many places as needed.
+ *Reliability* - 24-hour retry with exponential backoff to make sure events are delivered.
+ *Pay-per-event* - Pay only for the amount you use Event Grid.
+ *High throughput* - Build high-volume workloads on Event Grid with support for millions of events per second.
+ *Built-in Events* - Get up and running quickly with resource-defined built-in events.
+ *Custom Events* - Use Event Grid route, filter, and reliably deliver custom events in your app.

#### Subscribe events for Storage account blob service using Event Grid
1. Open the Azure Portal by navigating to [http://portal.azure.com](https://portal.azure.com).
2. Select the `Create a resource` button found on the upper left-hand corner of the Azure portal, then select `Compute` > `Function App`.

    ![Azure Functions](https://docs.microsoft.com/en-us/azure/includes/media/functions-create-function-app-portal/function-app-create-flow.png)
3. Use the function app settings as specified in the table below the image.
    ![Function App](https://docs.microsoft.com/en-us/azure/includes/media/functions-create-function-app-portal/function-app-create-flow2.png)
4. Select Create to provision and deploy the function app.
5. Select the Notification icon in the upper-right corner of the portal and watch for the Deployment succeeded message.
    
    ![Function App](https://docs.microsoft.com/en-us/azure/includes/media/functions-create-function-app-portal/function-app-create-notification.png)

6. Now, You need to create an Http Triggered function in the newly created Function app. Open and expand your new function app, then select the **+** button next to `Functions`, choose `In-portal`, and select `Continue`.

    ![Function](https://docs.microsoft.com/en-us/azure/azure-functions/media/functions-create-first-azure-function/function-app-quickstart-choose-portal.png)

7. Select `More templates` and click on `Finish and view templates` button.

    ![Event Grid Trigger Fn](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-02.png)
    
8. In the search box, type `Event grid` and select `Azure Event Grid Trigger` from the search results.

    ![Event Grid Trigger](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-03.png)

9. This will require and extension to be installed. Click on the `Install` button to install the `EventGrid` web jobs extension.

    ![Event Grid Trigger](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-04.png)

10. Once the installation of extension is completed, Type the name of your function and click `Create`.
    ![Event Grid Trigger](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-05.png)

11. You function is now created. 
12. Open the newly created function and click on the `Add Event Grid Subscription` link.

    ![Azure Functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-06.png)

13. Provide the subscription name and choose the topic type as `Storage accounts`. Choose your subscription, Resource group and storage account name. Click on the `Create` button.

    ![Azure functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-07.png)

14. Open the storage account, select `Events` from the menu options and verify the subscription you have created.

    ![Azure functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-08.png)

15. Open your Azure function in a new browser window and open the `Logs` window. When you add or remove blobs, the event details will be printed on the log window.

    ![Azure functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-09.png)

16. Open the storage account blob service in a new windows and create a new container.

    ![Azure functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-10.png)

16. Upload a new file to the container you have created.

    ![Azure functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-11.png)

17. Switch to the Azure function's window and you can see the event details printed on the log window.
    ![Azure functions](https://raw.githubusercontent.com/sonusathyadas/EventGrid-Samples/master/resources/images/EventGridFunction-12.png)