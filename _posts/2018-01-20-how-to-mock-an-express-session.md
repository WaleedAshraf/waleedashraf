---
layout: post
title: How to mock an Express session?
excerpt: "Mock session object in Nodejs integration test-cases."
modified: 2018-04-16T14:53:25+05:00
author: waleed_ashraf
image:
  path: ../images/cover_arch.jpg
  feature: cover_arch.jpg
  credit: thomashawk
  creditlink: http://thomashawk.com/
comments: true
share: true
---

_[This article was published in Nodejs-Collection on medium.](https://medium.com/the-node-js-collection/how-to-mock-an-express-session-fe62baf5a611)_

Are you using an Express app and need to add integration (end-to-end) testing? Here’s what I discovered when I was trying to do this for a project that I worked on. This article covers the background around the challenge that I faced when trying to add an integration test for an Express App, and a simple way to solve this.

Hopefully, this helps those that might be facing the same problem.

### Background

I was tasked with working on a project where I was uploading images to amazon s3 in bulk (albums) and then needed to apply some processing like resizing/compression/editing on them.

It is an Express app with middleware for authorization, authentication, setting context and is working fine. Recently, however, I started adding test cases for different routes using mocha. I’m using [cookie-session](https://www.npmjs.com/package/cookie-session) for session handling in the app, which means [cookie-session](https://www.npmjs.com/package/cookie-session) is responsible for encrypting/setting/decrypting all session related data.

I know that many folks use [sinon for unit testing](http://sinonjs.org/). If you are unfamiliar with it, it’s used to mock methods with JavaScript testing frameworks (like [mocha](https://mochajs.org/)), and is one of the most popular packages used for JavaScript testing. But what if you need it for Express Session? Is there a way to mock session objects in tests for [Express.js](https://expressjs.com/)?

![](https://cdn-images-1.medium.com/max/7104/1*gB4zVnI32ufFnMmGwL-yqA.jpeg)

### The Challenge

Here’s where the challenge initial started for me. I have two routes: one is to upload the photo to s3 (create) and the second one (submit) to apply further processing on uploaded images. For one route, I needed to mock `req.session` object. I needed to set its value before calling the other route. First route (create) set `req.session` value and redirect to the second one (submit).

**1) api/image/create**
```javascript
exports.create = function(req, res) {
  // upload image to s3
  var photos = PhotosTable.create(req.files.names); // add entries in database
  req.session.count = photos.length; // set count in req.session
  res.redirect(`api/image/submit`);
}
```

**2) api/image/submit (need to add a test case for this one)**

```javascript
exports.submit = function(req, res) {
  if (req.session.count && req.session.count > 0) { // get count from req.session
    // perform further tasks
    res.sendStatus(200);
  } else {
    res.sendStatus(400);
  }
}
```

I just want to test second route (api/image/submit). The issue is that api/image/submit route checks for `req.session.count` and than perform further tasks. If the count is not set, it sends 400 in response. With mocha you can only check response from one route, you can’t wait for the result from redirect call.

### **Fixes that Seem Smart, but Pose More Challenges**

The easy solution to fix this issue is to send a req-param from the test case, and check it in method if that param is present, then skip checking for `req.session.count`.

However, this solution isn’t the best because:

* It can’t be reused as it’s not a generic solution.

* Adding if/else for an extra parameter in controller logic will require more checks to verify that controller logic is not affected.

I could also easily get rid of the second route and add all logic in one. But should I? As far as I know,
> one should never change implementation for the sake of test cases.

This problem seems so common but I couldn’t find any solution on the internet (or at least StackOverflow). My first guess was that there would be some npm-package to handle this kind of situation. As mocha, [cookie-session](https://www.npmjs.com/package/cookie-session) are the most used modules with Express.

Alas! There isn’t any.

### How I Eventually Solved This Issue

I am using cookie-session with Express to maintain sessions. cookie-session maintains the session by keeping signed cookies. It uses cookies with Keygrip and Crypto to set and validate session keys.

It requires a ***name*** and a ***key*** for setting session cookies.
***name*** is used as cookie name.
***key*** is used to sign cookie value using Keygrip.

I thought that I could fix this by setting `req.session.count` value in the test case, before calling the route. Here’s the code:

```javascript
// name = "my-session" ->  base64 value of required object (cookie)
// name.sig = "my-session.sig" -> signed value of cookie using Keygrip

let cookie = Buffer.from(JSON.stringify({"count":2})).toString('base64'); // base64 converted value of cookie

let kg = Keygrip(['testKey']) // same key as I'm using in my app
let hash = kg.sign('my-session=' + cookie);

await request(app).get('api/image/submit').set("Accept", "text/html")
    .set('cookie', ['my-session=' + cookie + '; ' + 'my-session.sig=' + hash + ';'])
    .expect(200);
```

That’s it. By setting this cookie in the request, I was able to set `req.session.count` to my desired value.

```javascript
exports.submit = function(req, res) {
  if(req.session.count && req.session.count > 0) { // true
    //do something
    res.sendStatus(200);
  }
}
```

This is just one use case for mocking session in Express. Most of the time tests require customized values for the assertion, but it can be used wherever you need to set `req.session` with any specific value.

I have also created a package at npm: ***mock-session*** to make it easier for you. Please let me know if it works!
[https://www.npmjs.com/package/mock-session](https://www.npmjs.com/package/mock-session)
