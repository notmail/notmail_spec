# Send API
HTTP API which server uses to receive messages

## Description
**Send API** should be always used to send messages to any destination.  
When an application uses this API to send messages it needs to register within the server first, then it can ask for permission to be able to send messages to an specific user, and finally it can send them any time.

This API is composed by different endpoints which serve different purposes.

1. __/app/register/__  
An application will use this API to register itself within the destination server. It may perform a registration for each different server it needs to send messages to.
1. __/app/verify/__  
Allows an application to test if it is correctly registered within the destination server.
1. __/sub/request/__  
A registered application will use this endpoint to ask for permission for sending messages to an specific user within that server.
1. __/sub/verify/__  
Allows an application to test if it is allowed to send messages to a user.
1. __/msg/send/__  
A registered application may use this API to send messages to a set of subscribed users.

## Notes
> As a summary from the document 'server address resolution' and 'glossary' this are the main values that need to be considered when forgin HTTP messages.
>  
> Each user is identifies within a server by it's *notmail*  
> **\<notmail> = \<user>@\<server>**  
> 
> Each server will present the following data as server address on a DNS resolution  
> **\<server_addr> = \<server_ip>:\<server_port>**
>
> This data should be saved by the application in order to use this API.
> Then all messages should contain the following data:  
> IP Header: \<server_ip>  
> TCP Header: \<server_port>  
> HTTP 'Host' header value: \<server_addr>  

## Endpoints

### `/app/register/`

#### Example

### `/app/verify/`
#### Example
### `/sub/register/`
#### Example
### `/sub/verify/`
#### Example

### `/msg/send/`

#### Sending messages
To send a message to a user the server must be registered and the user must have allowed it to send messages to him. Application must have the unique_id and a shared_key.  

POST method should be used to send a message.

#### Request
HTTP headers:
````
POST /msg/send
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
   | unique_id (string)[*]
   | shared_key (string)[*]
| user (string)[only on single]
| users (array of strings)[only on list]
| message (object)[*]
   | type (string)[*]
   | msg (string)[*]
   | title (string)
   | deliver_time (string)
   | alert_time (string)
   | ack (boolean)
```
message types are:
   - single
   - list

#### Response Fields
HTTP response:
- Everything good: **200 OK**  
   "info" may be supplied
- Error Found: **403 Forbidden**  
   "error" must be send

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following 
```
| error (object)
   | wrong_request (boolean)
   | not_registered (boolean) 
   | no_user (boolean)                    # user does not exist
   | not_subscribed (boolean)             # user is not subscribed
| info
   | unsuscribed_users (array of strings) # from 'users' origin field
   | msg_id [only for ack messages]
```


#### Request Example
```
POST /msg/send
Host: 192.168.10.100:5001
Content-Type: application/json
Accept: application/json

{
   "application":{
      "unique_id":"13",
      "shared_key":"a8324kjho34uo23jjo234"
   },
   "user":"user1",
   "message":{
      "title":"New notification",
      "msg":"Hi!"
   }
}
```
#### Response Example
```
HTTP/1.1 200 OK
Content-Type: application/json
```

```
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
   "error":{
      "not_subscribed":true
   }
}
```
