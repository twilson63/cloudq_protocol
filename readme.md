# Cloudq Protocol

## What is a protocol

<blockquote>
A communications protocol is a formal description of digital message formats and the rules for exchanging those messages in or between computing systems and in telecommunications. Protocols may include signaling, authentication and error detection and correction capabilities. A protocol describes the syntax, semantics, and synchronization of communication and may be implemented in hardware or software, or both.
</blockquote>

__-- Wikipedia__


The cloudq protocol is a simple format for generic job queue processing.  By using a standard and simple format, we can implement remote or internal job queues and workers in any language and technology.  _The concept comes from some of the Ruby internal job queue processes like Delayed Job and Resque._ 

The protocol uses [JSON](http://www.json.org) JavaScript Object Notation.  This is one of the current standards on the we and brings a concise and readable format that virtually every language can understand.

## The Job

The Job is broken down into 2 nodes:

* klass - String
* args - Array

The class is the name of the object you would for the job to call, and the args is an Array of arguments that you would like to pass to the perform class method on the object.

### Example

    { "job": {"klass": "Archive", "args": [{"data": "foobar"}]}}
    
This job is asking a worker to call the perform method on a Class called Archive, passing the data object as a single argument.

### Consumer Example

The consumer will pull the job from the queue and execute the perform method
on the specified class.

    class Archive
      def perform(*args)
        puts "Archived - #{args.first['data']}"
      end
    end
    
    #=> Archived - foobar

By keeping the cloudq job protocol this simple, it allows for a very easy implementation of producers, brokers, and consumers.  Any language or technology
can implement, so you can have producers in ruby, queues in erlang, and consumers 
in javascript.  And they are able to communicate, because they share the same universal
concept of a job.

### The broker

The broker needs to implement 3 basic functions:

* publish
* reserve
* delete

It is recommended to use RESTful techniques to implement these functionalities, but it does not really matter.  Here is a common implementation:

    # Publish Job
    POST /[queue name]

    # Reserve Job
    GET /[queue name]
    
    # Delete Job
    DELETE /[queue name]/[job id]
    
This simple server implementation is very easy to create and easy to create producer and consumer clients to the queue.  

When implementing a queue server, you can use any backend storage system, based on your needs.

## Producer and Consumers

Building producers and consumers become very easy using the common cloudq protocol and server RESTful interface.  Here is an example in raw curl:

    # publish job
    
    curl -X POST -H "Content-Type:application/json" -d '{ "job": {"klass": "Archive", "args": [{"data": "foobar"}]}}' http://my.cloudq.com/archive_queue
    
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
    

    
    
    