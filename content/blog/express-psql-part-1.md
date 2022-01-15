---
title: Building a REST-API with Expressjs, Tabel and Postgres | Part-1
date: "2016-04-02"
categories:
  - code
tags:
  - express
  - es6
  - javascript
  - tabel
  - postgres
  - REST API
---

initializing a nodejs server application and sample test with chai.

<!-- more  -->


**Initialize**

`npm init`

-----------------------------



**Server**

`mkdir server`

now add index.js in server

*/server/index.js*

```js
'use strict';

console.log('It is working !!');

```

run server:

`node server`

It should print 

```bash
It is working !!
```

--------------------------------



**Test** 

We will be using [mocha](https://mochajs.org/) and [chai](http://www.chaijs.com/) for testing.

```bash
npm install --save-dev mocha chai
```

Create a directory `test` and write a sample test.

```bash
mkdir test
touch test/sample.js
```

*/test/sample.js*

```js
'use strict';

const chai =  require('chai');
chai.should();

var foo = 'bar';
var beverages = { tea: ['chai', 'matcha', 'oolong'] };

describe('sample Test', () => {
  it('It should fail', (done) => {
    foo.should.be.a('string');
    foo.should.equal('bar');
    foo.should.have.lengthOf(4);
    beverages.should.have.property('tea').with.lengthOf(3);
    done();
  });
});

```

run test 

`npm test`

test should fail with following message:

```bash

> mocha



  sample Test
    1) It should fail


  0 passing (12ms)
  1 failing

  1) sample Test
       It should fail:

      AssertionError: expected 'bar' to have a length of 4 but got 3
      + expected - actual

      -3
      +4

      at Context.it (test/sample.js:13:21)



npm ERR! Test failed.  See above for more details.
```



Now we have written a nodejs server and a sample test , we will continue by adding a database to our app in next part.



`CODE:` https://github.com/satyamyadav/postgres-express/tree/v0.1.0-0



--------------------------

`Thank You !!`