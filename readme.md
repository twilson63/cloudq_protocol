# Cloudq Protocol

Cloudq Protocol is a protocol for implementing highly scalable job queue.

## How it works

Three simple steps:

1. Post a message to a queue 
2. Get the first message available from the queue
3. Delete a message from the queue  

By leveraging web technology you quickly develop highly scalable, easy accessible
queues.

The body of the message has 3 nodes.

- Job
  - klass: { could be the object name you would like to perform the job}
  - args:  { the data that is required to execute the job }
  - id: {This is assigned by the server.}



## Pick or create a server.

## Design

Cloudq piggyback on the Http Protocol and utilizing RESTful concepts.  By using the 
http verbs POST, GET, DELETE, we are able to create a very skinny stack.

The transport is in JSON, which is a protocol that is available in just about any language
that uses the web.  

Cloudq leverages the url to specify the queue name.


## The Job (Message)

The Job is broken down into 2-3 nodes:

* klass - String
* args - Array
* id - String (Should be unique to the queue and assigned by the server)

The class is the name of the object you would for the job to call, and the args is an Array of arguments that you would like to pass to the perform class method on the object.

### Example

    { "job": {"klass": "Archive", "args": [{"data": "foobar"}]}}
    
This job is asking a worker to call the perform method on a Class called Archive, passing the data object as a single argument.

### Consumer Example

The consumer will pull the job from the queue and execute the perform method
on the specified class.

``` ruby
class Archive
  def self.perform(*args)
    puts "Archived - #{args.first['data']}"
  end
end

#=> Archived - foobar
```

By keeping the cloudq job protocol this simple, it allows for a very easy implementation of producers, brokers, and consumers.  Any technology
can implement, so you can have producers in ruby, queues in erlang, and consumers 
in javascript.  And they are able to communicate with each other, because they share the same universal concept of a job.

### The broker

The broker needs to implement 3 basic functions:

* publish
* reserve
* delete

It is recommended to use RESTful techniques to implement these methods.  Here is a common implementation:

    # Publish Job
    POST /[queue name]

    # Reserve Job
    GET /[queue name]

    # Delete Job
    DELETE /[queue name]/[job id]


## Producers and Consumers

Building producers and consumers using the common cloudq protocol.   Here is an example in raw curl:

    # publish job
    
    curl -X POST -H "Content-Type:application/json"  \ 
    -d '{ "job": {"klass": "Archive", "args": [{"data": "foobar"}]}}' http://my.cloudq.com/archive_queue
    
    # response
    { "status": "success" }
    
---

    # consume job
    
    curl http://my.cloudq.com/archive_queue
    
    # response
    
    { "job": {"klass": "Archive", "args": [{"data": "foobar"}], "id": "123456789"}}
    
---
    
    # delete job
    
    curl -X DELETE http://my.cloudq.com/archive_queue/123456789
    

---

# Examples

## Server Implementations

* [Cloudq](https://github.com/twilson63/cloudq)

## Client Implementations

* [Cloudq_client](https://github.com/twilson63/cloudq_client)
* [node-cloudqclient] Coming Soon



## Questions

* What about authentication?
* What about encryption?
* What about monitoring?

Since Cloudq Protocol is built on the basis of using http 1.1, then you can use toolkits like [Rack](http://rack.rubyforge.org/) and [Connect](http://senchalabs.github.com/connect/) to add middleware to your server.

This middleware can contain authentication, encryption, logging, etc...

* What about scheduling?

Implement a worker process that receives a job to be scheduled, then published the 
scheduled job when the alarm triggers.

