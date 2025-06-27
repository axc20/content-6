# Messaging and Notification Services

## SNS (Simple Notification Service)

- Cloud service provided by AWS that facilitates message delivery or sending of notifications in a scalable and flexible manner
- Supports several types of messaging, including mobile notifications, email, and messages sent to other distributed services
- Widely used in scenarios that require a reliable way to deliver messages or notifications to a large number of subscribers or endpoints

### Usage

1. AWS SNS notification is triggered based on various events, such as changes in data, system alerts, or user actions.
2. Backend service (like AWS Lambda or EC2 instance) that is subscribed to the SNS topic receives the notification.
   - The term "SNS topic" is derived from the "publish/subscribe" messaging paradigm. A "topic" is a communication channel to which publishers send messages and subscribers receive messages.
3. Backend service can push an update to the frontend. This could be done using WebSockets (with AWS WebSocket API Gateway for example), Server-Sent Events, or other real-time communication mechanisms.
