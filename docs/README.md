# Sequelize It

> Don't criticize it.

I built this because [the Sequelize Docs](http://docs.sequelizejs.com/) are not very good documentation.

For these docs, I will be doing everything with Sequelize V4, Postgresql, on OSX.

Sequelize has lots of ways of doing things. Within these docs, I will only present the one way that I suggest to use.

# Getting Started

1. Sequelize works with many flavors of SQL databases, so install one of them your computer.
  * mysql
  * sqlite
  * postgres - [postgresapp](https://postgresapp.com/) is an easy way to get postgres installed
  * mssql

1. Create your db

  ```bash
  $ createdb my-db
  ```

1. Install sequelize, sequelize-cli, and your database's modules (here postgres) locally:

  ```bash
  $ npm install sequelize sequelize-cli pg pg-hstore -g
  ```

1. Initializing Sequelize with the `sequelize-cli` kinda barfs a bunch of folders and files onto your project, so we are going to preemptively avoid that by setting up a `.sequelizerc` file puts all sequelize stuff into a folder called `/db`. So make that file with the following contents:

  ```
    const path = require('path');
    module.exports = {
      "config": path.resolve('./db/config', 'config.json'),
      "models-path": path.resolve('./db/models'),
      "seeders-path": path.resolve('./db/seeders'),
      "migrations-path": path.resolve('./db/migrations')
    };
  ```

1. Ok now let's initialize Sequelize with the command line tool

  ```bash
    $ sequelize init
  ```

1. Connect to your local db:

  ```js
  const Sequelize = require('sequelize');
  const username = '';
  const password = '';
  const sequelize = new Sequelize('my-db', username, password, {
    host: 'localhost',
    dialect: 'postgres'
  });
  ```

1. Define a Model

  ```js
  const User = sequelize.define('user', {
    firstName: { type: Sequelize.STRING },
    lastName: { type: Sequelize.STRING }
  });
  ```

# Models & Migrations

## Define a Model and it's Migration

```bash
$ sequelize model:create
```

```js

'use strict';
module.exports = (sequelize, DataTypes) => {
  var User = sequelize.define('User', {
    first_name: DataTypes.STRING,
    last_name: DataTypes.STRING,
    bio: DataTypes.TEXT
  })

  User.associate = function(models) {
    User.hasMany(models.Post); // PostId
  }

  return User;
};
```

## CRUD w/ Models

All the query helpers return promises, so either use the `then` `catch` pattern or the `async` `await` pattern.

> GOTCHA - you can't just call `user.fullName` like a normal library would work. You have to call it like this: `user.get('fullName')`. Ugh.

We'll include our models into any file using this line of code:

```js
const models  = require('../db/models');
```

## Index

```js
models.User.findAll({ where: { name: "Betty" } }).then(users => {

}).catch(err => ({

})

// OR WITH ASYNC/AWAIT

try {
  let user = await models.User.findAll({ where: { name: "Betty" } });
} catch (err) {

}
```

## Show

`findById()`

```js
models.User.findById(userId).then(user => {

}).catch(err => ({

})
```

`findOne`

```js
models.User.findOne({ where: { attribute: value } }).then(user => {

}).catch(err => ({

})
```


## Create

```js
models.Task.create(req.body).then(task => {
  // you can now access the newly created task via the variable task
}).catch(err => ({

})

// OR USE ASYNC/AWAIT

let task = await models.Task.create(req.body);
```

## Update

```js
models.Task.findById(taskId).then(task => {
  task.update(req.body).then(task => {

  }).catch(err => ({

  })
}).catch(err => ({

})

// OR USE ASYNC/AWAIT

let task = await models.Task.findById(taskId);
task = await task.update(req.body);
```

## Destroy
```js
Task.findById(taskId).then(task => {
  task.destroy();
}).then(() => {
 // now i'm gone :)
}).catch(err => ({

})
```

## Find or Create

A nice thing to have!

```js
models.User.findOrCreate()
```


# Migrations

## Create Tables

```js
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      first_name: {
        type: Sequelize.STRING
      },
      last_name: {
        type: Sequelize.STRING
      },
      bio: {
        type: Sequelize.TEXT
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Users');
  }
};
```

## Update/Drop Tables

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    queryInterface.addColumn('Users', 'dob', { type: Sequelize.DATE });
  },

  down: (queryInterface, Sequelize) => {
    queryInterface.removeColumn('Users', 'dob');
  }
};
```

## Data Migrations


# Associations: One to Many  

## Defining Association


## Fetching Children

```js
const models  = require('../db/models');

//POSTS#SHOW
app.get('/posts/:id', (req, res, next) {
  models.Post.findById(req.params.id).then(post => {
    post.getComments({ order: [['createdAt', 'DESC']] }).then(comments => {
      res.render('posts-show', { post: post, comments: comments})
    });
  });
});
```

> Look how cool this same code looks if we use `async` and `await` :D Compare & Contrast

```js
const models  = require('../db/models');

//POSTS#SHOW
app.get('/posts/:id', async (req, res, next) => {
  try {
    let post = await models.Post.findById(req.params.id);
    let comments = await post.getComments({ order: [['createdAt', 'DESC']] });
  } catch (err) {
    console.log(err);
  }

  res.render('posts-show', { post: post, comments: comments});
});
```

## Fetching Parent




## onDelete

`onDelete`


# Associations: Many to Many  

## Defining Association


## Fetching Associated Records


# Configure db Connection

```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },

  // SQLite only
  storage: 'path/to/database.sqlite',

  // http://docs.sequelizejs.com/manual/tutorial/querying.html#operators
  operatorsAliases: false
});
```

Or you can use a URI to connect:

```js
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

## Test Connection

If you can't tell if you've connected to your db, you can use this test to see:

```js

const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

sequelize.authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```

# Paranoid Setting

If the `paranoid` options is true, the object will not be deleted, instead the `deletedAt` column will be set to the current timestamp. To force the deletion, you can pass force: true to the destroy call:

```js
task.destroy({ force: true })
```

# Validations

# Hooks

```js
User.hook('beforeSave', (user, options) => {
  // encrypt password
});
```

Order of Operations

```
(1)
  beforeBulkCreate(instances, options)
  beforeBulkDestroy(options)
  beforeBulkUpdate(options)
(2)
  beforeValidate(instance, options)
(-)
  validate
(3)
  afterValidate(instance, options)
  - or -
  validationFailed(instance, options, error)
(4)
  beforeCreate(instance, options)
  beforeDestroy(instance, options)
  beforeUpdate(instance, options)
  beforeSave(instance, options)
  beforeUpsert(values, options)
(-)
  create
  destroy
  update
(5)
  afterCreate(instance, options)
  afterDestroy(instance, options)
  afterUpdate(instance, options)
  afterSave(instance, options)
  afterUpsert(created, options)
(6)
  afterBulkCreate(instances, options)
  afterBulkDestroy(options)
  afterBulkUpdate(options)
```

# Advanced Querying

## Where (advanced)

```js
const Op = Sequelize.Op;

Project.findAll({
  where: {
    id: {
      [Op.and]: {a: 5},           // AND (a = 5)
      [Op.or]: [{a: 5}, {a: 6}],  // (a = 5 OR a = 6)
      [Op.gt]: 6,                // id > 6
      [Op.gte]: 6,               // id >= 6
      [Op.lt]: 10,               // id < 10
      [Op.lte]: 10,              // id <= 10
      [Op.ne]: 20,               // id != 20
      [Op.between]: [6, 10],     // BETWEEN 6 AND 10
      [Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
      [Op.in]: [1, 2],           // IN [1, 2]
      [Op.notIn]: [1, 2],        // NOT IN [1, 2]
      [Op.like]: '%hat',         // LIKE '%hat'
      [Op.notLike]: '%hat',       // NOT LIKE '%hat'
      [Op.iLike]: '%hat',         // ILIKE '%hat' (case insensitive)  (PG only)
      [Op.notILike]: '%hat',      // NOT ILIKE '%hat'  (PG only)
      [Op.overlap]: [1, 2],       // && [1, 2] (PG array overlap operator)
      [Op.contains]: [1, 2],      // @> [1, 2] (PG array contains operator)
      [Op.contained]: [1, 2],     // <@ [1, 2] (PG array contained by operator)
      [Op.any]: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)
    },
    status: {
      [Op.not]: false           // status NOT FALSE
    }
  }
})
```

## Include TODO

```js
Project.findAll({
    include: [{
        model: Task,
        where: { state: Sequelize.col('project.state') }
    }]
})

```


## Order

```js
Subtask.findAll({
  order: [
    // Will escape title and validate DESC against a list of valid direction parameters
    ['title', 'DESC'],

    // Will order by max(age)
    sequelize.fn('max', sequelize.col('age')),

    // Will order by max(age) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // Will order by  otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // Will order an associated model's created_at using the model name as the association's name.
    [Task, 'createdAt', 'DESC'],

    // Will order through an associated model's created_at using the model names as the associations' names.
    [Task, Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using the name of the association.
    ['Task', 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using the names of the associations.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using an association object. (preferred method)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using association objects. (preferred method)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using a simple association object.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at simple association objects.
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
  ]

  // Will order by max age descending
  order: sequelize.literal('max(age) DESC')

  // Will order by max age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.fn('max', sequelize.col('age'))

  // Will order by age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.col('age')

  // Will order randomly based on the dialect (instead of fn('RAND') or fn('RANDOM'))
  order: sequelize.random()
})
```

## Pagination

```
// Skip 5 instances and fetch the 5 after that
Project.findAll({ offset: 5, limit: 5 })
```

# Limit or Exclude Instance Attributes

## Limit

```js
let tasks = await models.Task.findAll({ attributes: ['title', 'createdAt'] });
```

## Exclude
```js
let tasks = await models.User.findAll({ attributes: { exclude: ['email', 'password'] });
```

# Virtuals

# Seeds

# Less Useful Stuff

## Bulk Creating

```js
User.bulkCreate([
  { username: 'barfooz', isAdmin: true },
  { username: 'foo', isAdmin: true },
  { username: 'bar', isAdmin: false }
]).then(() => { // Notice: There are no arguments here, as of right now you'll have to...
  return User.findAll();
}).then(users => {
  console.log(users) // ... in order to get the array of user objects
})
```

## Incrementing & Decrementing

```js
User.findById(1).then(user => {
  return user.increment('my-integer-field', {by: 2})
}).then(user => {
  // Postgres will return the updated user by default (unless disabled by setting { returning: false })
  // In other dialects, you'll want to call user.reload() to get the updated instance...
})
```

## Scopes

These are virtually useless. See the [terrible Sequelize docs](http://docs.sequelizejs.com/manual/tutorial/scopes.html) to use them.

## Raw SQL Queries

You can do these, but why would you do that?

## Sync (don't use this)

Don't use sync.

// force: true will drop the table if it already exists {force: true}
// User.sync().then(() => {
  // Table created
  // return User.create({
  //   firstName: 'John',
  //   lastName: 'Hancock'
  // });
// });
