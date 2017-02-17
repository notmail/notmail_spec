# Server Logic
This document describes the behaviour of the server application.  

Server logic implementation is not defined so it is up to implementor's choice on how do it as long as it follows the following rules.

The server must responde to certain events triggered by communications and other programmed events.

## Processing

## Application API events

### General Rules
* In all cases, server must check for parsing parameters and ensuring that all of the required ones are present. Else, it will respond with HTTP 400 BAD REQUEST as well as { "error": {"code": "bad_request"} }  
Any other case must be answered as specified by application api specification.
* If any connection is done using HTTP not secured _and_ that connection is carrying the unique\_id/shared_key/root\_secret then
**server must be marked as unsecured source**


### New registration
1. Server registers application in database assigning a unique\_id and generating a random shared\_key of 20 characters.

### Update registration
1. Server will regenerate the random shared\_key as well as update the information ignoring the title field, which can not change.

### Delete registration
1. Server will delete permanently it's records on that server. unique id should not be reused as it might still have meaning for certain users.

### Check registration
1. Server will respond with it's information about the server

### Subscription request
1. After the checks server will add this application **in the list** of that users's application, but **marked as unsubscribed** so he can retrieve it and either accept or deny. 
1. It will also generate a **4 digit random number** and store for that user as well as sending it back as a response.
1. Finally a subscription timeout value is stored in unix time format with 5 minutes added to it.

* All these values are user-specific.
* User has 5 minutes to accept subscription. It can use the code that should also be retrieved with user api to compare both sides and ensure no other applicaton is hijacking the subscription.

### Subscription check
1. Server will check if destination user is ever or still subscribed to that application.

### Message sending
1. Server will store the message in the queue for the *user* and *users* specified as long as they are subscribed to this application.
1. Messages should be stored with all their information.
1. deliver\_time is the time at which the server must send this information to the user. It should not send this message until current\_time is over deliver\_time.
1. aler\_time is an informative value that should be sent to the user when he uses user api. It is the user client who should only show / or only notify this message when this time is arrived.
1. For messages with the ack value set to true, these ones need to be treated separately. When arrived, server must send back a message ID for that user.
* **NOT IMPLEMENTED** It is not implemented yet in neither api, but there should be a possibility to:  
in the app api: check whether user has received notification or not . 
in the user api: mark message as acknowledged.