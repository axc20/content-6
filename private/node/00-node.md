# Node

## querystring, process, timers

```js
const querystring = require('querystring');
const util = require('util');

console.log(querystring.parse('foo=bar&abc=xyz&abc=123'));
console.log(
  querystring.stringify({
    foo: 'bar',
    baz: ['leeroy jenkins', 'quux'],
    corge: ''
  })
);

// the process object is a global that provides info about and control over the current Node.js process
// console.log(process);

// the process object is an instance of EventEmitter
process.on('beforeExit', code => {
  console.log('Process beforeExit event with code: ', code);
});
process.on('exit', code => {
  console.log('Process exit event with code: ', code);
});

console.log('This message is displayed first.');

// scheduling timers
// a timer in Node.js is an internal construct that calls a given fn after a certain period of time
// when a timer's fn is called varies depending on which method is used to create timer and what other work
// the event loop is doing

// when multiple calls to setImmediate() are made, the callback fns are queued for execution in order of creation
// the entire callback queue is processed every event loop iteration
const setImmediatePromise = util.promisify(setImmediate);

async function timerExample() {
  console.log('Before I/O callbacks');
  await setImmediatePromise();
  console.log('After I/O callbacks');
}

timerExample();

// setInterval(callback[, delay[, ...args ]])
// schedules repeated execution callback every delay milliseconds

// setTimeout(callback[, delay[, ...args ]])
// schedules execution of a one-time callback after delay milliseconds

const setTimeoutPromise = util.promisify(setTimeout);

setTimeoutPromise(1000, 'foobar').then(value => {
  // value === 'foobar' (passing values is optional)
  // this is executed after about 1s
  console.log('execute 1s', value);
});

setTimeoutPromise();

// cancelling timers
// setImmediate(), setInterval(), setTimeout() return objects that represetn scheduled timers
// these can be used to cancel the timer and prevent it from triggering

// clearImmediate(immediate)
// clearInterval(timeout)
// clearTimeout(timeout)
```
