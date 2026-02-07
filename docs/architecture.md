## Product choice

- product name: telegram
- link to website: https://web.telegram.org/
- product description: Telegram is a fast, secure, cloud-based messaging app allowing users to send messages, photos, videos, and files of any type (up to 2GB each) to contacts or groups of up to 200,000 members. Founded by Pavel and Nikolai Durov, it focuses on speed, security, and cross-platform syncing across mobile and desktop devices. 

## Main components
![Telegram Component Diagram](../docs/diagrams/out/telegram/component-diagram/Component%20Diagram.svg)
[Telegram Component Diagram Code](../docs/diagrams/src/telegram/component-diagram.puml)

1. SMS Providers (Twilio/Others)
This is an external service that Telegram uses to send SMS messages (e.g., for phone number verification during login). It’s a straightforward integration that handles the actual delivery of text messages to users’ phones.
2. Push Services (FCM/APNs)
These are external platforms (Firebase Cloud Messaging for Android, Apple Push Notification Service for iOS) that deliver push notifications to users’ devices when they receive new messages or updates in Telegram. They simply "ring the bell" on your phone.
3. State Cache (Redis)
This component stores frequently used data (like user login sessions) in fast memory to speed up access. It avoids repeatedly checking the main database, making actions like logging in or loading chats feel instantaneous.
4. Event Bus (Kafka/Internal)
A messaging system that lets different parts of Telegram communicate asynchronously. For example, when you send a message, it quietly notifies other services (like the notification system) without needing direct coordination.
5. Media & File Service
This handles all media (photos, videos) sent in chats: uploading, storing, and delivering files to users. It ensures your cat videos and screenshots get to their destination quickly and reliably.

## Data flow
![Telegram Sequence Diagram](../docs/diagrams/out/telegram/sequence-diagram/Sequence%20Diagram.svg)
[Telegram Sequence Diagram Code](../docs/diagrams/src/telegram/sequence-diagram.puml)

Group 1: Media Upload (Encrypted Parts) – the upload phase before the actual message is sent.

Before Alice's photo message is delivered, Telegram first uploads the raw media file in encrypted chunks to its servers. This "pre-flight" approach ensures the file is safely stored before the message metadata (who sent it, to whom, when) is created – making message sending faster and more reliable since the heavy lifting (file transfer) is already done.

Component interaction and data exchange:
1. Mobile App → MTProto Gateway: Encrypted file chunk, including raw bytes and a temporary to track the upload
2. MTProto Gateway → Media Service: Streams the encrypted chunk data for processing
3. Media Service → Distributed DFS: Writes the encrypted chunk to Telegram's distributed file storage
4. Media Service → Sharded DB: Stores lightweight metadata about the file (owner=Alice, file size, encryption keys) – not the actual media bytes

## Deployment
![Telegram Deployment Diagram](../docs/diagrams/out/telegram/deploymet-diagram/Deployment%20Diagram.svg)
[Telegram Deployment Diagram Code](../docs/diagrams/src/telegram/deployment-diagram.puml)

Telegram's apps run on your phone, computer, or browser and connect to Telegram's servers over the internet. Telegram runs its own servers, databases, and storage in data centers around the world—it doesn't use services like AWS or Google Cloud for its main systems. Only a few helper services (like SMS codes and phone notifications) come from outside companies; bots are run by their creators on their own servers.

## Assumptions
- I assumed Telegram runs many data centers around the world, even though the diagram only showed one labeled "Primary DC."
- I assumed media files stay encrypted while stored in the Distributed File System, though the diagram didn't explicitly show where encryption starts or ends.

## Questions
- Where exactly are Telegram's data centers located around the world?
- How does Telegram keep my messages safe if a server crashes?
- What happens if I lose internet while sending a large video?