# ya-sqs
[![Build Status](https://travis-ci.org/neoziro/ya-sqs.svg?branch=master)](https://travis-ci.org/neoziro/ya-sqs)
[![Dependency Status](https://david-dm.org/neoziro/ya-sqs.svg?theme=shields.io)](https://david-dm.org/neoziro/ya-sqs)
[![devDependency Status](https://david-dm.org/neoziro/ya-sqs/dev-status.svg?theme=shields.io)](https://david-dm.org/neoziro/ya-sqs#info=devDependencies)

Yet Another AWS SQS wrapper with pull (long polling), push, error management and promises.

## Install

```
npm install ya-sqs
```

## Usage

```js
var sqs = require('ya-sqs');

var queue = sqs.createQueue({
  aws: {
    region: 'eu-west-1',
    accessKeyId: '...',
    secretAccessKey: '...'
  },
  name: 'ya-sqs-test'
});

// Push message in the queue.
queue.push({foo: 'bar'}, function (err) {
  console.log('Message pushed.');
});

// Pull message.
queue.pull(function (message, next) {
  console.log('Message pulled.');
  next(); // Remove message from queue and pull next.
});
```

### Example with promises

```js
var sqs = require('ya-sqs');

var queue = sqs.createQueue({
  aws: {region: 'eu-west-1'},
  name: 'ya-sqs-test'
});

// Push message in the queue.
queue.push({foo: 'bar'}).then(function () {
  console.log('Message pushed.');
});

// Pull message.
queue.pull(function (message) {
  console.log('Message pulled.');
  return Promise.resolve(); // Remove message from queue and pull next.
});
```

### sqs.createQueue(options)

Create a new queue. If you provide a name, the queue will be automatically created.

```
{object} options
{string} [options.name] Name of the queue
{string} [options.url] Url of the queue
{string} [options.aws] AWS config
{string} [options.waitTime=20] Polling time
{string} [options.formatter] Formatter (default JSON)
```

```js
var queue = sqs.createQueue({
  url: 'https://sqs.eu-west-1.amazonaws.com/279100839409/ya-sqs-test',
  waitTime: 10,
  aws: {
    region: 'eu-west',
    sslEnabled: true
  }
});
```

### queue.push(message, [cb])

Push a new message in the queue.

Promises:

```js
queue.push('hello').then(function () {
  console.log('Message pushed.');
}, function (err) {
  console.log('Error during push.');
});
```

Callback:

```js
queue.push('hello', function (err) {
  if (err) return console.log('Error during push.');
  console.log('Message pushed.');
});
```

### queue.mpush(messages, [cb])

Push message using batch method.

Promises:

```js
queue.mpush(['hello', 'world']).then(function () {
  console.log('Messages pushed.');
}, function (err) {
  console.log('Error during push.');
});
```

Callback:

```js
queue.mpush(['hello', 'world'], function (err) {
  if (err) return console.log('Error during push.');
  console.log('Message pushed.');
});
```

### queue.pull(handler, [options])

Pull message from the queue. When the promise returned is resolved or when next is called, the message will be remove and an other message will be pulled. If an error is sent, the error will be emitted and the message will not be removed.

Options:

```
@param {object} [options]
@param {number} [options.maxNumberOfMessages=10] Max number of messages
```

Promises:

```js
queue.pull(function (message) {
  console.log('Message pulled.', message);
  return Promise.resolve();
});
```

Callback:

```js
queue.pull(function (message, next) {
  console.log('Message pulled.', message);
  next();
});
```

### queue.close([cb])

Stop pulling events, the promise is resolved when current messages have been processed.

```js
queue.close().then(...);
```

## Formatters

The default formatter for the queue is JSON, you can write a custom formatter for messages. To do it, please refer to the [JSON formatter](https://github.com/neoziro/ya-sqs/blob/master/lib/json-formatter.js).

Example that specify a raw formatter (the message will not be formatted):

```js
ws.createQueue({
  formatter: {
    format: function (message) {
      return Promise.resolve(message);
    },
    parse: function (message) {
      return Promise.resolve(message);  
    }
  }
})
```

## Events

### "error"

Emitted when ReceiveMessage command has an error, when a message can't be parsed and when you return an error in the "next" method.

```js
queue.on('error', function (err) {
  // ...
});
```

### "message pushed"

Emitted when a message is pushed. The message in argument is not formatted.

```js
queue.on('message pushed', function (message) {
  // ...
});
```

### "message received"

Emitted when a new message is received. The message in argument is not parsed.

```js
queue.on('message received', function (message) {
  // ...
});
```

### "message processed"

Emitted when a message is processed. The message in argument is not parsed.

```js
queue.on('message processed', function (message) {
  // ...
});
```


## License

MIT
