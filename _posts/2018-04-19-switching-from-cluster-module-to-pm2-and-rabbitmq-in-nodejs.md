---
layout: post
title: Switching from cluster module to PM2 & RabbitMQ in Node.js?
excerpt: "How to use PM2 & RabbitMQ in Node.js"
modified: 2018-04-19T02:23:25+05:00
author: waleed_ashraf
image:
  path: ../images/cover_arch.jpg
  feature: cover_arch.jpg
  credit: thomashawk
  creditlink: http://thomashawk.com/
comments: true
share: true
---


_[This article was published in Nodejs-Collection on medium.](https://medium.com/the-node-js-collection/switching-from-cluster-module-to-pm2-rabbitmq-in-node-js-d0cce5eb96f4)_

If you have been using Node.js for sometime, you should know that it is single threaded. This is why you can’t take full advantage of multiple core machines unless you use the cluster module or a process manager like PM2.

I’m working on an application that used the cluster module for managing processes. Although, there were some benefits to this, I decided to move from the cluster module to [PM2](http://pm2.keymetrics.io/) & [RabbitMQ](https://www.rabbitmq.com/). This blog will cover the reasons why I made this change and provide background on how and why I moved to [PM2](http://pm2.keymetrics.io/) & [RabbitMQ](https://www.rabbitmq.com/).

![](https://cdn-images-1.medium.com/max/2408/1*dgzhHyI_MgyeEJtS4omIMA.jpeg)

## The Cluster module:

The cluster module in Node.js allows easy creation of child processes that all share server ports. I was using the cluster module to

* Run multiple processes of the app itself.

* Manage multiple `child_processes` for separate tasks like sending SMS, emails, notifications.

```javascript
var cluster = require('cluster');
const numCPUs = require('os').cpus().length;

module.exports.create = function(options, callback){

  if (cluster.isMaster) {
    // fork child process for notif/sms/email worker
    global.smsWorker    = require('child_process').fork('./smsWorker');
    global.emailWorker  = require('child_process').fork('./emailWorker');
    global.notifiWorker = require('child_process').fork('./notifWorker');

    // fork application workers
    for (var i = 0; i < numCPUs; i++) {
      var worker = cluster.fork().process;
      console.log('worker started. process id %s', worker.pid);
    }
  } else {
    callback(cluster);
  }
};
```

In the code above, there is only one master in the cluster, I get one child process for each worker:

`1 smsWorker + 1 emailWorker + 1 notifWorker + numCPUs app processes.`
With this implementation, I had to also manage the state of the worker process. If it goes down (due to multiple reasons), I had to restart it. After adding that functionality the code looks like this:

```javascript
var cluster = require('cluster');
const numCPUs = require('os').cpus().length;

module.exports.create = function(options, callback){

  if (cluster.isMaster) {
    // fork child process for notif/sms/email worker
    global.smsWorker    = require('child_process').fork('./smsWorker');
    global.emailWorker  = require('child_process').fork('./emailWorker');
    global.notifiWorker = require('child_process').fork('./notifWorker');

    // fork application workers
    for (var i = 0; i < numCPUs; i++) {
      var worker = cluster.fork().process;
      console.log('worker started. process id %s', worker.pid);
    }

    // if application worker gets disconnected, start new one.
    cluster.on('disconnect', function (worker) {
      console.error('Worker disconnect: ' + worker.id);
      var newWorker = cluster.fork().process;
      console.log('Worker started. Process id %s', newWorker.pid);
    });
  } else {
    callback(cluster);
  }
};
```

[`disconnect`](https://nodejs.org/api/cluster.html#cluster_event_disconnect_1) event is fired whenever a worker is killed. So, then I simply start a new process `cluster.fork().process;`.

The only thing left in this implementation is the process communication between the **app workers** and from ***app workers*** to the dedicated task (SMS/email/notif) workers.

### Process communication:

The cluster module provides the [`process.message`](https://nodejs.org/api/cluster.html#cluster_event_message) method to send a message to the master. With this, I was able to listen to different messages from the app and send it to the relevant worker. After adding process communication, the final code looks like this:

```javascript
var cluster = require('cluster');
const numCPUs = require('os').cpus().length;

module.exports.create = function (options, callback) {

  if (cluster.isMaster) {
    // fork child process for notif/sms/email worker
    global.smsWorker    = require('child_process').fork('./smsWorker');
    global.emailWorker  = require('child_process').fork('./emailWorker');
    global.notifiWorker = require('child_process').fork('./notifWorker');

    // fork application workers
    for (var i = 0; i < numCPUs; i++) {
      var worker = cluster.fork().process;
      console.log('worker started. process id %s', worker.pid);
    }

    // if application worker gets disconnected, start new one.
    cluster.on('disconnect', function (worker) {
      console.error('Worker disconnect: ' + worker.id);
      var newWorker = cluster.fork().process;
      console.log('Worker started. Process id %s', newWorker.pid);
    });

    cluster.on('online', function (worker) {
      console.log('New worker is online. worker: ' + worker.id);
      // master receive messages and then forward it to worker based on type.
      worker.on('message', function (message) {
        switch (message.type) {
          case 'sms':
            global.smsWorker.send(message);
            break;
          // each of these worker is listning to process.on('message')
          // and then perform relevant tasks.
          case 'email':
            global.emailWorker.send(message);
            break;
          case 'notif':
            global.notifWorker.send(message);
            break;
        }
      });
    });
  } else {
    global.smsWorker = {
      send: function (message) {
        message.type = 'sms';
        process.send(message); // send message to master
        console.log('sms Message sent from worker.');
      }
    };
    global.emailWorker = {
      send: function (message) {
        message.type = 'email';
        process.send(message); // send message to master
        console.log('email Message sent from worker.');
      }
    };

    global.notifWorker = {
      send: function (message) {
        message.type = 'notif';
        process.send(message); // send message to master
        console.log('notification Message sent from worker.');
      }
    };
    callback(cluster);
  }
};
```

I have defined global methods so that I can send messages from anywhere in the app to the worker. When I call `global.emailWorker.send('Email to send');` method from the app, it sends a message to the master process and then the master process forwards it to relevant `child_process`.

You’ll notice how global.smsWorker is started.
`global.smsWorker = require('child_process').fork('./smsWorker');`

`smsWorker.js, emailWorker.js & notifWorker.js` files are a function listening to messages on process to do relevant task.

```
process.on('message', function(data) {
  sendSMS(data); // use twilio or something to send sms
});
```

For a more clear understanding of the flow of communication between processes, take a look at this:

![](https://cdn-images-1.medium.com/max/3288/1*dLBOyhJAKMxeL5eTrymdCQ.png)

This implementation works well, but there are always some pros and cons.

### **Pros:**

* Scales according to number of CPU cores available on your machine.

* Easy to manage as there is no dependency on any other module/service.

* Easy to implement process communication.

### **Cons:**

* You take a hit on performance of the app if there are too many messages.

* The implementation doesn’t appear to be the best for managing communication of dedicated workers.

* You need to manage the process state by yourself (no automation).

* You can’t *start/stop/restart* (sms/email/notif) workers without affecting app because they are coupled.

I had to switch to PM2 and RabbitMQ because of performance issue. As dedicated workers are coupled with the main app, I was not able to start/stop individual workers and it became difficult to maintain them separately with this implementation.

## PM2, process manager for Node.js:

[PM2](http://pm2.keymetrics.io/) is a process manager for Node.js applications. It lets you efficiently manage multiple processes. It has many features including:

* Auto restart an app, if there is any change in code with [Watch & Reload](http://pm2.keymetrics.io/docs/usage/watch-and-restart/).

* An easy [log management](http://pm2.keymetrics.io/docs/usage/log-management/) for processes.

* [Monitoring](http://pm2.keymetrics.io/docs/usage/monitoring/) capabilities of the process.

* An auto restart if the system reaches [max memory](http://pm2.keymetrics.io/docs/usage/monitoring/#max-memory-restart) limit or it crashes.

* [Keymetrics monitoring](http://pm2.keymetrics.io/docs/usage/monitoring/#keymetrics-monitoring) over the web.

To configure PM2, I just added a config file at the root of the app directory.

```javascript
var pm2Config = {
    "apps": [
      {
        "name": "app",
        "script": "app.js",
        "exec_mode": "cluster_mode",
        "instances": "max"
      },
      {
        "name": "smsWorker",
        "script": "smsWorker.js",
        "instances": 1
      },
      {
        "name": "emailWorker",
        "script": "emailWorker.js",
        "instances": 1
      },
      {
        "name": "notifWorker",
        "script": "notifWorker.js",
        "instances": 1
      }
    ]
};

module.exports = pm2Config;
```

PM2 can also take advantage of the cluster module and manage it by itself. I have configured `app` process to run in `cluster_mode` with `max` possible processes available on the machine. This all depends on the number of cores.

As the processes are separated, now I can start/stop/restart them with `pm2.config.js`, i.e

* `pm2 start pm2.config.js` // start all processes

* `pm2 stop app` // stop app processes

* `pm2 restart smsWorker` // restart smsWorker

This solves the problem of managing all workers. To handle process communication, since all apps are running as a separate process, you can’t use `process.send(message); ` function anymore. This is why I decided to use RabbitMQ as a message broker.

## RabbitMQ, message broker:

[RabbitMQ](https://www.rabbitmq.com/) is one of the most widely used open source message brokers. It uses [amqp](https://www.rabbitmq.com/tutorials/amqp-concepts.html) protocol for messaging. RabbitMQ has many features including:

* Persistent message queues

* [Multiple exchange types](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

* Easy [monitoring and management](https://www.rabbitmq.com/management.html) for any environment

I’m using the [amqplib](https://www.npmjs.com/package/amqplib) module for Node.js with the [topic exchange](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchange-topic) type. I’ll not get into details of RabbitMQ implementation here because that needs a whole article. Maybe next time.

I completely removed the [cluster-module.js](https://gist.github.com/WaleedAshraf/7842f728b06b0ed794ec5bb836e82c85) file (shown above) from app and created [workers-mq.js](https://gist.github.com/WaleedAshraf/7746c51769318ad8b32a92a80fdb7cee):

```javascript
// exports RabbitMQ connection
const MQ = require('./rabbitmq-config');

global.smsWorker = {
  send: function (message) {
    // publish message on sms exchange
    return MQ.publish('sms', message);
  }
};

global.emailWorker = {
  send: function (message) {
    // publish message on email exchange
    return MQ.publish('email', message);
  }
};

global.notifWorker = {
  send: function (message) {
    // publish message on notif exchange
    return MQ.publish('notif', message);
  }
};
```

Here, instead of sending a message to process, you can publish it to RabbitMQ exchange. Workers that were subscribed to the `process.on('message')` now listen to RabbitMQ exchange. The [smsWorker.js](https://gist.github.com/WaleedAshraf/442be09f1fe7218302728d8d51ab130c) above now looks like this:

```
// exports RabbitMQ connection
const MQ = require('./rabbitmq-config');

MQ.on('sms', function(data) {
  sendSMS(data); // use twilio or something to send sms
});
```

With the current implementation, all workers remain independent of each other and RabbitMQ takes the responsibility of communication.

![](https://cdn-images-1.medium.com/max/3348/1*uT6pVNKoowr_eXIFqYr-Xg.png)

### **Pros:**

* PM2 manages all processes.

* It’s easy to start/stop/restart any worker.

* Workers remain independent of each other.

* The Node.js process is free of communication messages.

### **Cons:**

* RabbitMQ is not that easy to implement in production.

* Adds dependency of PM2 and RabbitMQ modules.

* Need to manage/monitor RabbitMQ server along with PM2.

Currently in my setup, *sms/email/notif *workers are entirely separate. Hopefully, in the next post, I’ll write about moving them to Serverless (AWS Lambda), which is the best place for them.

If you are working with [Express.js](https://expressjs.com/) and writing integration/api tests, check my previous post on **[How to Mock an Express Session](https://medium.com/the-node-js-collection/how-to-mock-an-express-session-fe62baf5a611)**.

If you liked this post, don’t forget to share and follow.
[@waleedashraf01](https://twitter.com/WaleedAshraf01)
[github.com/waleedashraf](https://github.com/waleedashraf)
