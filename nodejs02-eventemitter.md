# Event emitter

```js
const EventEmitter = require('node:events');

class Emitter extends EventEmitter {}

const myE = new Emitter();

myE.on('foo', () => {
  console.log('Event "foo" occurred');
});

myE.emit('foo');
```

## EventEmitter

nothing to do with event loop
it's pure javascript

when calling the on method, we store in an object the callback function with its name
when calling the emit('foo') we'll run all objects 'foo' in the object

myE.once("hello") -> will be emit only once, after it runs, the listener is removed
