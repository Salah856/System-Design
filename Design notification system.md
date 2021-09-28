
## The main requirements: 

- Push notification, SMS message, and email.
- Soft real-time system. 
- Notifications can be triggered by client applications. They can also be scheduled on the server-side.
- 10 million mobile push notifications, 1 million SMS messages, and 5 million emails.



### Contact info gathering flow

To send notifications, we need to gather mobile device tokens, phone numbers, or email addresses. When a user installs our app or signs up for the first time, API servers collect user contact info and store it in the database.

![image](https://user-images.githubusercontent.com/23625821/134760046-e1fd8f36-869c-4284-bc95-6c5a1aa79c84.png)



#### Notification sending/receiving flow

![image](https://user-images.githubusercontent.com/23625821/134760064-19a66727-2ac8-4be1-9ebf-a1ceb6e1ebc9.png)


Service 1 to N: A service can be a micro-service, a cron job, or a distributed system that triggers notification sending events. For example, a billing service sends emails to remind customers of their due payment or a shopping website tells customers that their packages will be delivered tomorrow via SMS messages.

Notification system: The notification system is the centerpiece of sending/receiving notifications. Starting with something simple, only one notification server is used. It provides APIs for services 1 to N, and builds notification payloads for third party services.


Third-party services: Third party services are responsible for delivering notifications to users. While integrating with third-party services, we need to pay extra attention to extensibility. Good extensibility means a flexible system that can easily plugging or unplugging of a third-party service. Another important consideration is that a third-party service might be unavailable in new markets or in the future. For instance, FCM is unavailable in China. Thus, alternative third-party services such as Jpush, PushY, etc are used there.


![image](https://user-images.githubusercontent.com/23625821/134855449-8c0881b1-e2b2-4b92-aa9b-b167bc2ec8b6.png)




















### References

System design interviews 
