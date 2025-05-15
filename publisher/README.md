### Questions:

### 1. How much data will your publisher program send to the message broker in one run?

In one run, the publisher sends **five separate messages** to the message broker. Each message contains a serialized struct called `UserCreatedEventMessage`, which includes a `user_id` and a `user_name` field—both of which are strings. The actual size in bytes for each message depends on the length of the strings and the overhead introduced by the Borsh serialization format, but in general, the total data size is small since each message is lightweight. Despite the small size, these messages are treated as independent events, which simulates a real-world scenario where multiple users might be created at once. By sending multiple messages in one run, the system can demonstrate and test the asynchronous nature of message-based communication. The use of five events helps ensure that the subscriber receives and processes multiple pieces of data in a queue-based workflow.

### 2. The URL of: "amqp://guest:guest@localhost:5672" is the same as in the subscriber program. What does it mean?

This URL is used by both the publisher and the subscriber to **connect to the same RabbitMQ server instance** using the AMQP protocol. The format of the URL includes credentials (`guest:guest`), the host (`localhost`), and the port number (`5672`). The first "guest" is the **username**, and the second "guest" is the **password** used to authenticate the client with RabbitMQ. "localhost" means that the RabbitMQ server is expected to be running on the **same machine** as the application, which is typical in a local development setup. Port 5672 is the **default port** used by AMQP. Using the same URL ensures that the publisher sends events to the exact message broker that the subscriber is listening to, enabling proper event-driven communication. This setup helps simulate a real microservices architecture where services interact through a shared message broker rather than direct calls.

## RabbitMQ Running Screenshot

![RabbitMQ Running](media/rabbitmq_screenshot.png)

## Sending and Processing Event

After setting up both the subscriber and the publisher, I ran them using two separate terminal windows.

1. In the first terminal, I navigated to the `subscriber` directory and ran `cago run`
This started the subscriber and connected it to the RabbitMQ message broker. On the RabbitMQ web interface (usually available at http://localhost:15672), I could see one active connection and one queue named `user_created`.

2. In the second terminal, I navigated to the `publisher` directory and ran `cargo run`
The publisher immediately sent five events to the `user_created` queue on RabbitMQ. These events contained user creation data such as user ID and name.

3. The subscriber, already listening to the queue, instantly received and processed all five events. Each event triggered a message output in the subscriber's terminal showing the event data was successfully received and handled.

This demonstrates the flow of data in an event-driven architecture where the publisher emits events, RabbitMQ acts as the intermediary, and the subscriber processes those events.

### 📸 Screenshot of Terminals and RabbitMQ

![Sending and Receiving Events](media/events_terminal_screenshot.png)
![RabbitMQ Running](media/events_rabbitmq_screenshot.png)


## Monitoring Chart Based on Publisher

When observing the RabbitMQ Management UI, you may notice a sudden spike in the "Messages Published" and "Messages Ready" charts. These spikes represent bursts of traffic and message activity occurring within the broker.

The spike appears when the publisher application is executed. Each time I run `cargo run` in the publisher directory, it sends five messages (events) to the RabbitMQ queue named `user_created`. These messages are added almost instantly to the message queue, which causes the brief spike in the chart as RabbitMQ registers multiple incoming messages in a short time.

As soon as the subscriber consumes those events, the chart may drop back down, showing that messages have been acknowledged and removed from the queue. Repeatedly running the publisher causes multiple spikes, visually showing the message flow from publisher to broker and on to the subscriber.

This visual feedback confirms the interaction of the publisher with RabbitMQ and demonstrates how message-based systems reflect real-time activity.

### 📈 Screenshot of Monitoring Chart

![RabbitMQ Monitoring Spike](media/monitoring_spike.png)
