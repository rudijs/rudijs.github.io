---
layout: post
title:  "Couchbase Node.js SDK Callbacks to Promises"
date:   2015-08-19 12:00:00
categories: couchbase node.js sdk callbacks promises
tags: Couchbase Node.js SDK Callback Promises
---

## Overview

Where possible I prefer to use Promises rather than Callbacks when coding in Node.js.

The [Couchbase Node.js SDK 2.0](http://docs.couchbase.com/developer/node-2.0/introduction.html) documentation examples use the callback style of coding.

Here is an approach to "roll your own Couchbase Node.js Promises".

The following Node.js code uses [Q](https://github.com/kriskowal/q) to convert those code examples to Promises.

These examples will dependency inject into the functions rather than reference global variables.
 
The final example will use ramda.js and curry the functions.
 
### Setup
 
```
'use strict';

var assert = require('assert'),
  couchbase = require('couchbase'),
  Q = require('q'),
  R = require('ramda');

var cluster = new couchbase.Cluster('couchbase://localhost');
var bucket = cluster.openBucket('default');
```

### Code - Callbacks to Promises

```
// myBucket.insert('document_name', {some:'value'}, function(err, res) {
// console.log('Success!');
// });

function bucketInsert(bucket, documentName, documentValue) {

  assert.equal(typeof bucket, 'object', 'argument bucket must be an object');
  assert.equal(typeof documentName, 'string', 'argument documentName must be a string');
  assert.equal(typeof documentValue, 'object', 'argument documentValue must be an object');

  return Q.ninvoke(bucket, 'insert', documentName, documentValue).then(function (res) {

    if (!res.cas) {
      throw new Error('Bucket Insert Failed.');
    }

    return res;

  });

}

// myBucket.get('document_name', function(err, res) {
// console.log('Value: ', res.value);
// });

function bucketGet(bucket, documentName) {

  assert.equal(typeof bucket, 'object', 'argument bucket must be an object');
  assert.equal(typeof documentName, 'string', 'argument documentName must be a string');

  return Q.ninvoke(bucket, 'get', documentName).then(function (res) {

    if (!res.cas || !res.value) {
      throw new Error('Bucket Get Failed.');
    }

    return res.value;

  });

}
```

### Usage

```
/* Insert */
bucketInsert(bucket, 'document_name', {some: 'value'})
  .then(function (res) {
    console.log('Success!');
  })
  .catch(function (err) {
    console.log(err);
  });

// outputs:
// Success!


/* Get */
bucketGet(bucket, 'document_name')
  .then(function (res) {
    console.log(res);
  })
  .catch(function (err) {
    console.log(err);
  });
  
// outputs
// { some: 'value' }


/* Chained Insert and Get */
bucketInsert(bucket, 'document_name2', {some: 'value2'})
  .then(function () {
    return bucketGet(bucket, 'document_name2')
  })
  .then(function (res) {
    console.log(res);
  })
  .catch(function (err) {
    console.log(err);
  });

// outputs
// { some: 'value2' }
 

/* Curry and Chain Insert and Get */
var curriedInsert = R.curry(bucketInsert),
  insert = curriedInsert(bucket);

var curriedGet = R.curry(bucketGet),
  get = curriedGet(bucket);

insert('document_name3', {some: 'value3'})
  .then(function () {
    return get('document_name3')
  })
  .then(function (res) {
    console.log(res);
  })
  .catch(function (err) {
    console.log(err);
  });
  
// outputs
// { some: 'value3' }  

```

### Summary

This approach works well for me and I'm quite pleased with it.

The next post will demonstrate Unit testing this code with Mocha, Sinon.js and Chai.js.
 
I hope this helps, comments and feedback are very much welcomed.

If I've overlooked anything, if you can see room for improvement or if any errors please do let me know.

Thanks!
