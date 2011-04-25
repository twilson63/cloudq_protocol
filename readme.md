# Cloudq Protocol

Cloudq Protocol is a common definition for implementing job worker processing.

## What is a protocol

<blockquote>
A communications protocol is a formal description of digital message formats and the rules for exchanging those messages in or between computing systems and in telecommunications. Protocols may include signaling, authentication and error detection and correction capabilities. A protocol describes the syntax, semantics, and synchronization of communication and may be implemented in hardware or software, or both.
</blockquote>

__-- Wikipedia__

## Why another Job Worker Queue?

We have a ton of job workers queues implemented in all kinds of technologies from
database driven to java, to anything you can think of.  Just like other spaces like
web servers, and datastores.  It can be very confusing based on your needs and what is out there what exactly do you need?  Delayed Job?  RabbitMQ?  0MQ? etc?

How do you know what you need, and when you implement one solution and out grow it, what do you do?  What if you need to access you queue outside of your application?

Don't they make AMQP and The Enterprise Service Bus?  Yeah!

But what if you don't all that jazz, what if you just want to put a job on a stack, and have a collection of workers pick it from that stack and perform the job?

Well, I have not found much in this space and what I found has a different implementation.  Why not come together and create a common implementation standard so that the developer can choose which queue system they would like to use, but not have to change their codebase to implement it.  If each queue worker system implemented the same specification, then the app developer could swap the system out based on their needs.

This protocol is a first step or shout out in this direction.

If you have any feedback or want to contribute please email cloudq@jackhq.com or follow and tweet to [@cloudq_protocol](http://twitter.com/cloudq_protocol)



The cloudq protocol is a simple format for generic job queue processing.  

Using a simple format, we can implement remote or internal job queues and workers in any language and technology.  _The concept comes from some of the Ruby internal job queue processes like Delayed Job and Resque._ 

The protocol uses [JSON](http://www.json.org) JavaScript Object Notation.  JSON has serializers and deserializers implemented in several languages.  See the JSON homepage for more details.


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

