# taskjob
My take at observable, cancelable asynchronous operations…


## Motivation

I'm trying to find a good asynchronous task library for tasks that supports:

- Promises. Who wants to mess with callback nowadays?
- Progress. I want to be able to be notified of progress.
- Nesting. Some tasks are composed of sub-tasks.
- Cancellation. What if I don't want the result anymore?

Nongoals: I don't care about

- Chaining.

I came across whatwg/fetch#27 and that discussion is too long. Basically, I don't want to to extend Promises with cancellations.


## What I am thinking...

### Creating a task:

```js
function createTask() {
  return new Task(function worker(job) {
    return new Promise(function(resolve, reject) {
      var timeout = setTimeout(resolve, 2000)
      job.oncancel = function(reason) {
        // I lost my job, my work is worthless.
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

You can only delegate one at a time.


### Manual Check:

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
