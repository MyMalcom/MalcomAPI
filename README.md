Notifications API
=========
 
Introduction
-
Malcom has a communication public interface between software component to make easy the integration of thirds owners platforms with Malcom.

This interface in presented through a REST API (Representation State Transfer).

Objetive
-
This document introduces the new format of push notifications mechanism of Malcom, which make easy the integration with external platforms to send push notifications to different platforms.

General Notes
-
###Conection
Over HTTPS communication protocol.

###Content codification
The "Content-Type" must be **"application/json"** for request and answer.

###Content limit
Notifications are limited to 250 characters

Authentication
-
HTTP Basic Auth with Malcom's user data.

Manage API Errors
-
Errors are returned by standard syntax of HTTP codes. Any additional information as result of the error call is included in the body of the answer in JSON format.

List of error codes:

| Code   | Description     |
| ------------- |:-------------:| 
| 200    | Standard response for successful request |
| 401    | Authentication has failed     |
| 404    | Not exist resource in that path      |
| 412    | The request is invalid or not fulfill a condition to execute it. The answer usually contains a specific description of what have produced the error      |
| 500    | Internal error or when no more specific message is suitable      |


As we mentioned, with a code 412 API return in the body of the message a JSON object with this format:
```json
{
  "response": {
    "description": <Error description>,
    "code": <Error code>
  }
}
```

Resources
-

This is a list of API REST resources available under the URL api.mymalcom.com

Notifications
-

**/v3/notification/push**

####Description
It allows the delivery of a Push Notification to all the devices registered on the
application, segment or in the specified devices.

####URL's structure
api.mymalcom.com/v3/notification/push

####Version
v3

####Method
POST

####Parameters
Request do not require any parameter.

####Request body

| Field   | Description     | Type | Character | 
|:------------- |:-------------|:-------------:|:------------:| 
| notification    |Object container | JSON | Mandatory |
| applicationCode    |App UUID identifier in Malcom | String | Conditional |
| enviroment    |Environment belong the devices that will receive the push notification. Valid values are **SANDBOX**: Devices registered in sandbox environment. **PRODUCTION**: Devices registered in production environment. | String | Mandatory |
| message    |Text of the notification conveniently escaped | String | Mandatory |
| segmentId    |Segment ID that belong all the devices to receive the notification. This property is exclusive of **applicationCode and segmentIds** fields | Integer | Conditional |
| segmentIds    |Segment IDs that belong all the devices to receive the notification. This property is exclusive of **applicationCode and segmentIds** fields. If one device is in several segments only one notification will be sent to it. This property is exclusive of applicationCode and segmentId fields | JSON Array de Integer | Conditional |
| udids    |Device IDs to send the notification separated by commas. If not specified, will be sent to all devices in the application specified in the **applicationCode** field | JSON Array | Conditional |
| saturationControl    |Activate or deactivate saturation control. For any device that have received a notification in the time window defined in **saturationCtrlDays and saturationCtrlHours** fields, will not send the specific request through this message. **Value by default is false** | Booleano | Optional |
| saturationCtrlDays    |Number of days since last notification sent to the device. **Value by default is 0** | Integer | Optional |
| saturationCtrlHours    |Number of hours since last notification sent to the device. **Value by default is 0** | Integer | Optional |
| notificationCustomization    |Container for the customization of the notification | JSON | Optional |
| badge    |Number plate displayed on the notification | Integer | Optional |
| customfield    |Container for customized fields. Belong to the parameters sent to the devices to be interpreted by the application. You can send multiple parameters | JSON Array | Optional |
| entry    |Key-value pair to be interpreted by the application | JSON | Optional |
| sound    |Sound of the notification | String | Optional |
| notificationType    |Notification type to send. In this version just indicate if notification to send is using TAGS filter | String | Optional |
| filter    |Filter to apply if notification is a TAG type | JSON | Optional |
| tags    |Tag name or group of tags to filter the notification | Array | Optional |

####Answer
If the request has been successful, the answer contain the code generated for the notification sent.
```json
{
  "response": {
    "value": {
      "notificationId": 19581
    },
    "code": "OK"
  }
}
```

####Example 1 - Basic notification
```json
{
  "notification":{
    "applicationCode":"3eb5fdfd-8045-4111-8889-999999",
    "environment":"SANDBOX",
    "message":"Este es un mensaje de ejemplo"
  }
}
```

####Example 2 - Notification to different segments
```json
{
  "notification":{
    "segmentIds":[1637,1638],
    "environment":"SANDBOX",
    "message":"Este es un mensaje de ejemplo"
  }
}
```

####Example 3 - Notification to specific tags
```json
{
  "notification":{
    "applicationCode":"3eb5fdfd-8045-4111-8889-999999",
    "environment":"SANDBOX",
    "message":"Este es una notificacion con filtro de tags", 
    "saturationControl":false,
    "saturationCtrlDays":0, 
    "saturationCtrlHours":0, 
    "notificationType":"TAGS",
    "filter":{
      "tags":["myTag"]
    }
  }
}
```

####Example 4 - Notification alert sound
```json
{
  "notification":{
    "applicationCode":"3eb5fdfd-8045-4111-8889-999999", 
    "environment":"SANDBOX",
    "message":"Este es una notificacion con filtro de tags", 
    "notificationCustomization":{
      "sound":"default"
     }
  }
}
```

####Example 5 - Notification custom fields
```json
{
  "notification":{
    "applicationCode":"3eb5fdfd-8045-4111-8889-999999", 
    "environment":"SANDBOX",
    "message":"Este es una notificacion con filtro de tags", 
    "notificationCustomization":{
      "sound":"default",
      "customFields":{
        "entry" : [{
            "key" : "foo",
            "value" : "bar"
           }]
     }
  }
}
```

####Example 6 - Notification to specific devices
```json
{
  "notification":{
    "applicationCode":"3eb5fdfd-8045-4111-8889-999999",
    "environment":"SANDBOX",
    "message":"This is a notification to a specific device",
    "udids":[1234567890]
  }
}
```
    
Notifications status
-

**/v3/notification/push/{id}**

####Description
It allows query the status of a push notification with given {id}

####URL's structure
api.mymalcom.com/v3/notification/push/{id}

####Version
v3

####Method
GET

####Parameters
{id} identifier of the push notification

####Response body

| Field   | Description     | Type | 
|:------------- |:-------------|:-------------:|
| status    |Object container | JSON |
| id    |Notification identifier | Long |
| candidates    |Number of target devices | Integer | 
| failedPushes    |Total falied notifications | Integer |
| sentOn    |Timestamp indicating when notification has been delivered to APNS or GCM services | Date |
| sentPushes    |Total succeed notifications| Integer|

####Answer 1
If the request has been successful, the answer contain the information related to notification.
```json
{
 "status":{
   "candidates":10,
   "failedPushes":0,
   "id":2000,
   "sentOn":"2013-10-04T11:04:44+02:00",
   "sentPushes":10
  }
}
```

####Answer 2
If there isn't a notification with given identifier, the answer is:
```json
{
 "response":
  {
   "description":"Notification with id 2200 not found",
   "code":"Not Found"
  }
}
```

    
