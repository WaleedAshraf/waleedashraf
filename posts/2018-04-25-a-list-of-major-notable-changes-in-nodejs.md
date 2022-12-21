---
layout: post
title: A list of major/notable changes in Node.js versions.
excerpt: "Help you migrate to latest version."
modified: 2018-04-25T02:23:25+05:00
author: waleed_ashraf
image:
  path: ../images/cover_arch.jpg
  feature: cover_arch.jpg
  credit: thomashawk
  creditlink: http://thomashawk.com/
comments: true
share: true
---

This article contains info on major/notable changes in Node.js versions to help you migrate. It was written for the [Release Updates](https://medium.com/the-node-js-collection/april-2018-release-updates-from-the-node-js-project-71687e1f7742) article by Myles Borins in Nodejs Collection.

I have listed major changes in

- <a href="#5.0.0">5.0.0</a><br/>
- <a href="#6.0.0">6.0.0</a><br/>
- <a href="#7.0.0">7.0.0</a><br/>
- <a href="#8.0.0">8.0.0</a><br/>
- <a href="#9.0.0">9.0.0</a><br/>
- <a href="#10.0.0">10.0.0</a><br/>

Note that changes in Node.js versions are cumulative.
So, If you are planning to upgrade from 4.x to 8.x, you should take a look at all changes in 5.0.0 + 6.0.0 + 7.0.0 + 8.0.0.

Similarly, If you plan to upgrade from 4.x to 6.x, you should only check changes in 5.0.0 + 6.0.0.

I tried to add all major changes from node/changelog, but in-case I missed something, feel free to open a PR in [WaleedAshraf/nodejs-major-changes](https://github.com/WaleedAshraf/nodejs-major-changes)

<a id="5.0.0"></a>
# Version 5.0.0

* **buffer**: _(Breaking)_ Removed both `'raw'` and `'raws'` encoding types from `Buffer`, these have been deprecated for a long time (Sakthipriyan Vairamani) [#2859](https://github.com/nodejs/node/pull/2859).
* **console**: _(Breaking)_ Values reported by `console.time()` now have 3 decimals of accuracy added (Michaël Zasso) [#3166](https://github.com/nodejs/node/pull/3166).
* **fs**:
  - `fs.readFile*()`, `fs.writeFile*()`, and `fs.appendFile*()` now also accept a file descriptor as their first argument (Johannes Wüller) [#3163](https://github.com/nodejs/node/pull/3163).
  - _(Breaking)_ In `fs.readFile()`, if an encoding is specified and the internal `toString()` fails the error is no longer _thrown_ but is passed to the callback (Evan Lucas) [#3485](https://github.com/nodejs/node/pull/3485).
  - _(Breaking)_ In `fs.read()` (using the `fs.read(fd, length, position, encoding, callback)` form), if the internal `toString()` fails the error is no longer _thrown_ but is passed to the callback (Evan Lucas) [#3503](https://github.com/nodejs/node/pull/3503).
* **http**:
  - Fixed a bug where pipelined http requests would stall (Fedor Indutny) [#3342](https://github.com/nodejs/node/pull/3342).
  - _(Breaking)_ When parsing HTTP, don't add duplicates of the following headers: `Retry-After`, `ETag`, `Last-Modified`, `Server`, `Age`, `Expires`. This is in addition to the following headers which already block duplicates: `Content-Type`, `Content-Length`, `User-Agent`, `Referer`, `Host`, `Authorization`, `Proxy-Authorization`, `If-Modified-Since`, `If-Unmodified-Since`, `From`, `Location`, `Max-Forwards` (James M Snell) [#3090](https://github.com/nodejs/node/pull/3090).
  - _(Breaking)_ The `callback` argument to `OutgoingMessage#setTimeout()` must be a function or a `TypeError` is thrown (James M Snell) [#3090](https://github.com/nodejs/node/pull/3090).
  - _(Breaking)_ HTTP methods and header names must now conform to the RFC 2616 "token" rule, a list of allowed characters that excludes control characters and a number of _separator_ characters. Specifically, methods and header names must now match ```/^[a-zA-Z0-9_!#$%&'*+.^`|~-]+$/``` or a `TypeError` will be thrown (James M Snell) [#2526](https://github.com/nodejs/node/pull/2526).
* **node**:
  - _(Breaking)_ Deprecated the `_linklist` module (Rich Trott) [#3078](https://github.com/nodejs/node/pull/3078).
  - _(Breaking)_ Removed `require.paths` and `require.registerExtension()`, both had been previously set to throw `Error` when accessed (Sakthipriyan Vairamani) [#2922](https://github.com/nodejs/node/pull/2922).
* **npm**: Upgraded to version 3.3.6 from 2.14.7, see https://github.com/npm/npm/releases/tag/v3.3.6 for more details. This is a major version bump for npm and it has seen a significant amount of change. Please see the original [npm v3.0.0 release notes](https://github.com/npm/npm/blob/master/CHANGELOG.md#v300-2015-06-25) for a list of major changes (Rebecca Turner) [#3310](https://github.com/nodejs/node/pull/3310).
* **src**: _(Breaking)_ Bumped `NODE_MODULE_VERSION` to `47` from `46`, this is necessary due to the V8 upgrade. Native add-ons will need to be recompiled (Rod Vagg) [#3400](https://github.com/nodejs/node/pull/3400).
* **timers**: Attempt to reuse the timer handle for `setTimeout().unref()`. This fixes a long-standing known issue where unrefed timers would perviously hold `beforeExit` open (Fedor Indutny) [#3407](https://github.com/nodejs/node/pull/3407).
* **tls**:
  - Added ALPN Support (Shigeki Ohtsu) [#2564](https://github.com/nodejs/node/pull/2564).
  - TLS options can now be passed in an object to `createSecurePair()` (Коренберг Марк) [#2441](https://github.com/nodejs/node/pull/2441).
  - _(Breaking)_ The default minimum DH key size for `tls.connect()` is now 1024 bits and a warning is shown when DH key size is less than 2048 bits. This a security consideration to prevent "logjam" attacks. A new `minDHSize` TLS option can be used to override the default. (Shigeki Ohtsu) [#1831](https://github.com/nodejs/node/pull/1831).
* **util**:
  - _(Breaking)_ `util.p()` was deprecated for years, and has now been removed (Wyatt Preul) [#3432](https://github.com/nodejs/node/pull/3432).
  - _(Breaking)_ `util.inherits()` can now work with ES6 classes. This is considered a breaking change because of potential subtle side-effects caused by a change from directly reassigning the prototype of the constructor using `ctor.prototype = Object.create(superCtor.prototype, { constructor: { ... } })` to using `Object.setPrototypeOf(ctor.prototype, superCtor.prototype)` (Michaël Zasso) [#3455](https://github.com/nodejs/node/pull/3455).
* **v8**: _(Breaking)_ Upgraded to 4.6.85.25 from 4.5.103.35  (Ali Ijaz Sheikh) [#3351](https://github.com/nodejs/node/pull/3351).
  - Implements the spread operator, see https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator for further information.
  - Implements `new.target`, see https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target for further information.
* **zlib**: Decompression now throws on truncated input (e.g. unexpected end of file) (Yuval Brik) [#2595](https://github.com/nodejs/node/pull/2595).

### Known issues

* Surrogate pair in REPL can freeze terminal. [#690](https://github.com/nodejs/node/issues/690)
* Calling `dns.setServers()` while a DNS query is in progress can cause the process to crash on a failed assertion. [#894](https://github.com/nodejs/node/issues/894)
* `url.resolve` may transfer the auth portion of the url when resolving between two full hosts, see [#1435](https://github.com/nodejs/node/issues/1435).
* Unicode characters in filesystem paths are not handled consistently across platforms or Node.js APIs. See [#2088](https://github.com/nodejs/node/issues/2088), [#3401](https://github.com/nodejs/node/issues/3401) and [#3519](https://github.com/nodejs/node/issues/3519).

<a id="6.0.0"></a>
# Version 6.0.0

The following significant changes have been made since the previous Node.js
v5.0.0 release.

* Buffer
  * New Buffer constructors have been added
    [#4682](https://github.com/nodejs/node/pull/4682) and
    [#5833](https://github.com/nodejs/node/pull/5833).
  * Existing `Buffer()` and `SlowBuffer()` constructors have been deprecated
    in docs [#4682](https://github.com/nodejs/node/pull/4682) and
    [#5833](https://github.com/nodejs/node/pull/5833).
  * Previously deprecated Buffer APIs are removed
    [#5048](https://github.com/nodejs/node/pull/5048),
    [#4594](https://github.com/nodejs/node/pull/4594).
  * Improved error handling [#4514](https://github.com/nodejs/node/pull/4514).
  * The `Buffer.prototype.lastIndexOf()` method has been added
    [#4846](https://github.com/nodejs/node/pull/4846).
* Cluster
  * Worker emitted as first argument in 'message' event
    [#5361](https://github.com/nodejs/node/pull/5361).
  * The `worker.exitedAfterDisconnect` property replaces `worker.suicide`
    [#3743](https://github.com/nodejs/node/pull/3743).
* Console
  * Calling `console.timeEnd()` with an unknown label now emits a process
    warning rather than throwing
    [#5901](https://github.com/nodejs/node/pull/5901).
* Crypto
  * Improved error handling [#3100](https://github.com/nodejs/node/pull/3100),
    [#5611](https://github.com/nodejs/node/pull/5611).
  * Simplified Certificate class bindings
    [#5382](https://github.com/nodejs/node/pull/5382).
  * Improved control over FIPS mode
    [#5181](https://github.com/nodejs/node/pull/5181).
  * pbkdf2 digest overloading is deprecated
    [#4047](https://github.com/nodejs/node/pull/4047).
* Dependencies
  * Reintroduce shared c-ares build support
    [#5775](https://github.com/nodejs/node/pull/5775).
  * V8 updated to 5.0.71.35 [#6372](https://github.com/nodejs/node/pull/6372).
* DNS
  * Add `dns.resolvePtr()` API to query plain DNS PTR records
    [#4921](https://github.com/nodejs/node/pull/4921).
* Domains
  * Clear stack when no error handler
  [#4659](https://github.com/nodejs/node/pull/4659).
* Events
  * The `EventEmitter.prototype._events` object no longer inherits from
    Object.prototype [#6092](https://github.com/nodejs/node/pull/6092).
  * The `EventEmitter.prototype.prependListener()` and
    `EventEmitter.prototype.prependOnceListener()` methods have been added
    [#6032](https://github.com/nodejs/node/pull/6032).
* File System
  * The `fs.realpath()` and `fs.realpathSync()` methods have been updated
    to use a more efficient libuv-based implementation. This change includes
    the removal of the `cache` argument and the method can throw new errors
    [#3594](https://github.com/nodejs/node/pull/3594).
  * FS apis can now accept and return paths as Buffers
    [#5616](https://github.com/nodejs/node/pull/5616).
  * Error handling and type checking improvements
    [#5616](https://github.com/nodejs/node/pull/5616),
    [#5590](https://github.com/nodejs/node/pull/5590),
    [#4518](https://github.com/nodejs/node/pull/4518),
    [#3917](https://github.com/nodejs/node/pull/3917).
  * fs.read's string interface is deprecated
    [#4525](https://github.com/nodejs/node/pull/4525).
* HTTP
  * 'clientError' can now be used to return custom errors from an
    HTTP server [#4557](https://github.com/nodejs/node/pull/4557).
* Modules
  * Current directory is now prioritized for local lookups
    [#5689](https://github.com/nodejs/node/pull/5689).
  * Symbolic links are preserved when requiring modules
    [#5950](https://github.com/nodejs/node/pull/5950).
* Net
  * DNS hints no longer implicitly set
    [#6021](https://github.com/nodejs/node/pull/6021).
  * Improved error handling and type checking
    [#5981](https://github.com/nodejs/node/pull/5981),
    [#5733](https://github.com/nodejs/node/pull/5733),
    [#2904](https://github.com/nodejs/node/pull/2904).
* npm
  * Running npm requires the node binary to be in the path
    [#6098](https://github.com/nodejs/node/pull/6098).
* OS X
  * MACOSX_DEPLOYMENT_TARGET has been bumped up to 10.7
    [#6402](https://github.com/nodejs/node/pull/6402).
* Path
  * Improved type checking [#5348](https://github.com/nodejs/node/pull/5348).
* Process
  * Introduce process warnings API
    [#4782](https://github.com/nodejs/node/pull/4782).
  * Throw exception when non-function passed to nextTick
    [#3860](https://github.com/nodejs/node/pull/3860).
* Querystring
  * The object returned by `querystring.parse()` no longer inherits from
    Object.prototype [#6055](https://github.com/nodejs/node/pull/6055).
* Readline
  * Key info is emitted unconditionally
    [#6024](https://github.com/nodejs/node/pull/6024).
  * History can now be explicitly disabled
    [#6352](https://github.com/nodejs/node/pull/6352).
* REPL
  * Assignment to `_` will emit a warning
    [#5535](https://github.com/nodejs/node/pull/5535).
  * Expressions will no longer be completed when eval fails
    [#6328](https://github.com/nodejs/node/pull/6328).
* Timers
  * Fail early when callback is not a function
    [#4362](https://github.com/nodejs/node/pull/4362).
* Streams
  * `null` is now an invalid chunk to write in object mode
    [#6170](https://github.com/nodejs/node/pull/6170).
* TLS
  * Rename 'clientError' to 'tlsClientError'
    [#4557](https://github.com/nodejs/node/pull/4557).
  * SHA1 used for sessionIdContext
    [#3866](https://github.com/nodejs/node/pull/3866).
* TTY
  * Previously deprecated setRawMode wrapper is removed
    [#2528](https://github.com/nodejs/node/pull/2528).
* URL
  * Username and password will be dropped by `url.resolve()` if the host
    changes [#1480](https://github.com/nodejs/node/pull/1480).
* Util
  * Changes to Error object formatting
    [#4582](https://github.com/nodejs/node/pull/4582).
  * The `util._extend()` method has been deprecated
    [#4903](https://github.com/nodejs/node/pull/4903)
  * The `util.log()` method has been deprecated
    [#6161](https://github.com/nodejs/node/pull/6161).
* Windows
  * Windows XP and Vista are no longer supported
    [#5167](https://github.com/nodejs/node/pull/5167).
* Zlib
  * Multiple improvements have been made to Zlib processing
    [#5883](https://github.com/nodejs/node/pull/5883) and
    [#5707](https://github.com/nodejs/node/pull/5707).




<a id="7.0.0"></a>
# Version 7.0.0

* Buffer
  * Passing invalid input to Buffer.byteLength will now throw an error [#8946](https://github.com/nodejs/node/pull/8946).
  * Calling Buffer without new is now deprecated and will emit a process warning [#8169](https://github.com/nodejs/node/pull/8169).
  * Passing a negative number to allocUnsafe will now throw an error [#7079](https://github.com/nodejs/node/pull/7079).
* Child Process
  * The fork and execFile methods now have stronger argument validation [#7399](https://github.com/nodejs/node/pull/7399).
* Cluster
  * The worker.suicide method is deprecated and will emit a process warning [#3747](https://github.com/nodejs/node/pull/3747).
* Deps
  * V8 has been updated to 5.4.500.36 [#8317](https://github.com/nodejs/node/pull/8317), [#8852](https://github.com/nodejs/node/pull/8852),
  [#9253](https://github.com/nodejs/node/pull/9253).
  * NODE_MODULE_VERSION has been updated to 51 [#8808](https://github.com/nodejs/node/pull/8808).
* File System
  * A process warning is emitted if a callback is not passed to async file system methods [#7897](https://github.com/nodejs/node/pull/7897).
* Intl
  * Intl.v8BreakIterator constructor has been deprecated and will emit a process warning [#8908](https://github.com/nodejs/node/pull/8908).
* Promises
  * Unhandled Promise rejections have been deprecated and will emit a process warning [#8217](https://github.com/nodejs/node/pull/8217).
* Punycode
  * The `punycode` module has been deprecated [#7941](https://github.com/nodejs/node/pull/7941).
* URL
  * An Experimental WHATWG URL Parser has been introduced [#7448](https://github.com/nodejs/node/pull/7448).




<a id="8.0.0"></a>
# Version 8.0.0

Node.js 8.0.0 is a major new release that includes a significant number of
`semver-major` and `semver-minor` changes. Notable changes are listed below.

Note that the
[LTS lifespan](https://github.com/nodejs/lts) for 8.x will end on December 31st,
2019.

* **Async Hooks**
  * The `async_hooks` module has landed in core
    [[`4a7233c178`](https://github.com/nodejs/node/commit/4a7233c178)]
    [#12892](https://github.com/nodejs/node/pull/12892).

* **Buffer**
  * Using the `--pending-deprecation` flag will cause Node.js to emit a
    deprecation warning when using `new Buffer(num)` or `Buffer(num)`.
    [[`d2d32ea5a2`](https://github.com/nodejs/node/commit/d2d32ea5a2)]
    [#11968](https://github.com/nodejs/node/pull/11968).
  * `new Buffer(num)` and `Buffer(num)` will zero-fill new `Buffer` instances
    [[`7eb1b4658e`](https://github.com/nodejs/node/commit/7eb1b4658e)]
    [#12141](https://github.com/nodejs/node/pull/12141).
  * Many `Buffer` methods now accept `Uint8Array` as input
    [[`beca3244e2`](https://github.com/nodejs/node/commit/beca3244e2)]
    [#10236](https://github.com/nodejs/node/pull/10236).

* **Child Process**
  * Argument and kill signal validations have been improved
    [[`97a77288ce`](https://github.com/nodejs/node/commit/97a77288ce)]
    [#12348](https://github.com/nodejs/node/pull/12348),
    [[`d75fdd96aa`](https://github.com/nodejs/node/commit/d75fdd96aa)]
    [#10423](https://github.com/nodejs/node/pull/10423).
  * Child Process methods accept `Uint8Array` as input
    [[`627ecee9ed`](https://github.com/nodejs/node/commit/627ecee9ed)]
    [#10653](https://github.com/nodejs/node/pull/10653).

* **Console**
  * Error events emitted when using `console` methods are now supressed.
    [[`f18e08d820`](https://github.com/nodejs/node/commit/f18e08d820)]
    [#9744](https://github.com/nodejs/node/pull/9744).

* **Dependencies**
  * The npm client has been updated to 5.0.0
    [[`3c3b36af0f`](https://github.com/nodejs/node/commit/3c3b36af0f)]
    [#12936](https://github.com/nodejs/node/pull/12936).
  * V8 has been updated to 5.8 with forward ABI stability to 6.0
    [[`60d1aac8d2`](https://github.com/nodejs/node/commit/60d1aac8d2)]
    [#12784](https://github.com/nodejs/node/pull/12784).

* **Domains**
  * Native `Promise` instances are now `Domain` aware
    [[`84dabe8373`](https://github.com/nodejs/node/commit/84dabe8373)]
    [#12489](https://github.com/nodejs/node/pull/12489).

* **Errors**
  * We have started assigning static error codes to errors generated by Node.js.
    This has been done through multiple commits and is still a work in
    progress.

* **File System**
  * The utility class `fs.SyncWriteStream` has been deprecated
    [[`7a55e34ef4`](https://github.com/nodejs/node/commit/7a55e34ef4)]
    [#10467](https://github.com/nodejs/node/pull/10467).
  * The deprecated `fs.read()` string interface has been removed
    [[`3c2a9361ff`](https://github.com/nodejs/node/commit/3c2a9361ff)]
    [#9683](https://github.com/nodejs/node/pull/9683).

* **HTTP**
  * Improved support for userland implemented Agents
    [[`90403dd1d0`](https://github.com/nodejs/node/commit/90403dd1d0)]
    [#11567](https://github.com/nodejs/node/pull/11567).
  * Outgoing Cookie headers are concatenated into a single string
    [[`d3480776c7`](https://github.com/nodejs/node/commit/d3480776c7)]
    [#11259](https://github.com/nodejs/node/pull/11259).
  * The `httpResponse.writeHeader()` method has been deprecated
    [[`fb71ba4921`](https://github.com/nodejs/node/commit/fb71ba4921)]
    [#11355](https://github.com/nodejs/node/pull/11355).
  * New methods for accessing HTTP headers have been added to `OutgoingMessage`
    [[`3e6f1032a4`](https://github.com/nodejs/node/commit/3e6f1032a4)]
    [#10805](https://github.com/nodejs/node/pull/10805).

* **Lib**
  * All deprecation messages have been assigned static identifiers
    [[`5de3cf099c`](https://github.com/nodejs/node/commit/5de3cf099c)]
    [#10116](https://github.com/nodejs/node/pull/10116).
  * The legacy `linkedlist` module has been removed
    [[`84a23391f6`](https://github.com/nodejs/node/commit/84a23391f6)]
    [#12113](https://github.com/nodejs/node/pull/12113).

* **N-API**
  * Experimental support for the new N-API API has been added
    [[`56e881d0b0`](https://github.com/nodejs/node/commit/56e881d0b0)]
    [#11975](https://github.com/nodejs/node/pull/11975).

* **Process**
  * Process warning output can be redirected to a file using the
    `--redirect-warnings` command-line argument
    [[`03e89b3ff2`](https://github.com/nodejs/node/commit/03e89b3ff2)]
    [#10116](https://github.com/nodejs/node/pull/10116).
  * Process warnings may now include additional detail
    [[`dd20e68b0f`](https://github.com/nodejs/node/commit/dd20e68b0f)]
    [#12725](https://github.com/nodejs/node/pull/12725).

* **REPL**
  * REPL magic mode has been deprecated
    [[`3f27f02da0`](https://github.com/nodejs/node/commit/3f27f02da0)]
    [#11599](https://github.com/nodejs/node/pull/11599).

* **Src**
  * `NODE_MODULE_VERSION` has been updated to 57
    [[`ec7cbaf266`](https://github.com/nodejs/node/commit/ec7cbaf266)]
    [#12995](https://github.com/nodejs/node/pull/12995).
  * Add `--pending-deprecation` command-line argument and
    `NODE_PENDING_DEPRECATION` environment variable
    [[`a16b570f8c`](https://github.com/nodejs/node/commit/a16b570f8c)]
    [#11968](https://github.com/nodejs/node/pull/11968).
  * The `--debug` command-line argument has been deprecated. Note that
    using `--debug` will enable the *new* Inspector-based debug protocol
    as the legacy Debugger protocol previously used by Node.js has been
    removed. [[`010f864426`](https://github.com/nodejs/node/commit/010f864426)]
    [#12949](https://github.com/nodejs/node/pull/12949).
  * Throw when the `-c` and `-e` command-line arguments are used at the same
    time [[`a5f91ab230`](https://github.com/nodejs/node/commit/a5f91ab230)]
    [#11689](https://github.com/nodejs/node/pull/11689).
  * Throw when the `--use-bundled-ca` and `--use-openssl-ca` command-line
    arguments are used at the same time.
    [[`8a7db9d4b5`](https://github.com/nodejs/node/commit/8a7db9d4b5)]
    [#12087](https://github.com/nodejs/node/pull/12087).

* **Stream**
  * `Stream` now supports `destroy()` and `_destroy()` APIs
    [[`b6e1d22fa6`](https://github.com/nodejs/node/commit/b6e1d22fa6)]
    [#12925](https://github.com/nodejs/node/pull/12925).
  * `Stream` now supports the `_final()` API
    [[`07c7f198db`](https://github.com/nodejs/node/commit/07c7f198db)]
    [#12828](https://github.com/nodejs/node/pull/12828).

* **TLS**
  * The `rejectUnauthorized` option now defaults to `true`
    [[`348cc80a3c`](https://github.com/nodejs/node/commit/348cc80a3c)]
    [#5923](https://github.com/nodejs/node/pull/5923).
  * The `tls.createSecurePair()` API now emits a runtime deprecation
    [[`a2ae08999b`](https://github.com/nodejs/node/commit/a2ae08999b)]
    [#11349](https://github.com/nodejs/node/pull/11349).
  * A runtime deprecation will now be emitted when `dhparam` is less than
    2048 bits [[`d523eb9c40`](https://github.com/nodejs/node/commit/d523eb9c40)]
    [#11447](https://github.com/nodejs/node/pull/11447).

* **URL**
  * The WHATWG URL implementation is now a fully-supported Node.js API
    [[`d080ead0f9`](https://github.com/nodejs/node/commit/d080ead0f9)]
    [#12710](https://github.com/nodejs/node/pull/12710).

* **Util**
  * `Symbol` keys are now displayed by default when using `util.inspect()`
    [[`5bfd13b81e`](https://github.com/nodejs/node/commit/5bfd13b81e)]
    [#9726](https://github.com/nodejs/node/pull/9726).
  * `toJSON` errors will be thrown when formatting `%j`
    [[`455e6f1dd8`](https://github.com/nodejs/node/commit/455e6f1dd8)]
    [#11708](https://github.com/nodejs/node/pull/11708).
  * Convert `inspect.styles` and `inspect.colors` to prototype-less objects
    [[`aab0d202f8`](https://github.com/nodejs/node/commit/aab0d202f8)]
    [#11624](https://github.com/nodejs/node/pull/11624).
  * The new `util.promisify()` API has been added
    [[`99da8e8e02`](https://github.com/nodejs/node/commit/99da8e8e02)]
    [#12442](https://github.com/nodejs/node/pull/12442).

* **Zlib**
  * Support `Uint8Array` in Zlib convenience methods
    [[`91383e47fd`](https://github.com/nodejs/node/commit/91383e47fd)]
    [#12001](https://github.com/nodejs/node/pull/12001).
  * Zlib errors now use `RangeError` and `TypeError` consistently
    [[`b514bd231e`](https://github.com/nodejs/node/commit/b514bd231e)]
    [#11391](https://github.com/nodejs/node/pull/11391).



<a id="9.0.0"></a>
# Version 9.0.0

* **Async hooks**
  * Older experimental APIs have been removed. [[`d731369b1d`](https://github.com/nodejs/node/commit/d731369b1d)] [#14414](https://github.com/nodejs/node/pull/14414)

* **Errors**
  * Improvements have been made to `buffer` module error messages. [[`9e0f771224`](https://github.com/nodejs/node/commit/9e0f771224)] [#14975](https://github.com/nodejs/node/pull/14975)
  * The assignment of static error codes to Node.js error continues:
    * `buffer`: [[`e79a61cf80`](https://github.com/nodejs/node/commit/e79a61cf80)] [#16352](https://github.com/nodejs/node/pull/16352), [[`dbfe8c4ea2`](https://github.com/nodejs/node/commit/dbfe8c4ea2)] [#13976](https://github.com/nodejs/node/pull/13976)
    * `child_process`: [[`fe730d34ce`](https://github.com/nodejs/node/commit/fe730d34ce)] [#14009](https://github.com/nodejs/node/pull/14009)
    * `console`: [[`0ecdf29340`](https://github.com/nodejs/node/commit/0ecdf29340)] [#11340](https://github.com/nodejs/node/pull/11340)
    * `crypto`: [[`ee76f3153b`](https://github.com/nodejs/node/commit/ee76f3153b)] [#16428](https://github.com/nodejs/node/pull/16428), [[`df8c6c3651`](https://github.com/nodejs/node/commit/df8c6c3651)] [#16453](https://github.com/nodejs/node/pull/16453), [[`0a03e350fb`](https://github.com/nodejs/node/commit/0a03e350fb)] [#16454](https://github.com/nodejs/node/pull/16454), [[`eeada6ca63`](https://github.com/nodejs/node/commit/eeada6ca63)] [#16448](https://github.com/nodejs/node/pull/16448), [[`a78327f48b`](https://github.com/nodejs/node/commit/a78327f48b)] [#16429](https://github.com/nodejs/node/pull/16429), [[`b8bc652869`](https://github.com/nodejs/node/commit/b8bc652869)] [#15757](https://github.com/nodejs/node/pull/15757), [[`7124b466d9`](https://github.com/nodejs/node/commit/7124b466d9)] [#15746](https://github.com/nodejs/node/pull/15746), [[`3ddc88b5c2`](https://github.com/nodejs/node/commit/3ddc88b5c2)] [#15756](https://github.com/nodejs/node/pull/15756)
    * `dns`: [[`9cb390d899`](https://github.com/nodejs/node/commit/9cb390d899)] [#14212](https://github.com/nodejs/node/pull/14212)
    * `events`: [[`e5ad5456a2`](https://github.com/nodejs/node/commit/e5ad5456a2)] [#15623](https://github.com/nodejs/node/pull/15623)
    * `fs`: [[`219932a9f7`](https://github.com/nodejs/node/commit/219932a9f7)] [#15043](https://github.com/nodejs/node/pull/15043), [[`b61cab2234`](https://github.com/nodejs/node/commit/b61cab2234)] [#11317](https://github.com/nodejs/node/pull/11317)
    * `http`: [[`11a2ca29ba`](https://github.com/nodejs/node/commit/11a2ca29ba)] [#14735](https://github.com/nodejs/node/pull/14735), [[`a9f798ebcc`](https://github.com/nodejs/node/commit/a9f798ebcc)] [#13301](https://github.com/nodejs/node/pull/13301), [[`bdfbce9241`](https://github.com/nodejs/node/commit/bdfbce9241)] [#14423](https://github.com/nodejs/node/pull/14423), [[`4843c2f415`](https://github.com/nodejs/node/commit/4843c2f415)] [#15603](https://github.com/nodejs/node/pull/15603)
    * `inspector`: [[`4cf56ad6f2`](https://github.com/nodejs/node/commit/4cf56ad6f2)] [#15619](https://github.com/nodejs/node/pull/15619)
    * `net`: [[`a03d8cee1f`](https://github.com/nodejs/node/commit/a03d8cee1f)] [#11356](https://github.com/nodejs/node/pull/11356), [[`7f55349079`](https://github.com/nodejs/node/commit/7f55349079)] [#14782](https://github.com/nodejs/node/pull/14782)
    * `path`: [[`dcfbbacba8`](https://github.com/nodejs/node/commit/dcfbbacba8)] [#11319](https://github.com/nodejs/node/pull/11319)
    * `process`: [[`a0f7284346`](https://github.com/nodejs/node/commit/a0f7284346)] [#13739](https://github.com/nodejs/node/pull/13739), [[`062071a9c3`](https://github.com/nodejs/node/commit/062071a9c3)] [#13285](https://github.com/nodejs/node/pull/13285), [[`3129b2c035`](https://github.com/nodejs/node/commit/3129b2c035)] [#13982](https://github.com/nodejs/node/pull/13982)
    * `querystring`: [[`9788e96836`](https://github.com/nodejs/node/commit/9788e96836)] [#15565](https://github.com/nodejs/node/pull/15565)
    * `readline`: [[`7f3f72c19b`](https://github.com/nodejs/node/commit/7f3f72c19b)] [#11390](https://github.com/nodejs/node/pull/11390)
    * `repl`: [[`aff8d358fa`](https://github.com/nodejs/node/commit/aff8d358fa)] [#11347](https://github.com/nodejs/node/pull/11347), [[`28227963fa`](https://github.com/nodejs/node/commit/28227963fa)] [#13299](https://github.com/nodejs/node/pull/13299)
    * `streams`: [[`d50a802feb`](https://github.com/nodejs/node/commit/d50a802feb)] [#13310](https://github.com/nodejs/node/pull/13310), [[`d2913384aa`](https://github.com/nodejs/node/commit/d2913384aa)] [#13291](https://github.com/nodejs/node/pull/13291), [[`6e86a6651c`](https://github.com/nodejs/node/commit/6e86a6651c)] [#16589](https://github.com/nodejs/node/pull/16589), [[`88fb359c57`](https://github.com/nodejs/node/commit/88fb359c57)] [#15042](https://github.com/nodejs/node/pull/15042), [[`db7d1339c3`](https://github.com/nodejs/node/commit/db7d1339c3)] [#15665](https://github.com/nodejs/node/pull/15665)
    * `string_decoder`: [[`eb4940e2d2`](https://github.com/nodejs/node/commit/eb4940e2d2)] [#14682](https://github.com/nodejs/node/pull/14682)
    * `timers`: [[`4d893e093a`](https://github.com/nodejs/node/commit/4d893e093a)] [#14659](https://github.com/nodejs/node/pull/14659)
    * `tls`: [[`f67aa566a6`](https://github.com/nodejs/node/commit/f67aa566a6)] [#13476](https://github.com/nodejs/node/pull/13476), [[`3ccfeb483d`](https://github.com/nodejs/node/commit/3ccfeb483d)] [#13994](https://github.com/nodejs/node/pull/13994)
    * `url`: [[`473f0eff29`](https://github.com/nodejs/node/commit/473f0eff29)] [#13963](https://github.com/nodejs/node/pull/13963)
    * `util`: [[`de4a749788`](https://github.com/nodejs/node/commit/de4a749788)] [#11301](https://github.com/nodejs/node/pull/11301), [[`1609899142`](https://github.com/nodejs/node/commit/1609899142)] [#13293](https://github.com/nodejs/node/pull/13293)
    * `v8`: [[`ef238fb485`](https://github.com/nodejs/node/commit/ef238fb485)] [#16535](https://github.com/nodejs/node/pull/16535)
    * `zlib`: [[`896eaf6820`](https://github.com/nodejs/node/commit/896eaf6820)] [#16540](https://github.com/nodejs/node/pull/16540), [[`74891412f1`](https://github.com/nodejs/node/commit/74891412f1)] [#15618](https://github.com/nodejs/node/pull/15618)

* **Child Processes**
  * Errors are emitted on process nextTick. [[`f2b01cba7b`](https://github.com/nodejs/node/commit/f2b01cba7b)] [#4670](https://github.com/nodejs/node/pull/4670)

* **Domains**
  * The long-deprecated `.dispose()` method has been removed [[`602fd36d95`](https://github.com/nodejs/node/commit/602fd36d95)] [#15412](https://github.com/nodejs/node/pull/15412)

* **fs**
  * The `fs.ReadStream` and `fs.WriteStream` classes now use `destroy()`. [[`e5c290bed9`](https://github.com/nodejs/node/commit/e5c290bed9)] [#15407](https://github.com/nodejs/node/pull/15407)
  * `fs` module callbacks are now invoked with an undefined context. [[`2249234fee`](https://github.com/nodejs/node/commit/2249234fee)] [#14645](https://github.com/nodejs/node/pull/14645)

* **HTTP/1**
  * A 400 Bad Request response will now be sent when parsing fails. [[`f2f391e575`](https://github.com/nodejs/node/commit/f2f391e575)] [#15324](https://github.com/nodejs/node/pull/15324)
  * Socket timeout will be set when the socket connects. [[`10be20a0e8`](https://github.com/nodejs/node/commit/10be20a0e8)] [#8895](https://github.com/nodejs/node/pull/8895)
  * A bug causing the request `'error'` event to fire twice was fixed. [[`620ba41694`](https://github.com/nodejs/node/commit/620ba41694)] [#14659](https://github.com/nodejs/node/pull/14659)
  * HTTP clients may now use generic `Duplex` streams in addition to `net.Socket`. [[`3e25e4d00f`](https://github.com/nodejs/node/commit/3e25e4d00f)] [#16267](https://github.com/nodejs/node/pull/16267)

* **Intl**
  * The deprecated `Intl.v8BreakIterator` has been removed. [[`668ad44922`](https://github.com/nodejs/node/commit/668ad44922)] [#15238](https://github.com/nodejs/node/pull/15238)

* **OS**
  * The `os.EOL` property is now read-only [[`f6caeb9526`](https://github.com/nodejs/node/commit/f6caeb9526)] [#14622](https://github.com/nodejs/node/pull/14622)

* **Timers**
  * `setTimeout()` will emit a warning if the timeout is larger that the maximum 32-bit unsigned integer. [[`ce3586da31`](https://github.com/nodejs/node/commit/ce3586da31)] [#15627](https://github.com/nodejs/node/pull/15627)




<a id="10.0.0"></a>
# Version 10.0.0

* Assert
  * Calling `assert.fail()` with more than one argument is deprecated. [[`70dcacd710`](https://github.com/nodejs/node/commit/70dcacd710)]
  * Calling `assert.ok()` with no arguments will now throw. [[`3cd7977a42`](https://github.com/nodejs/node/commit/3cd7977a42)]
  * Calling `assert.ifError()` will now throw with any argument other than `undefined` or `null`. Previously the method would throw with any truthy value. [[`e65a6e81ef`](https://github.com/nodejs/node/commit/e65a6e81ef)]
  * The `assert.rejects()` and `assert.doesNotReject()` methods have been added for working with async functions. [[`599337f43e`](https://github.com/nodejs/node/commit/599337f43e)]
* Async_hooks
  * Older experimental async_hooks APIs have been removed. [[`1cc6b993b9`](https://github.com/nodejs/node/commit/1cc6b993b9)]
* Buffer
  * Uses of `new Buffer()` and `Buffer()` outside of the `node_modules` directory will now emit a runtime deprecation warning. [[`9d4ab90117`](https://github.com/nodejs/node/commit/9d4ab90117)]
  * `Buffer.isEncoding()` now returns `undefined` for falsy values, including an empty string. [[`452eed956e`](https://github.com/nodejs/node/commit/452eed956e)]
  * `Buffer.fill()` will throw if an attempt is made to fill with an empty `Buffer`. [[`1e802539b2`](https://github.com/nodejs/node/commit/1e802539b2)]
* Child Process
  * Undefined properties of env are ignored. [[`38ee25e2e2`](https://github.com/nodejs/node/commit/38ee25e2e2)], [[`85739b6c5b`](https://github.com/nodejs/node/commit/85739b6c5b)]
* Console
  * The `console.table()` method has been added. [[`97ace04492`](https://github.com/nodejs/node/commit/97ace04492)]
* Crypto
  * The `crypto.createCipher()` and `crypto.createDecipher()` methods have been deprecated. Please use `crypto.createCipheriv()` and `crypto.createDecipheriv()` instead. [[`81f88e30dd`](https://github.com/nodejs/node/commit/81f88e30dd)]
  * The `decipher.finaltol()` method has been deprecated. [[`19f3927d92`](https://github.com/nodejs/node/commit/19f3927d92)]
  * The `crypto.DEFAULT_ENCODING` property has been deprecated. [[`6035beea93`](https://github.com/nodejs/node/commit/6035beea93)]
  * The `ECDH.convertKey()` method has been added. [[`f2e02883e7`](https://github.com/nodejs/node/commit/f2e02883e7)]
  * The `crypto.fips` property has been deprecated. [[`6e7992e8b8`](https://github.com/nodejs/node/commit/6e7992e8b8)]
* Dependencies
  * V8 has been updated to 6.6. [[`9daebb48d6`](https://github.com/nodejs/node/commit/9daebb48d6)]
  * OpenSSL has been updated to 1.1.0h. [[`66cb29e646`](https://github.com/nodejs/node/commit/66cb29e646)]
* EventEmitter
  * The `EventEmitter.prototype.off()` method has been added as an alias for `EventEmitter.prototype.removeListener()`. [[`3bb6f07d52`](https://github.com/nodejs/node/commit/3bb6f07d52)]
* File System
  * The `fs/promises` API provides experimental promisified versions of the `fs` functions. [[`329fc78e49`](https://github.com/nodejs/node/commit/329fc78e49)]
  * Invalid path errors are now thrown synchronously. [[`d8f73385e2`](https://github.com/nodejs/node/commit/d8f73385e2)]
  * The `fs.readFile()` method now partitions reads to avoid thread pool exhaustion. [[`67a4ce1c6e`](https://github.com/nodejs/node/commit/67a4ce1c6e)]
* HTTP
  * Processing of HTTP Status codes `100`, `102-199` has been improved. [[`baf8495078`](https://github.com/nodejs/node/commit/baf8495078)]
  * Multi-byte characters in URL paths are now forbidden. [[`b961d9fd83`](https://github.com/nodejs/node/commit/b961d9fd83)]
* N-API
  * The n-api is no longer experimental. [[`cd7d7b15c1`](https://github.com/nodejs/node/commit/cd7d7b15c1)]
* Net
  * The `'close'` event will be emitted after `'end'`. [[`9b7a6914a7`](https://github.com/nodejs/node/commit/9b7a6914a7)]
* Perf_hooks
  * The `PerformanceObserver` class is now an `AsyncResource` and can be monitored using `async_hooks`. [[`009e41826f`](https://github.com/nodejs/node/commit/009e41826f)]
  * Trace events are now emitted for performance events. [[`9e509b622b`](https://github.com/nodejs/node/commit/9e509b622b)]
  * The `performance` API has been simplified. [[`2ec6995555`](https://github.com/nodejs/node/commit/2ec6995555)]
  * Performance milestone marks will be emitted as trace events. [[`96cb4fb795`](https://github.com/nodejs/node/commit/96cb4fb795)]
* Process
  * Using non-string values for `process.env` is deprecated. [[`5826fe4e79`](https://github.com/nodejs/node/commit/5826fe4e79)]
  * The `process.assert()` method is deprecated. [[`703e37cf3f`](https://github.com/nodejs/node/commit/703e37cf3f)]
* REPL
  * REPL now experimentally supports top-level await when using the `--experimental-repl-await` flag. [[`eeab7bc068`](https://github.com/nodejs/node/commit/eeab7bc068)]
  * The previously deprecated "magic mode" has been removed. [[`4893f70d12`](https://github.com/nodejs/node/commit/4893f70d12)]
  * The previously deprecated `NODE_REPL_HISTORY_FILE` environment variable has been removed. [[`60c9ad7979`](https://github.com/nodejs/node/commit/60c9ad7979)]
  * Proxy objects are shown as Proxy objects when inspected. [[`90a43906ab`](https://github.com/nodejs/node/commit/90a43906ab)]
* Streams
  * The `'readable'` event is now always deferred with nextTick. [[`1e0f3315c7`](https://github.com/nodejs/node/commit/1e0f3315c7)]
  * A new `pipeline()` method has been provided for building end-to-data stream pipelines. [[`a5cf3feaf1`](https://github.com/nodejs/node/commit/a5cf3feaf1)]
  * Experimental support for async for-await has been added to `stream.Readable`. [[`61b4d60c5d`](https://github.com/nodejs/node/commit/61b4d60c5d)]
* Timers
  * The `enroll()` and `unenroll()` methods have been deprecated. [[`68783ae0b8`](https://github.com/nodejs/node/commit/68783ae0b8)]
* TLS
  * The `tls.convertNPNProtocols()` method has been deprecated. [[`9204a0db6e`](https://github.com/nodejs/node/commit/9204a0db6e)]
  * Support for NPN (next protocol negotiation) has been dropped. [[`5bfbe5ceae`](https://github.com/nodejs/node/commit/5bfbe5ceae)]
  * The `ecdhCurve` default is now `'auto'`. [[`af78840b19`](https://github.com/nodejs/node/commit/af78840b19)]
* Trace Events
  * A new `trace_events` top-level module allows trace event categories to be enabled/disabled at runtime. [[`da5d818a54`](https://github.com/nodejs/node/commit/da5d818a54)]
* URL
  * The WHATWG URL API is now a global. [[`312414662b`](https://github.com/nodejs/node/commit/312414662b)]
* Util
  * `util.types.is[…]` type checks have been added. [[`b20af8088a`](https://github.com/nodejs/node/commit/b20af8088a)]
  * Support for bigint formatting has been added to `util.inspect()`. [[`39dc947409`](https://github.com/nodejs/node/commit/39dc947409)]

#### Deprecations:

The following APIs have been deprecated in Node.js 10.0.0

* Passing more than one argument to `assert.fail()` will emit a runtime deprecation warning. [[`70dcacd710`](https://github.com/nodejs/node/commit/70dcacd710)]
* Previously deprecated legacy async_hooks APIs have reached end-of-life and have been removed. [[`1cc6b993b9`](https://github.com/nodejs/node/commit/1cc6b993b9)]
* Using `require()` to access several of Node.js' own internal dependencies will emit a runtime deprecation. [[`0e10717e43`](https://github.com/nodejs/node/commit/0e10717e43)]
* The `crypto.createCipher()` and `crypto.createDecipher()` methods have been deprecated in documentation.[[`81f88e30dd`](https://github.com/nodejs/node/commit/81f88e30dd)]
* Using the `Decipher.finaltol()` method will emit a runtime deprecation warning. [[`19f3927d92`](https://github.com/nodejs/node/commit/19f3927d92)]
* Using the `crypto.DEFAULT_ENCODING` property will emit a runtime deprecation warning. [[`6035beea93`](https://github.com/nodejs/node/commit/6035beea93)]
* Use by native addons of the `MakeCallback()` variant that passes a `Domain` will emit a runtime deprecation warning. [[`14bc3e22f3`](https://github.com/nodejs/node/commit/14bc3e22f3)], [[`efb32592e1`](https://github.com/nodejs/node/commit/efb32592e1)]
* Previously deprecated internal getters/setters on `net.Server` has reached end-of-life and have been removed. [[`3701b02309`](https://github.com/nodejs/node/commit/3701b02309)]
* Use of non-string values for `process.env` has been deprecated in documentation. [[`5826fe4e79`](https://github.com/nodejs/node/commit/5826fe4e79)]
* Use of `process.assert()` will emit a runtime deprecation warning. [[`703e37cf3f`](https://github.com/nodejs/node/commit/703e37cf3f)]
* Previously deprecated `NODE_REPL_HISTORY_FILE` environment variable has reached end-of-life and has been removed. [[`60c9ad7979`](https://github.com/nodejs/node/commit/60c9ad7979)]
* Use of the `timers.enroll()` and `timers.unenroll()` methods will emit a runtime deprecation warning. [[`68783ae0b8`](https://github.com/nodejs/node/commit/68783ae0b8)]
* Use of the `tls.convertNPNProtocols()` method will emit a runtime deprecation warning. Support for NPN has been removed from Node.js. [[`9204a0db6e`](https://github.com/nodejs/node/commit/9204a0db6e)]
* The `crypto.fips` property has been deprecated in documentation. [[`6e7992e8b8`](https://github.com/nodejs/node/commit/6e7992e8b8)]
