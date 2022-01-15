---
title: Building a REST-API with Expressjs, Tabel and Postgres | Part-2
date: "2016-05-02"
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

Setting up database and orm.  [Tabel](http://tabel.fractaltech.in/) : A simple orm for PostgreSQL, built over knex.js, which works with simple javascript objects and arrays. More of a table gateway that can behave like an orm, and scale back down to a table-gateway when needed.

<!-- more  -->

**Install postgress**

Set up  `postgres`  and `redis` on your system.

A repository for helping in instalations. [Link Here](https://github.com/dg92/node-express-postgres-redis-starter-kit)

Postgres docs : [Link](https://www.postgresql.org/download/)

Redis docs: [Link](https://redis.io/download)

-----------



**Configure [Tabel](http://tabel.fractaltech.in/) ORM**

```bash
npm install config tabel pg

mkdir config

touch config/default.js
```
Copy config for orm from [Tabel](http://tabel.fractaltech.in/) docs and update db name, user and password. 

------------



**Migrations**

```bash

mkdir migerations

touch migrate.js

```

/migrate.js

```js
const migrate = require('tabel/lib/migrate');
const ormConfig = require('config');

migrate(ormConfig);
```
After adding above content to migrate.js run following migerate commonds from base path:
```sh
node migrate.js make CreateUsersTable
node migrate.js make CreatePostsTable
```

Now there should appear the migeration files in /migerations as <TIME_STAMP>_CreateUsersTable.js and <TIME_STAMP>__CreatePostsTable.js with content 

```js
function up(knex) {

}

function down(knex) {

}

module.exports = {up, down};
```
update these file contents then run 
```bash
node migrate.js latest
```

now we can verify it in psql it should show blank tables created :

```bash
 api_dev=# \dt
```

List of relations               

| Schema | Name   | Type                 | Owner |      |
| ------ | ------ | -------------------- | ----- | ---- |
|        | public | knex_migrations      | table | dev  |
|        | public | knex_migrations_lock | table | dev  |
|        | public | posts                | table | dev  |
|        | public | users                | table | dev  |

-------------



**Table Definitions**

Define Tables, a single file is sufficient to define tables but we can have a break down table specific files (modules) for definition as follow:

```bash
mkdir orm
mkdir orm/tables
touch orm/index.js
mkdir orm/tables
touch orm/tables/users.js
touch orm/tables/posts.js
```
Add definition as in full config section of [tabel docs](http://tabel.fractaltech.in/table-definitions.html#Full-Config) for posts and users both.

Example:
```js
'use strict';

module.exports = function(orm) {
  orm.defineTable({
    name: 'users',
    props: {
      key: 'id',
      autoId: false,
      perPage: 25,
      timestamps: true
    },
    scopes: {},
    joints: {
      joinComments() {
        return this.joinPosts().join(
          'comments',
          'comments.post_id',
          '=',
          'posts.id'
        );
      },
      joinPosts() {
        return this.join('posts', 'users.id', '=', 'posts.author_id');
      }
    },
    relations: {
      posts: function() {
        return this.hasMany('posts', 'author_id');
      }
    }
  });
};

```
and require the two definitions in orm/tables/index.js

```js
module.exports = function(orm) {
  require('./users')(orm);
  require('./posts')(orm);
};
```
now require the tables in orm/index.js

```js
const Tabel = require('tabel');
const config = require('config');
const orm = new Tabel(config);

require('./tables')(orm);

module.exports = orm.exports;
```

--------------



**Seeds**

To seed some sample (fake data):

```
mkdir seeds
touch seeds/post.js
touch seeds/user.js
```
In seed files `insert` some data into table using tabel orm methods according to schema in migerations , I have used [Faker.js](https://github.com/marak/Faker.js/).

`npm install faker`

Example Posts :

```js
const faker = require('faker');
const { table } = require('../orm');

var seed = Promise.all(
  [1, 2, 3].map(function(n) {
    return table('posts').insert([
      {
        id: faker.random.uuid(),
        author_id: faker.random.uuid(),
        title: faker.lorem.sentence(),
        body: faker.lorem.sentence(),
        slug: faker.lorem.sentence().replace(/ /g, '-')
      }
    ]);
  })
);

seed.then(d => {
  console.log('seeded Posts', d);
});

```

--------


**Server**

Now we make an app (express app) and define routes. Before that let's verify if everything is working now by logging query results to console.

```bash
mkdir server
touch index.js
```

Adding a query and console log for the result in /server/index.js

```js
const { table } = require('../orm');

const posts = table('posts').all();

posts.then(d => console.log(d));

```

Now run `node sever` from command from root of project. It should print the list of posts we have seeded in posts table.

------------------------



Next we need to define an express app and add test for routes and then add routes and controllers for creating end points for REST API.





`CODE:` https://github.com/satyamyadav/postgres-express/tree/v0.1.0-1


--------------------------

`Thank You !!`
