---
layout: post
title: "The Ember.js Run Loop"
modified:
categories:
comments: true
excerpt: Explore into Ember.js Run Loop implementation
tags:
  - Ember.js
image:
  feature:
  credit:
  creditlink:
date: 2014-09-24T13:54:49-07:00
---

## Introduction

Ember.js is an ambitious Javascript MVC framework. It not only provides a clear way to build a web application using router, controller and view. but also utilizes a clever structure, the run loop, to schedule the jobs in an Ember application so as to avoid unnecessary computations.

The run loop is entirely wrapped in the implementation of Ember. So developers building light web applications  don't need to worry about explicitly invoking it. Nevertheless, it is worthwhile for us to take a deep look at its implementation.

## Basic Idea

Ember.js arranges jobs into six different queues:

+ `sync`
+ `actions`
+ `routerTransitions`
+ `render`
+ `afterRender`
+ `destroy`

----

### sync

Synchronization jobs. e.g. all the bindings in Ember.js.

### actions

A general work queue. Typically contains scheduled jobs.

### routerTransitions

Contains router transition jobs.

### render

Contains rendering jobs, which typically updates DOM. e.g. template rendering.

### afterRender

Contains scheduled jobs that should be triggered after the render jobs finish.

### destroy

Contains jobs to destroy objects.

---

The priority of these queues is first to last, from `sync` to `destroy`.

### The algorithm

In a run loop, Ember.js will look at each queue in the order of priority. If a queue has pending tasks, the run loop will execute them. Obviously, this will possibly schedule more tasks. So the run loop continues to execute the tasks in the above order until there is no pending jobs.

## Advantages

The Ember run loop separates logics and rendering so that it avoids running many unnecessary codes, e.g. updates the bindings.

To illustrate this, let me use an example from [the official website](http://emberjs.com).

We have an object, `User`,

{% highlight javascript %}
var User = Ember.Object.extend({
  firstName: null,
  lastName: null,
  fullName: function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName')
});
{% endhighlight %}

a simple template,

{% highlight javascript %}
{{firstName}}
{{fullName}}
{% endhighlight %}

and a few Javascript code.

{% highlight javascript %}
var user = User.create({firstName:'Tom', lastName:'Huda'});
user.set('firstName', 'Yehuda');
user.set('lastName', 'Katz');
{% endhighlight %}

Without the run loop, the above code should ask the browser to render the template three times:

{% highlight javascript %}
var user = User.create({firstName:'Tom', lastName:'Huda'});
// object created, first render

user.set('firstName', 'Yehuda');
// attributes updated, second render, {{firstName}} and {{fullName}} are updated

user.set('lastName', 'Katz');
// attributes updated, third render, {{lastName}} and {{fullName}} are updated
{% endhighlight %}

But with the run loop, the rendering jobs in the `render` queue won't be executed before all the above code is done:

{% highlight javascript %}
var user = User.create({firstName:'Tom', lastName:'Huda'});
user.set('firstName', 'Yehuda');
user.set('lastName', 'Katz');
// first render, {{firstName}} and {{fullName}} are updated
{% endhighlight %}

## Implementation

### Code structure

+ `ember-metal/lib/run_loop.js` is where the run loop code lies. It provides all the API methods developers need to know. Moreover, it adds `sync`, `actions` and `destroy` queues.
+ `ember-metal/lib/backburner.js` is the underlying actual run loop handler used by `run_loop.js`.
+ `ember-views/lib/ext.js` adds `render` and `afterRender` queues.
+ `ember-routing/lib/ext/run_loop.js` adds the `routerTransitions` queue.

### Backburner structure

`backburner.js` provides a class of run loop controller, `Backburner`. Each `Backburner` handles one or more similar `DeferredActionQueues` objects, aka the run loops. Moreover, each `DeferredActionQueues` is consisted of `queue`s, which is the place the scheduled jobs rest.

In conclusion, the `Backburner` starts `DeferredActionQueues` to run different jobs. Each `DeferredActionQueues` executed the jobs in a specific priority order by sending them to different `queue`s.

### Actual Code

+ Initiating the actual handler, `backburner`.

  `run_loop.js`

{% highlight javascript %}
function onBegin(current) {
  run.currentRunLoop = current;
}

function onEnd(current, next) {
  run.currentRunLoop = next;
}

var Backburner = requireModule('backburner').Backburner;
var backburner = new Backburner(['sync', 'actions', 'destroy'], {
  sync: {
    before: beginPropertyChanges,
    after: endPropertyChanges
  },
  defaultQueue: 'actions',
  onBegin: onBegin,
  onEnd: onEnd,
  onErrorTarget: Ember,
  onErrorMethod: 'onerror'
});
{% endhighlight %}

  ---

  `backburner.js`

{% highlight javascript %}
function Backburner(queueNames, options) {
  this.queueNames = queueNames;
  this.options = options || {};
  if (!this.options.defaultQueue) {
    this.options.defaultQueue = queueNames[0];
  }
  this.instanceStack = [];
  this._debouncees = [];
  this._throttlers = [];
}

Backburner.prototype = {
  queueNames: null,
  options: null,
  currentInstance: null,
  instanceStack: null,
  begin: function() {
    var options = this.options,
        onBegin = options && options.onBegin,
        previousInstance = this.currentInstance;

    if (previousInstance) {
      this.instanceStack.push(previousInstance);
    }

    this.currentInstance = new DeferredActionQueues(this.queueNames, options);
    if (onBegin) {
      onBegin(this.currentInstance, previousInstance);
    }
  },

  end: function() {
    var options = this.options,
        onEnd = options && options.onEnd,
        currentInstance = this.currentInstance,
        nextInstance = null;

    var finallyAlreadyCalled = false;
    try {
      currentInstance.flush();
    } finally {
      if (!finallyAlreadyCalled) {
        finallyAlreadyCalled = true;

        this.currentInstance = null;

        if (this.instanceStack.length) {
          nextInstance = this.instanceStack.pop();
          this.currentInstance = nextInstance;
        }

        if (onEnd) {
          onEnd(currentInstance, nextInstance);
        }
      }
    }
  },
  ... // Other methods will be discussed later
};
{% endhighlight %}

  In `backburner.begin` and `backburner.end`, we can see that `instanceStack` is used as a stack of run loops. Together with the `onBegin` and `onEnd` functions provided in `run_loop.js`, it is able to handle run loops that are invoked during the execution of another run loop.

  Note that `defaultQueue` here is specified as `actions`. If we don't explicitly set `defaultQueue`, `queueNames[0]` will be chosen as `defaultQueue`. `defaultQueue` is the queue where jobs are scheduled into if no target queue is specified.

+ `Ember.run`

  `run_loop.js`

{% highlight javascript %}
function run() {
  return apply(backburner, backburner.run, arguments);
}
{% endhighlight %}

  ---

  `backburner.js`

{% highlight javascript %}
Backburner.prototype = {
  run: function(target, method /*, args */) {
    var onError = getOnError(this.options);

    this.begin();

    if (!method) {
      method = target;
      target = null;
    }

    if (isString(method)) {
      method = target[method];
    }

    var args = slice.call(arguments, 2);

    // guard against Safari 6's double-finally bug
    var didFinally = false;

    if (onError) {
      try {
        return method.apply(target, args);
      } catch(error) {
        onError(error);
      } finally {
        if (!didFinally) {
          didFinally = true;
          this.end();
        }
      }
    } else {
      try {
        return method.apply(target, args);
      } finally {
        if (!didFinally) {
          didFinally = true;
          this.end();
        }
      }
    }
  },
  ...
};

...

DeferredActionQueues.prototype = {
  flush: function() {
    var queues = this.queues,
        queueNames = this.queueNames,
        queueName, queue, queueItems, priorQueueNameIndex,
        queueNameIndex = 0, numberOfQueues = queueNames.length,
        options = this.options,
        onError = options.onError || (options.onErrorTarget && options.onErrorTarget[options.onErrorMethod]),
        invoke = onError ? this.invokeWithOnError : this.invoke;

    outerloop:
    while (queueNameIndex < numberOfQueues) {
      queueName = queueNames[queueNameIndex];
      queue = queues[queueName];
      queueItems = queue._queueBeingFlushed = queue._queue.slice();
      queue._queue = [];

      var queueOptions = queue.options, // TODO: write a test for this
          before = queueOptions && queueOptions.before,
          after = queueOptions && queueOptions.after,
          target, method, args, stack,
          queueIndex = 0, numberOfQueueItems = queueItems.length;

      if (numberOfQueueItems && before) { before(); }

      while (queueIndex < numberOfQueueItems) {
        target = queueItems[queueIndex];
        method = queueItems[queueIndex+1];
        args   = queueItems[queueIndex+2];
        stack  = queueItems[queueIndex+3]; // Debugging assistance

        if (isString(method)) { method = target[method]; }

        // method could have been nullified / canceled during flush
        if (method) {
          invoke(target, method, args, onError);
        }

        queueIndex += 4;
      }

      queue._queueBeingFlushed = null;
      if (numberOfQueueItems && after) { after(); }

      if ((priorQueueNameIndex = indexOfPriorQueueWithActions(this, queueNameIndex)) !== -1) {
        queueNameIndex = priorQueueNameIndex;
        continue outerloop;
      }

      queueNameIndex++;
    }
  },
  ...
};
{% endhighlight %}

  Asks `backburner` to initiate a new run loop to run the job in arguments. It wraps the execution in `begin` and `end` calls so that the `flush` call ensures that bindings are updated and events are responded.

+ `Ember.run.join`

  `run_loop.js`

{% highlight javascript %}
run.join = function(target, method /* args */) {
  if (!run.currentRunLoop) {
    return apply(Ember, run, arguments);
  }

  var args = slice.call(arguments);
  args.unshift('actions');
  apply(run, run.schedule, args);
};
{% endhighlight %}

  Works almost the same as `Ember.run`. However, if there already exists a run loop, instead of initiating a new one, `Ember.run.join` will schedule the job into the existing run loop in the `actions` queue.

+ `Ember.run.bind`

  `run_loop.js`

{% highlight javascript %}
run.bind = function(target, method /* args*/) {
  var args = slice.call(arguments);
  return function() {
    return apply(run, run.join, args.concat(slice.call(arguments)));
  };
};
{% endhighlight %}

   Provides a callback method for third party modules to use.

+ `Ember.run.begin` and `Ember.run.end`

  `run_loop.js`

{% highlight javascript %}
run.begin = function() {
  backburner.begin();
};

run.end = function() {
  backburner.end();
};
{% endhighlight %}

  Directly asks backburner to start or end the current run loop.

+ `Ember.run.schedule`, `Ember.run.scheduleOnce` and `Ember.run.once`

  `run_loop.js`

{% highlight javascript %}
run.schedule = function(queue, target, method) {
  checkAutoRun();
  apply(backburner, backburner.schedule, arguments);
};

run.scheduleOnce = function(queue, target, method) {
  checkAutoRun();
  return apply(backburner, backburner.scheduleOnce, arguments);
};

run.once = function(target, method) {
  checkAutoRun();
  var args = slice.call(arguments);
  args.unshift('actions');
  return apply(backburner, backburner.scheduleOnce, args);
};
{% endhighlight %}

  ---

  `backburner.js`

{% highlight javascript %}
Backburner.prototype = {
  defer: function(queueName, target, method /* , args */) {
    if (!method) {
      method = target;
      target = null;
    }

    if (isString(method)) {
      method = target[method];
    }

    var stack = this.DEBUG ? new Error() : undefined,
        args = arguments.length > 3 ? slice.call(arguments, 3) : undefined;
    if (!this.currentInstance) { createAutorun(this); }
    return this.currentInstance.schedule(queueName, target, method, args, false, stack);
  },

  deferOnce: function(queueName, target, method /* , args */) {
    if (!method) {
      method = target;
      target = null;
    }

    if (isString(method)) {
      method = target[method];
    }

    var stack = this.DEBUG ? new Error() : undefined,
        args = arguments.length > 3 ? slice.call(arguments, 3) : undefined;
    if (!this.currentInstance) { createAutorun(this); }
    return this.currentInstance.schedule(queueName, target, method, args, true, stack);
  },
  ...
};

Backburner.prototype.schedule = Backburner.prototype.defer;
Backburner.prototype.scheduleOnce = Backburner.prototype.deferOnce;

function createAutorun(backburner) {
  backburner.begin();
  backburner._autorun = global.setTimeout(function() {
    backburner._autorun = null;
    backburner.end();
  });
}

...

DeferredActionQueues.prototype = {
  schedule: function(queueName, target, method, args, onceFlag, stack) {
    var queues = this.queues,
        queue = queues[queueName];

    if (!queue) { throw new Error("You attempted to schedule an action in a queue (" + queueName + ") that doesn't exist"); }

    if (onceFlag) {
      return queue.pushUnique(target, method, args, stack);
    } else {
      return queue.push(target, method, args, stack);
    }
  },
  ...
};
{% endhighlight %}

  All these three methods schedule jobs to the current run loop. The different is that `scheduleOnce` ensures that the job is only executed once and that `once` is the convenient method for `scheduleOnce` on the `actions` queue. All of them invoke `backburner.schedule`, which asks the current run loop, `currentInstance` to push the job into corresponding queue.

+ `Ember.run.later`, `Ember.run.next`, `Ember.run.hasScheduledTimers`, `Ember.run.cancel` and `Ember.run.cancelTimers`

  `run_loop.js`

{% highlight javascript %}
run.later = function(target, method) {
  return apply(backburner, backburner.later, arguments);
};

run.next = function() {
  var args = slice.call(arguments);
  args.push(1);
  return apply(backburner, backburner.later, args);
};

run.hasScheduledTimers = function() {
  return backburner.hasTimers();
};

run.cancel = function(timer) {
  return backburner.cancel(timer);
};

run.cancelTimers = function () {
  backburner.cancelTimers();
};
{% endhighlight %}

  ---

  `backburner.js`

{% highlight javascript %}
DeferredActionQueues.prototype = {
  setTimeout: function() {
    var args = slice.call(arguments),
        length = args.length,
        method, wait, target,
        methodOrTarget, methodOrWait, methodOrArgs;

    if (length === 0) {
      return;
    } else if (length === 1) {
      method = args.shift();
      wait = 0;
    } else if (length === 2) {
      methodOrTarget = args[0];
      methodOrWait = args[1];

      if (isFunction(methodOrWait) || isFunction(methodOrTarget[methodOrWait])) {
        target = args.shift();
        method = args.shift();
        wait = 0;
      } else if (isCoercableNumber(methodOrWait)) {
        method = args.shift();
        wait = args.shift();
      } else {
        method = args.shift();
        wait =  0;
      }
    } else {
      var last = args[args.length - 1];

      if (isCoercableNumber(last)) {
        wait = args.pop();
      } else {
        wait = 0;
      }

      methodOrTarget = args[0];
      methodOrArgs = args[1];

      if (isFunction(methodOrArgs) || (isString(methodOrArgs) &&
                                      methodOrTarget !== null &&
                                      methodOrArgs in methodOrTarget)) {
        target = args.shift();
        method = args.shift();
      } else {
        method = args.shift();
      }
    }

    var executeAt = (+new Date()) + parseInt(wait, 10);

    if (isString(method)) {
      method = target[method];
    }

    var onError = getOnError(this.options);

    function fn() {
      if (onError) {
        try {
          method.apply(target, args);
        } catch (e) {
          onError(e);
        }
      } else {
        method.apply(target, args);
      }
    }

    // find position to insert
    var i = searchTimer(executeAt, timers);

    timers.splice(i, 0, executeAt, fn);

    updateLaterTimer(this, executeAt, wait);

    return fn;
  },

  cancelTimers: function() {
    var clearItems = function(item) {
      clearTimeout(item[2]);
    };

    each(this._throttlers, clearItems);
    this._throttlers = [];

    each(this._debouncees, clearItems);
    this._debouncees = [];

    if (this._laterTimer) {
      clearTimeout(this._laterTimer);
      this._laterTimer = null;
    }
    timers = [];

    if (this._autorun) {
      clearTimeout(this._autorun);
      this._autorun = null;
    }
  },

  hasTimers: function() {
    return !!timers.length || !!this._debouncees.length || !!this._throttlers.length || this._autorun;
  },

  cancel: function(timer) {
    var timerType = typeof timer;

    if (timer && timerType === 'object' && timer.queue && timer.method) { // we're cancelling a deferOnce
      return timer.queue.cancel(timer);
    } else if (timerType === 'function') { // we're cancelling a setTimeout
      for (var i = 0, l = timers.length; i < l; i += 2) {
        if (timers[i + 1] === timer) {
          timers.splice(i, 2); // remove the two elements
          return true;
        }
      }
    } else if (Object.prototype.toString.call(timer) === "[object Array]"){ // we're cancelling a throttle or debounce
      return this._cancelItem(findThrottler, this._throttlers, timer) ||
               this._cancelItem(findDebouncee, this._debouncees, timer);
    } else {
      return; // timer was null or not a timer
    }
  },
  ...
};

function updateLaterTimer(self, executeAt, wait) {
  if (!self._laterTimer || executeAt < self._laterTimerExpiresAt) {
    self._laterTimer = global.setTimeout(function() {
      self._laterTimer = null;
      self._laterTimerExpiresAt = null;
      executeTimers(self);
    }, wait);
    self._laterTimerExpiresAt = executeAt;
  }
}

function executeTimers(self) {
  var now = +new Date(),
      time, fns, i, l;

  self.run(function() {
    i = searchTimer(now, timers);

    fns = timers.splice(0, i);

    for (i = 1, l = fns.length; i < l; i += 2) {
      self.schedule(self.options.defaultQueue, null, fns[i]);
    }
  });

  if (timers.length) {
    updateLaterTimer(self, timers[0], timers[0] - now);
  }
}

function searchTimer(time, timers) {
  var start = 0,
      end = timers.length - 2,
      middle, l;

  while (start < end) {
    // since timers is an array of pairs 'l' will always
    // be an integer
    l = (end - start) / 2;

    // compensate for the index in case even number
    // of pairs inside timers
    middle = start + l - (l % 2);

    if (time >= timers[middle]) {
      start = middle + 2;
    } else {
      end = middle;
    }
  }

  return (time >= timers[start]) ? start + 2 : start;
}
{% endhighlight %}

  This set of functions, together with `Ember.run.throttle` and `Ember.run.debounce` below enables developers to schedule delayed jobs. These functions maintain an array, `timers`, to handle the delayed jobs. `timers` is an array of `(executeAt, fn)` pairs.

  + `searchTimer` finds the place in `timers` where a new delayed job should be inserted.
  + `executeTimers` runs all the delayed jobs that should be executed up till now.
  + `updateLaterTimer` uses the native `setTimeOut` function to schedule `executeTimers` at the time when next delayed job should be executed. (Note: I think there should be a `clearTimeout` here, but I'm not sure of this.)
  + `hasScheduledTimers` returns true if and only if there are delayed jobs pending.

  With these three functions,

  + `Ember.run.later` uses `searchTimer` to place the job in `timers` array and calls `updateLaterTimer` to update the next `executeTimers` call.
  + `Ember.run.next` schedules a job 1ms later. What it actually does is asking Ember to start a new run loop of the job right after the current one.
  + `Ember.run.hasScheduledTimers` returns the value of `hasScheduledTimers`.
  + `Ember.run.cancel` cancels a specific timer by removing it from `timers`and other relative collections.
  + `Ember.run.cancelTimers` cancels all the pending timers by clearing the `timers` array and other relative attributes.

+ `Ember.run.sync`

  `run_loop.js`

{% highlight javascript %}
run.sync = function() {
  if (backburner.currentInstance) {
    backburner.currentInstance.queues.sync.flush();
  }
};
{% endhighlight %}

  ---

  `backburner.js`

{% highlight javascript %}
Queue.prototype = {
  // TODO: remove me, only being used for Ember.run.sync
  flush: function() {
    var queue = this._queue,
        globalOptions = this.globalOptions,
        options = this.options,
        before = options && options.before,
        after = options && options.after,
        onError = globalOptions.onError || (globalOptions.onErrorTarget && globalOptions.onErrorTarget[globalOptions.onErrorMethod]),
        target, method, args, stack, i, l = queue.length;

    if (l && before) { before(); }
    for (i = 0; i < l; i += 4) {
      target = queue[i];
      method = queue[i+1];
      args   = queue[i+2];
      stack  = queue[i+3]; // Debugging assistance

      // TODO: error handling
      if (args && args.length > 0) {
        if (onError) {
          try {
            method.apply(target, args);
          } catch (e) {
            onError(e);
          }
        } else {
          method.apply(target, args);
        }
      } else {
        if (onError) {
          try {
            method.call(target);
          } catch(e) {
            onError(e);
          }
        } else {
          method.call(target);
        }
      }
    }
    if (l && after) { after(); }

    // check if new items have been added
    if (queue.length > l) {
      this._queue = queue.slice(l);
      this.flush();
    } else {
      this._queue.length = 0;
    }
  },
  ...
};
{% endhighlight %}

  `Ember.run.sync` is actually the only method that uses `flush` on a specific queue. The Ember developer team is trying to figure out a way to replace it with other functions. Nevertheless, it is still helpful since there indeed are places we need to synchronize the bindings.

+ `Ember.run.debounce` and `Ember.run.throttle`

  `run_loop.js`

{% highlight javascript %}
run.debounce = function() {
  return apply(backburner, backburner.debounce, arguments);
};

run.throttle = function() {
  return apply(backburner, backburner.throttle, arguments);
};
{% endhighlight %}

  ---

  `backburner.js`

{% highlight javascript %}
Queue.prototype = {
  debounce: function(target, method /* , args, wait, [immediate] */) {
    var self = this,
        args = arguments,
        immediate = pop.call(args),
        wait,
        index,
        debouncee,
        timer;

    if (isNumber(immediate) || isString(immediate)) {
      wait = immediate;
      immediate = false;
    } else {
      wait = pop.call(args);
    }

    wait = parseInt(wait, 10);
    // Remove debouncee
    index = findDebouncee(target, method, this._debouncees);

    if (index > -1) {
      debouncee = this._debouncees[index];
      this._debouncees.splice(index, 1);
      clearTimeout(debouncee[2]);
    }

    timer = global.setTimeout(function() {
      if (!immediate) {
        self.run.apply(self, args);
      }
      var index = findDebouncee(target, method, self._debouncees);
      if (index > -1) {
        self._debouncees.splice(index, 1);
      }
    }, wait);

    if (immediate && index === -1) {
      self.run.apply(self, args);
    }

    debouncee = [target, method, timer];

    self._debouncees.push(debouncee);

    return debouncee;
  },

  throttle: function(target, method /* , args, wait, [immediate] */) {
    var self = this,
        args = arguments,
        immediate = pop.call(args),
        wait,
        throttler,
        index,
        timer;

    if (isNumber(immediate) || isString(immediate)) {
      wait = immediate;
      immediate = true;
    } else {
      wait = pop.call(args);
    }

    wait = parseInt(wait, 10);

    index = findThrottler(target, method, this._throttlers);
    if (index > -1) { return this._throttlers[index]; } // throttled

    timer = global.setTimeout(function() {
      if (!immediate) {
        self.run.apply(self, args);
      }
      var index = findThrottler(target, method, self._throttlers);
      if (index > -1) {
        self._throttlers.splice(index, 1);
      }
    }, wait);

    if (immediate) {
      self.run.apply(self, args);
    }

    throttler = [target, method, timer];

    this._throttlers.push(throttler);

    return throttler;
  },
  ...
};
{% endhighlight %}

  #### What do they do?

  + `Ember.run.debounce` ensures a job is never executed within a specific time interval.
  + `Ember.run.throttle` ensures a job is never executed more frequently than a specific time interval.

  #### How do they do it?

  The functions use `_debouncees` and `_throttlers` to maintain the list of jobs that are using these functions. They also use the native `setTimeOut` to schedule of calls.

  Note: `Ember.run.cancel` can also use return values of these functions to cancel debounce and throttle timers.

+ `Ember.run._addQueue`

  `run_loop.js`

{% highlight javascript %}
run._addQueue = function(name, after) {
  if (indexOf.call(run.queues, name) === -1) {
    run.queues.splice(indexOf.call(run.queues, after)+1, 0, name);
  }
};
{% endhighlight %}

  Adds a new queue, `name`. All jobs of this queue is executed after `after`. `ember-routing` and `ember-views` use this method to add `render`, `afterRender` and `routerTransitions` queue.

## How is the run loop used?

Using the run loop is amazingly simple. For instance, in `ember-views/lib/views/view.js`,

{% highlight javascript %}
 _insertElementLater: function(fn) {
  this._scheduledInsert = run.scheduleOnce('render', this, '_insertElement', fn);
},

_insertElement: function (fn) {
  this._scheduledInsert = null;
  this.currentState.insertElement(this, fn);
},

...

replaceIn: function(target) {
  Ember.assert("You tried to replace in (" + target + ") but that isn't in the DOM", jQuery(target).length > 0);
  Ember.assert("You cannot replace an existing Ember.View. Consider using Ember.ContainerView instead.", !jQuery(target).is('.ember-view') && !jQuery(target).parents().is('.ember-view'));

  this._insertElementLater(function() {
    jQuery(target).empty();
    this.$().appendTo(target);
  });
}
{% endhighlight %}

`replaceIn` uses the `Ember.run.scheduleOnce` method to schedule a replace in the `render` queue.

## When does Ember.js start a run loop?

As we have seen in the above code, Ember.js uses the run loop everywhere. But besides these run loop calls from inside Ember.js, when does Ember.js start a run loop?

To answer this question, let's take a look at this piece of code from `ember-views/lib/system/event_dispatcher.js`,

{% highlight javascript %}
_dispatchEvent: function(object, evt, eventName, view) {
  var result = true;

  var handler = object[eventName];
  if (typeOf(handler) === 'function') {
    result = run(object, handler, evt, view);
    // Do not preventDefault in eventManagers.
    evt.stopPropagation();
  }
  else {
    result = this._bubbleEvent(view, evt, eventName);
  }

  return result;
},

_bubbleEvent: function(view, evt, eventName) {
  return run(view, view.handleEvent, eventName, evt);
}
{% endhighlight %}

Notice the `Ember.run` calls in these functions. Every event sent to Ember.js initiates a run loop. The event handle will then possibly trigger some binding jobs and render jobs. After all the jobs are down (no pending jobs), the event is successfully handled and the run loop is completed.

## Conclusion

The run loop is a smart idea to avoid unneeded computations, which is largely favorable in web app development. Also, Ember.js provides a detailed API for the developers to fully take advantage of the run loop structure.

