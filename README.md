## Add dynamic instrumentation to emitters

`emitter-listener` does a bunch of the work necessary to wrap event emitters
methods in wrappers you provide:

```javascript
var EventEmitter = require('events').EventEmitter;
var wrapEmitter = require('emitter-listener');

var ee = new EventEmitter();

var id = 0;

wrapEmitter(
  ee,
  function mark(listener) {
    listener.id = id++;
  },
  function prepare(listener) {
    console.log('listener id is %d', listener.id);
  }
);
```

### Mandatory disclaimer

There are times when it's necessary to monkeypatch default behavior in
JavaScript and Node. However, changing the behavior of the runtime on the fly
is rarely a good idea, and you should be using this module because you need to,
not because it seems like fun.

#### wrapEmitter(emitter, mark, prepare)

Wrap an EventEmitter's event listeners. Each listener will be passed to
`mark` when it is registered with `.addListener()` or `.on()`, and then
each listener is passed to `prepare` to be wrapped before it's called
by the `.emit()` call. `wrapEmitter` deals with the single listener
vs array of listeners logic, and also ensures that edge cases like
`.removeListener()` being called from within an `.emit()` for the same
event type is handled properly.

The wrapped EE can be restored to its pristine state by using
emitter.__unwrap(), but this should only be used if you *really* know
what you're doing.

Note: When wrapping the same event emitter multiple times, the provided `mark`
callbacks will be called in the order in which they have been registered but
the provided `prepare` callbacks will be called in the reverse order. That is,
the `prepare` argument given to the last `wrapEmitter` call will be called first
while the `prepare` given to the first `wrapEmitter` call will be called last.
