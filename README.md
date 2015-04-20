# new Task :: ((Job) -> Promise) -> Task

My take at observable, cancelable asynchronous operations…


## Motivation and Quality Attributes

I'm trying to find a good asynchronous task library for tasks that supports:

- __Promises.__ Who wants to mess with callback nowadays?
- __Progress.__ I want to be able to be notified of progress.
- __Nesting.__ Some tasks are composed of sub-tasks.
- __Cancellation.__ What if I don't want the result anymore?

Nongoals: I don't care about

- __Chaining.__ Tasks don't need to be chainable.
- __Uncancellation.__ If a task is canceled, then it is indeed canceled!

I came across whatwg/fetch#27, but that discussion is too long.
Basically, I don't want to to extend Promises with cancellations.


## What I am thinking...

### Creating a task:

```js
function createTask() {
  return new Task(function worker(job) {
    return new Promise(function(resolve, reject) {
      var timeout = setTimeout(resolve, 2000)
      job.oncancel = function(reason) {
        // I lost my job, so my work is worthless now.
        clearTimeout(timeout)
      }
    })
  })
}
```

### Delegation:

```js
function createTask() {
  return new Task(function worker(job) {
    return job.delegate(anotherTask)
    .then(function(result) { return job.delegate(createTask2(result)) })
    .then(function(result) { return workWith(result) })
  })
}
```

You can not delegate concurrently but can delegate multiple sub-tasks at the same time (but have to be done at once).


### Manual Checking:

```js
function createTask() {
  return new Task(function worker(job) {
    return doStuff1()
    .then(job.ensure)
    .then(doStuff2)
    .then(job.ensure)
  })
}
```

You can only delegate one at a time.
