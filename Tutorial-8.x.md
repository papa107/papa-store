# Interactive Tutorial 8.1 
**Getting Familiar with Cloud Pub/Sub**

Google Cloud Pub/Sub is a messaging service for exchanging event data among applications and services. A producer of data publishes messages to a Cloud Pub/Sub topic. A consumer creates a subscription to that topic.
What you'll learn to do:

* Set up a topic to hold data.
* Subscribe to a topic to access the data.
* Publish and then consume messages with a pull subscriber.

## Step 1.

**Setting up Pub/Sub**

You can use the Google Cloud Shell console to perform operations in Google Cloud Pub/Sub.
1.0 - To use a Pub/Sub, you create a topic to hold data and a subscription to access data published to the topic.

1.1 **Navigation to the menu** > Pub/Sub > Topics.

1.2 **Click the Create topic**

1.3 The topic must have a unique name. For this lab, name your topic MyTopic. In the Create a topic dialog:

For the Topic ID, **type MyTopic**.

1.4 Leave Encryption at the default value.

1.5 **Click Create Topic**
 
## Step 2

2.0 Now you'll make a subscription to access the topic

2.1 **Click Topics** in the left panel to return to the Topics page. For the topic you just made click the three dot icon > Create subscription.

2.2 In the Add subscription to topic dialog:

**Type a name for the subscription**, such as MySub

2.3 **Set the Delivery Type** to Pull.

2.4 Leave all other options at the default values.

2.5 **Click Create**

Your subscription is listed in the Subscription list.

## Step 3 
In this step you will Publish a message to the topic;

3.1 At the bottom of the Topics details page, **click MESSAGES tab** and then **click PUBLISH MESSAGE**.

3.2 **Enter Hello World** in the Message field and **click Publish**.

## Step 4 
In this step you will View the message;

4.1 To view the message you'll use the subscription (MySub) to pull the message (Hello World) from the topic (MyTopic).

4.2 **Open Cloud Shell** and Enter the following command in the command line:
```
gcloud pubsub subscriptions pull --auto-ack MySub
```
The message appears in the DATA field of the command output.

**Congratulations!** You have completed the Pub/Sub Tutorial
You created a Pub/Sub topic, published to the topic, created a subscription, then used the subscription to pull data from the topic.
