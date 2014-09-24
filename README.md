# seneca-quartz-scheduler

A scheduler plugin for the [Seneca](http://senecajs.org) toolkit that wraps [node-quartz-scheduler](https://github.com/nherment/node-quartz-scheduler).

Quartz does not expose HTTP services by itself - you'll need to build (maven) a war file from [quartz-http](https://github.com/nherment/quartz-http).

## Support

If you're using this module, feel free to contact me on twitter if you
have any questions! [@wildfiction](http://twitter.com/wildfiction)

Current Version: 0.1.0

Tested on: node 0.10.31, seneca 0.5.19

## Visually

![Scheduler Data Flow](https://raw.githubusercontent.com/guyellis/seneca-quartz-scheduler/master/docs/scheduler-data-flow.png "Scheduler Data Flow")

## Quick examples

Schedule a single future event:

```
var seneca = require('seneca')();

seneca.use('quartz-scheduler');

seneca.ready(function(err){
  if( err ) return console.log(err);

  // Setup the receiver.
  // When the scheduler calls back to your web site then the cmd 'event'
  // is raised on the 'scheduler' micro-service. By hooking into this
  // you can route the executed event to the appropriate place, usually
  // another micro-service.
  this.add({role:'scheduler', cmd:'event'}, function(args, callback){
    var seneca = this;

    // This is the payload you sent to the scheduler when you originally
    // set-it-up with the 'register' cmd.
    var payload = args.data;

    if(payload.action === 'send-email') {
      seneca.act({role:'mailer', cmd:'send', subject: payload.subject, body: payload.body}, function(err,result) {
        // process err and result appropriately
      });
    }
  });

  seneca.act({
    role:'scheduler',
    cmd:'register',
    when: new Date(2014, 5, 1),
    name: 'emailer', // any name you want
    data: {subject: 'test email', body: '<p>test body</p>'}
    }, function(err, data) {
      // If there is no error then data will have a property called jobId.
      // This was generated by the quartz-scheduler and you should store this
      // if you ever want to cancel or update the job that you just scheduled.
      // If you never want to cancel or update the job then you can discard this value.
      var jobId = data.jobId;
      //
      // TIP: You might want to store all the information needed to act on a scheduled
      // event in your local database. In a case like that then the only information you
      // need to send to the scheduler is the identity (unique key) of the record that
      // holds that information. Your workflow might look something like this:
      // 1. Create a record that holds the data to allow execution of a process in the future.
      // 2. Register a scheduler event and in the data payload send {id: record-key-from-above}
      // 3. Receive response from scheduler that event has been scheduled and update the record
      //    from 1 with the jobId
      // 4. When you receive an event from the scheduler in the future use the id in the payload
      //    to retrieve the record you stored in 1. and execute your internal code with that data.
      // 5. (optional) If you need to update or cancel the scheduled job then use the jobId stored
      //    in the record from 1 to do that.
    }
  );

})
```

## Install

```
npm install seneca
npm install seneca-quartz-scheduler
```

You'll need the [seneca](http://github.com/rjrodger/seneca) module to use this module - it's just a plugin.


## Usage

To load the plugin:

```
seneca.use('quartz-scheduler', { ... options ... })
```

For available options, see [node-quartz-scheduler](https://github.com/nherment/node-quartz-scheduler).


## Actions

All actions provide results via the standard callback format: <code>function(error,data){ ... }</code>.


### ACTION: role:scheduler, cmd:register

Register a task with the scheduler.

#### Arguments:

   * _when_: a Date object
   * _name_: A name for your job, can be anything you want and will be sent back when scheduler calls you
   * _data_: A JSON object of the data you want sent to you when the scheduler calls you

## Logging

To see what this plugin is doing, try:

```
node your-app.js --seneca.log=plugin:quartz-scheduler
```

This will print action logs and plugin logs for the user plugin. To skip the action logs, use:

```
node your-app.js --seneca.log=type:plugin,plugin:quartz-scheduler
```

For more logging options, see the [Seneca logging tutorial](http://senecajs.org/logging-example.html).


## Test

Run tests with:

```
npm test
```