---
title: Building a REST-API with Expressjs, Tabel and Postgres | Part-3
date: "2016-06-02"
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

`TDD;` Test Driven Development of an Expressjs app with mocha and chai.  

<!-- more -->

**Test**

Lets write a quick test :

`npm install  chai-http --save-dev`

create a new file *routes.index.test.js* in test directory

`touch test/routes.index.test.js`

```js
'use strict';

process.env.NODE_ENV = 'test';

const chai = require('chai');
const should = chai.should();
const chaiHttp = require('chai-http');
chai.use(chaiHttp);

const server = require('../server');

describe('routes : index', () => {
  describe('GET /', () => {
    it('should return json', (done) => {
      chai.request(server)
      .get('/')
      .end((err, res) => {
        should.not.exist(err);
        res.status.should.eql(200);
        res.type.should.eql('application/json');
        res.body.status.should.eql('success');
        res.body.message.should.eql('hello, world!');
        done();
      }); 
    });
  });
});
```

  run test `npm test` it should return 

```bash
 1) routes : index
       GET /
         should return json:
     TypeError: app.address is not a function
```



As we have not defined any route yet and no server is running , lets define an express app in *server/index.js*

`npm install express --save`

```js
'use strict';

const config =  require('config');
const PORT = config.port || 4000;
const express =  require('express');

var app = express();

app.use('/', (req, res) => {
  res.json({
    status: 'success',
    message: 'hello, world!'
  });
});

const server = app.listen(PORT, () => {
  console.log(`Server listening on port: ${PORT}`);
});

module.exports = server;
```

run `npm start ` and nevigate to `http://localhost:4000/` 

response should be :

```js
{
    status	"success"
	message	"hello, world!"
}
```

Now kill the server and run `npm test`  , it should pass now:

```bash
routes : index
    GET /
      ✓ should return json

  sample Test
    1) It should fail


  1 passing (33ms)
  1 failing

  1) sample Test
       It should fail:

      AssertionError: expected 'bar' to have a length of 4 but got 3
      + expected - actual

      -3
      +4

      at Context.it (test/sample.js:13:21)
```



Create a new folder called “routes” within “server”, and then add an *index.js*  file to it:

*/server/routes/index.js*

```js
'use strict';

const express = require("express");
const router = express.Router();

module.exports = function () {

  router.route('/')
  .get((req, res) => {
      res.json({
      status: 'success',
      message: 'hello, world!'
    });
  });

  return router;
};
```

Update the *server/index.js* to use routes

```js
'use strict';

const config =  require('config');
const PORT = config.port || 4000;
const express =  require('express');
const routes = require('./routes');

var app = express();

app.use('/', routes());

const server = app.listen(PORT, () => {
  console.log(`Server listening on port: ${PORT}`);
});

module.exports = server;
```

So we have moved our route `/`  to routes, ensure that tests still pass by running `npm test`.





------



**Routes**



We’ll take a test-first approach in writing our routes so add an module template first for posts routes.

/posts

create a file posts.js inside routes with following route definition and export it :

```js
'use strict';

const express = require('express');
const router = express.Router();

module.exports = function(app, db) {
  return router;
};
```

Update *routes/index.js*

```js
'use strict';

const express = require("express");
const router = express.Router();
const posts = require('./posts');

module.exports = function (app, db) {

  router.use('/api/posts', posts(app, db));
  
  router.route('*').all(function(req, res) {
    res.json({
      message: 'hello, world!',
      status: 'success'
    });
  });

  return router;
};
```

In  *server/index.js* define db and pass table and app to routes:

```js
"use strict";

const config = require("config");
const { table } = require("../orm");
const PORT = config.port || 4000;
const express = require("express");
const routes = require("./routes");

var app = express();

app.use("/", routes(app, table));

const server = app.listen(PORT, () => {
  console.log(`Server listening on port: ${PORT}`);
});

module.exports = server;
```



Get all Posts :

To get started from test first , add  file *routes.posts.test.js* in test :

```js
'use strict';

process.env.NODE_ENV = 'test';

const chai = require('chai');
const should = chai.should();
const chaiHttp = require('chai-http');
chai.use(chaiHttp);

const server = require('../server');


describe('GET /api/posts', () => {
  it('should return all posts', done => {
    chai
      .request(server)
      .get('/api/posts')
      .end((err, res) => {
        // there should be no errors
        should.not.exist(err);
        // there should be a 200 status code
        res.status.should.equal(200);
        // the response should be JSON
        res.type.should.equal('application/json');
        // the JSON response body should have a
        // key-value pair of {'status': 'success'}
        res.body.status.should.eql('success');
        // the JSON response body should have a
        // key-value pair of {'data': [3 post objects]}
        res.body.data.length.should.eql(6);
        // the first object in the data array should
        // have the right keys
        res.body.data[0].should.include.keys(
          'id',
          'title',
          'body',
          'slug',
          'author_id',
          'created_at',
          'updated_at'
        );
        done();
      });
  });
});

```

Run `npm test`

It should fail as we have not added route in posts.js in routes . Lets add route to return posts:

*/routes/posts.js*

```js
'use strict';

const express = require('express');
const router = express.Router();

module.exports = (app, db) => {

  router.route('/').get(async (req, res) => {  
    const posts = await db('posts').all()
    
    res.json({
      data: posts,
      status: 'success'
    });
    
  });

  return router;
};

```

now run test again , it should pass this time.

```bash
 GET /api/posts
    ✓ should return all posts
```

-----------------------





`CODE:` https://github.com/satyamyadav/postgres-express/tree/v0.1.0-2



--------------------------

`Thank You !!`