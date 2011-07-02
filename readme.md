# Cloudq Protocol v 0.1.0

Cloudq Protocol is a simple specification for implementing job queue.

## Description

A job queue server allows you to send long running and unique processes
to a set of provisioned workers to execute the jobs, while your core application
can return quickly to the user to allow the user to perform other features in the
application.  

Job Queues are mainly used for common background tasks like sending e-mail or 
faxes, which may be behind your firewall on in your same network.  

Cloudq gives you the capability of connecting remote workers to your queue.  For
example, if you had a client that was in an environment that needed to pull data
from a client server application to your cloud database for your e-commerce store.
Or you needed to pull information down from your cloud database to a local application.

Cloudq would act as a broker between the two applications and allow you to post and
consume jobs between the two systems via a SSL connection, without having to poll 
you cloud application or cloud database.  

Cloudq would give you the ability to run 1 to N cloudq servers and 1 to N cloudq
workers.

## Workflow

An Application Posts a JOB to a Cloudq Server Queue, that JOB is set to a state
of :new.  Then a Worker Requests (GET) a JOB from the Cloudq Server Queue, the 
server sets the state of that JOB to :reserved and hands it to the worker.  The
worker verifies that it go the JOB, then submits a DELETE request to the Cloudq 
Server.  This changes the status of the JOB from :reserved to :deleted.  

## Definition of a JOB

A JOB is a structured message that contains a consistent structure that can be
accepted and understood by any client.

### JOB Format

The JOB message format is in JSON a javascript object notation specification.
[JSON](http://json.org)

### Elements

- Job
  - klass: [STRING] { could be the object name you would like to perform the job}
  - args:  [ARRAY] { the data that is required to execute the job }
  - id: [UNIQUE] {This is assigned by the server.}

### Example 

```
 {
"klass": "SendEmail",
"args": ["from@email.com", "to@email.com", "SUBJECT: Hello", "BODY: Goodbye"],
"id": "112345566"
 }
```

### Response Message

The response message is the message that the cloudq server returns when a request
is executed.  It is also in JSON Format and lets the client know if the request 
was successful or not, and if not why.  This message contains two attributes:

* status
* message

```
{
	"status": "success|error|empty",
	"message": "Optional"
}
```

## How to implement your own cloudq server

Implementing your cloudq server is pretty easy, first choose your language and 
datastore.  Make sure your language can support the http protocol and has some
good networking libraries.  Most languages have some awesome high level, http api libraries
like Sinatra, Expressjs, etc.

The Cloudq server basically only needs to handle three http methods:

* POST /:queue
* GET /:queue
* DELETE /:queue/:id

### POST /:queue

When a client requests a POST for a queue name the server needs to first make
sure that JOB is valid, if the job is not valid then return a response message.

_For our ruby and node implementations we use MongoDb and create a collection called jobs and the documents in the 
jobs collection contain an attribute called :queue_

### GET /:queue

When a client performs a get request, they are requesting an items from the :queue, the 
server should find a JOB with a status of :new and update that item to a status of :reserved
then respond to the client with the JOB.  If there are no JOBS in a new status for that 
queue then the server should return an empty response.

### DELETE /:queue/:id

When a client requests a DELETE JOB request, the server needs to locate the JOB and
modify the status of the job to :deleted, and return a success response, if it can't
find the job then return an error response or empty response.

---

The above is just a guideline, you can build and modify your server anyway you want, these methods and messages are 
defined to be simple and easy to implement as a core JOB Queue, to allow for extremely easy implementations of clients
in several different languages.

---

## Cloudq Client Implementation

A Cloudq client has the ability to post a JOB to the Queue and the ability to reserve and delete a JOB.

## cURL job examples...

Building producers and consumers using the common cloudq protocol.   Here is an example in raw curl:

### publish job
    
    curl -X POST -H "Content-Type:application/json"  \ 
    -d '{ "job": {"klass": "Archive", "args": [{"data": "foobar"}]}}' http://my.cloudq.com/archive_queue
    
    # response
    { "status": "success" }
    

### consume job
    
    curl http://my.cloudq.com/archive_queue
    
### response
    
    { "job": {"klass": "Archive", "args": [{"data": "foobar"}], "id": "123456789"}}
    
    
### delete job
    
    curl -X DELETE http://my.cloudq.com/archive_queue/123456789
    

## Build a user friendly api with your cloudq client

If you are implementing a client you may want to create a simple api that makes it extremely easy for 
the users of your client library to interact with the cloudq server.  

## For Example:

To post a job you may want to allow the user to simple call:

CloudQClient.publish [QUEUE], [KLASS], [ARG1], [ARG2], [ARG3]

```
client.publish "EMAIL-QUEUE", "WELCOME", "to@email.com", "subject", "body"
```

Then in your client library compose this into a json JOB object:

```
{
	"klass": "WELCOME",
	"args": ["to@email.com", "subject", "body"]
}
```

And create a RESTful url to post the JOB to:

```
http://server.com/email-queue
```

Then execute a POST method on your http network client using the url and the JOB as the body of the request 
method.

Here is an example using cURL

```
curl -XPOST -H "Content-Type:application/json" -d '{"klass": "WELCOME", "args": ["to@email.com", "subject", "body"]}' http://server.com/email-queue
```

## For more ideas look at the ruby client implementation

[Cloudq_client](https://github.com/twilson63/cloudq_client)

## For a complete list of cloudq implementations visit our wiki

[https://github.com/twilson63/cloudq_protocol/wiki](https://github.com/twilson63/cloudq_protocol/wiki)



