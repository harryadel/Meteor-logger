[![support](https://img.shields.io/badge/support-GitHub-white)](https://github.com/sponsors/dr-dimitru)
[![support](https://img.shields.io/badge/support-PayPal-white)](https://paypal.me/veliovgroup)
<a href="https://ostr.io/info/built-by-developers-for-developers">
  <img src="https://ostr.io/apple-touch-icon-60x60.png" height="20">
</a>

# Isomorphic logging driver

Logger driver for Meteor.js with different adapters. To use this package install an adapter (*separately*):

- [File](https://atmospherejs.com/ostrio/loggerfile) - Store application log messages into the file (FS), log rotation included;
- [Mongo](https://atmospherejs.com/ostrio/loggermongo) - Store application log messages into MongoDB;
- [Console](https://atmospherejs.com/ostrio/loggerconsole) - Print Client's application log messages to Server's console, messages colorized for better readability.

## Features:

- 👷‍♂️ 100% tests coverage;
- 💪 Flexible log level filters, ex: write `FATAL`, `ERROR`, and `WARN` to file, `DEBUG` to console, and all other to MongoDB;
- 👨‍💻 `userId` is automatically passed and logged, data is associated with logged-in user;
- 📟 Pass logs from *Client* to *Server*;
- 🕷 Catch all browser's errors.

![Meteor Logger Library](https://raw.githubusercontent.com/VeliovGroup/Meteor-logger/master/meteor-logger.jpg)

## Installation:

```shell
meteor add ostrio:logger
```

### ES6 Import:

```js
import { Logger } from 'meteor/ostrio:logger';
```

## Usage

Initiate a logger and pass it into the adapter constructor

### Logger [*Isomorphic*]

```js
const log = new Logger();

/* Activate adapters with default settings */
/* meteor add ostrio:loggerfile */
new LoggerFile(log).enable();
/* meteor add ostrio:loggermongo */
new LoggerMongo(log).enable();
/* meteor add ostrio:loggerconsole */
new LoggerConsole(log).enable();

/* Log message
 * message {String|Number} - Any text message
 * data    {Object} - [optional] Any additional info as object
 * userId  {String} - [optional] Current user id
 */
log.info(message, data, userId);
log.debug(message, data, userId);
log.error(message, data, userId);
log.fatal(message, data, userId);
log.warn(message, data, userId);
log.trace(message, data, userId);
log._(message, data, userId); //--> Plain shortcut

/* Use with throw */
throw log.error(message, data, userId);
```

### Catch-all Client's errors example: [*CLIENT*]

```js
/* Store original window.onerror */
const _GlobalErrorHandler = window.onerror;

window.onerror = function (msg, url, line) {
  log.error(msg, {file: url, onLine: line});
  if (_GlobalErrorHandler) {
    _GlobalErrorHandler.apply(this, arguments);
  }
};
```

### Catch-all Server's errors example: [*Server*]

```js
const bound = Meteor.bindEnvironment((callback) => {callback();});
process.on('uncaughtException', function (err) {
  bound(() => {
    log.error('Server Crashed!', err);
    console.error(err.stack);
    process.exit(7);
  });
});
```

### Catch-all Meteor's errors example: [*Server*]

```js
// store original Meteor error
const originalMeteorDebug = Meteor._debug;
Meteor._debug = (message, stack) => {
  const error = new Error(message);
  error.stack = stack;
  log.error('Meteor Error!', error);
  return originalMeteorDebug.apply(this, arguments);
};
```

### Register new adapter [*Isomorphic*]

*Mainly should be used by adapter developers, a.k.a. developer API.*

```js
/**
 * Logger#add() — register new adapter
 * 
 *  Emitter function
 * name        {String}    - Adapter name
 * emitter     {Function}  - Function called on Meteor.log...
 * init        {Function}  - Adapter initialization function
 * denyClient  {Boolean}   - Strictly deny execution on client
 * denyServer  {Boolean}   - Strictly deny execution on server
 * Example: log.add(name, emitter, init, denyClient, denyServer);
 */

const emitter = (level, message, data, userId) => {
  /* .. do something with a message .. */
};

const init = () => {
  /* Initialization function */
  /* For example create a collection */
  log.collection = new Meteor.Collection('logs');
};

log.add('AdapterName', emitter, init, true, false);
```

### Enable/disable adapter and set its settings [*Isomorphic*]

```js
/**
 * Logger#rule() — register adapter default rules
 *
 * name    {String} - Adapter name
 * options {Object} - Settings object, accepts next properties:
 * options.enable {Boolean} - Enable/disable adapter
 * options.filter {Array}   - Array of strings, accepts:
 *                            'ERROR', 'FATAL', 'WARN', 'DEBUG', 'INFO', '*'
 *                            in lowercase and uppercase
 *                            default: ['*'] - Accept all
 * options.client {Boolean} - Allow execution on Client
 * options.server {Boolean} - Allow execution on Server
 * Example: log.rule(name, options);
 */

/* Example: */
log.rule('AdapterName', {
  enable: true,
  filter: ['ERROR', 'FATAL', 'WARN'],
  client: false, /* Allow to call, but not execute on Client */
  server: true   /* Calls from client will be executed on Server */
});
```

## Running Tests

1. Clone this package
2. In Terminal (*Console*) go to directory where package is cloned
3. Then run:

### Meteor/Tinytest

```shell
# Default
meteor test-packages ./

# With custom port
meteor test-packages ./ --port 8888

# With local MongoDB and custom port
MONGO_URL="mongodb://127.0.0.1:27017/logger-tests" meteor test-packages ./ --port 8888
```

## Support our open source contribution:

- [Sponsor via GitHub](https://github.com/sponsors/dr-dimitru)
- [Support via PayPal](https://paypal.me/veliovgroup)
- Use [ostr.io](https://ostr.io) — [Monitoring](https://snmp-monitoring.com), [Analytics](https://ostr.io/info/web-analytics), [WebSec](https://domain-protection.info), [Web-CRON](https://web-cron.info) and [Pre-rendering](https://prerendering.com) for a website
