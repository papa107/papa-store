# Interactive Tutorial 6.1 Getting Familiar with Cloud Pub/Sub
## Overview
Google Cloud Pub/Sub is a messaging service for exchanging event data among applications and services. A producer of data publishes messages to a Cloud Pub/Sub topic. A consumer creates a subscription to that topic. Subscribers either pull messages from a subscription or are configured as webhooks for push subscriptions. Every subscriber must acknowledge each message within a configurable window of time.
What you'll do:
* Set up a topic to hold data.
* Subscribe to a topic to access the data.
* Publish and then consume messages with a pull subscriber.
 
## Step 1 Setting up Pub/Sub
You can use the Google Cloud Shell console to perform operations in Google Cloud Pub/Sub.

Step 1.1 - To use a Pub/Sub, you create a topic to hold data and a subscription to access data published to the topic.
**Navigation to the menu > Pub/Sub > Topics**.

Step 1.2 **Click the Create topic**.

The topic must have a unique name. For this lab, name your topic MyTopic. In the Create a topic dialog:
* For Topic ID, type MyTopic.
* Leave Encryption at the default value.

Step 1.3 **Click Create Topic**
 
## Step 2 - Make a subscription to access the topic

Step 2.1 **Click Topics** in the left-side panel to return to the Topics page. For the topic you just made **click the three dot icon** > Create subscription.
In the Add subscription to topic dialog:
* Type a name for the subscription, such as MySub
* Set the Delivery Type to Pull.
* Leave all other options at the default values.

Step 2.2 **Click Create**.
Your subscription is listed in the Subscription list.

## Step 3 - Publish a message to the topic;

Step 3.1 At the bottom of the Topics details page, **Click MESSAGES** tab and then **click PUBLISH MESSAGE**.

Step 3.2 **Enter Hello World** in the Message field and click Publish.

## Step 4 - View the message;
To view the message you'll use the subscription (MySub) to pull the message (Hello World) from the topic (MyTopic).

Step 4.1 **Open Cloud Shell** and Enter the following command in the command line.
```
gcloud pubsub subscriptions pull --auto-ack MySub
```

The message appears in the DATA field of the command output.

**Congratulations!** You created a Pub/Sub topic, published to the topic, created a subscription, then used the subscription to pull data from the topic.
Congratulations! You have completed the Pub/Sub Tutorial
