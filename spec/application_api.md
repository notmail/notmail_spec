# APPLICATION API
HTTP API which is used to send messages to other users. This API is accessed from the destination's user's server.

## Description
**APPLICATION API** should always be used to send messages to any destination from an application.  
When an application uses this API to send messages, it needs to register within the server first, then it can ask for permission to be able to send messages to an specific user, and finally it can send them at any time.

This API is composed by different endpoints which serve multiple purposes.

1. __/app/info/register/__  
An application will use this API to register itself within the destination server. It may perform a registration for each different server it needs to send messages to.
1. __/app/info/verify/__  
Allows an application to test if it is correctly registered within the destination server.
1. __/app/info/update/__  
Allows an application to update some information about itself.
1. __/app/info/delete/__  
Allows an application to be removed from the server.
1. __/app/sub/request/__  
A registered application will use this endpoint to ask for permission for sending messages to an specific user within that server.
1. __/app/sub/verify/__  
Allows an application to test if it is allowed to send messages to a user.
1. __/app/msg/send/__  
A registered application may use this API to send messages to a set of subscribed users.

## Notes
> As a summary from the document 'server address resolution' and 'glossary' this are the main values that need to be considered when forgin HTTP messages.
>  
> Each user is identified within a server by it's *notmail*  
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

## Typical Flow
Example of a typical flow, ignoring checks, updates etc.
```
Server %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% Application  

1) Application registers and stores unique_id, shared_key and root_secret from response.
           <-- register ----|  
            |---------------> 

2) Application provides unique_id, shared_key and a user to ask him for subscription permission
           <-- subscription --|
            |-----------> ack

3) Once user has accepted, app can send messages using shared_key and unique_id
           <----- message ----|
            |-----------> ack 
```

## Endpoints

### `/app/info/register/`

#### Register application
So as to send messages to users behind a remote server, an application must register itself and obtain a shared_key and a unique_id. This application must store this values for each server it needs to connect to so as to send messages.

#### Request
HTTP headers:
````
POST /app/info/register/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
    | title (string)[*]
    | description (string)
    | url (string)
```

#### Response
HTTP response:
- Everything good: **200 OK**  
   "info" must be sent
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and one subfield is mandatory in case of an error. The server must check for the first error it encounters in the following list.
```
| error (object)
   | wrong_request (boolean)
   | not_allowed (boolean) #server might not accept new registrations
| info
   | shared_key[*]
   | unique_id[*]
   | root_secret[*]
   
```
An app should store all returned values, it will only last one if special cases though.

### `/app/info/verify/`

#### Verify registration
An application may use this endpoint to verify it's current state on a server.  

#### Request
HTTP headers:
````
POST /app/info/verify/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
    | unique_id (string)[*]
    | shared_key (string)[*]
```

#### Response
HTTP response:
- Everything good: **200 OK**  
   "info" must be supplied
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following list.
```
| error (object)
   | wrong_request (boolean)              # malformed
   | not_registered (boolean)             # wrong data or not registered 
| info
   | unsecured_source (boolean)
```
### `/app/info/update/`

#### Register application
To update or add information about the application such us the url or the description. Note that **title** cannot be updated.

#### Request
HTTP headers:
````
POST /app/info/update/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
    | unique_id (string)[*]
    | root_secret (string)[*]
    | shared_key (string)[*]
    | description (string)
    | url (string)
```
- Server will regenerate the shared_key. unique_id or root_secret should never change.

#### Response
HTTP response:
- Everything good: **200 OK**  
   "info" must be supplied
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following list.
```
| error (object)
   | wrong_request (boolean)
   | not_registered (boolean)             # update info failed 
   | wrong_root_secret (boolean)          
| info
   | shared_key[*]
   | unique_id[*]
   
```

### `/app/info/delete/`

#### Verify registration
An application may use this endpoint to get removed from the server.

#### Request
HTTP headers:
````
POST /app/info/delete/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
    | unique_id (string)[*]
    | shared_key (string)[*]
    | root_secret (string)[*]
```

#### Response
HTTP response:
- Everything good: **200 OK**  
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following list.
```
| error (object)
   | wrong_request (boolean)
   | not_registered (boolean)             # update info failed 
   | wrong_root_secret (boolean)  
```

### `/app/sub/request/`

#### Register Subscription
Once an application is registered in the destination server it must ask for permission to each user it needs to send messages to.  
This is called subscription registering. Subscriptions are 'requested' using this endpoint and only last for a few minutes after they expire. They show a message to the user in both sides that should be observed and manually compared to ensure that there is no third party attempting to hijack the subscription.

#### Request
HTTP headers:
````
POST /app/sub/request/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
    | unique_id (string)[*]
    | shared_key (string)[*]
| user (string)[*]
```

#### Response
HTTP response:
- Everything good: **200 OK**  
   "info" must be supplied
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following list.
```
| error (object)
   | wrong_request (boolean)              # malformed
   | not_registered (boolean)             # wrong data or not 
   registered 
   | no_user (boolean)
| info
   | validation (-4 digit number- string)[*]
```

### `/app/sub/verify/`

#### Verify subscription
An application may use this endpoint to verify if it is allowed to send messages to a user.

#### Request
HTTP headers:
````
POST /app/sub/verify/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
    | unique_id (string)[*]
    | shared_key (string)[*]
| user (string)[*]
```

#### Response
HTTP response:
- Everything good: **200 OK**  
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following list.
```
| error (object)
   | wrong_request (boolean)              # malformed
   | not_subscribed (boolean)             # not subscribed 
```

### `/app/msg/send/`

#### Sending messages
To send a message to a user the server must be registered and the user must have allowed it to send him messages. 

POST method should be used to send a message.

#### Request
HTTP headers:
````
POST /app/msg/send/
Host: <server_addr>
Content-Type: application/json
````

Parameters (after a line break)
```
| application (object)[*]
   | unique_id (string)[*]
   | shared_key (string)[*]
| user (string)[# only on single]           
| users (array of strings)[# only on list]
| message (object)[*]
   | type (string)[# default if omitted]
   | msg (string)[*]
   | title (string)
   | deliver_time (string)
   | alert_time (string)
   | ack (boolean)
```
message types are:
   - single [# default if omitted]
   - list

#### Response
HTTP response:
- Everything good: **200 OK**  
   "info" may be supplied
- Error Found: **403 Forbidden**  
   "error" must be sent

Only error and another extra field is mandatory in case of an error. The server must check for the first error it encounters in the following list.
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
A 'list' type message may not fail even if users are not subscribed.