# Sequelize It

> Don't criticize it. Sequelize it.

<!-- Place this tag where you want the button to render. -->
<a class="github-button" href="https://github.com/ajbraus/sequelize-it" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star ajbraus/sequelize-it on GitHub">Star</a>

<!-- Place this tag where you want the button to render. -->
<a class="github-button" href="https://github.com/ajbraus" data-size="large" data-show-count="true" aria-label="Follow @ajbraus on GitHub">Follow @ajbraus</a>


I built this because [the Sequelize Docs](http://docs.sequelizejs.com/) are currently not very good documentation and it has just been years, so time for something better.

The core principles of these docs are:

1. Consistency (of examples, patterns, style, etc)
1. Information Hierarchy (In the order of: getting started, best practices, extra stuff)

For these docs, I will be doing everything with Sequelize V4, Postgresql, on OSX.

Sequelize has lots of ways of doing things. Within these docs, I will only present the one way of doing things. So it is both a usage guide and a style guide.

These docs are built using [docify-cli](https://www.npmjs.com/package/docsify-cli) and hosted using [Github Pages](https://pages.github.com/).

Please contribute to these docs by submitting a pull request to the project's [GitHub repo](https://github.com/ajbraus/sequelize-it).

# Quick Start: Getting Connected

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
  $ npm install sequelize sequelize-cli pg pg-hstore --save
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

  This will create the following folder and file structure:
  ```
    /db
      /config
        - config.json
      /models
        - index.js
      /migrations
      /seeders
  ```

  The `config/config.json` file has the configuration setup for your development, test, and production environments. The `models/index.js` file has boilerplate code that will unify and associate the models you put in the `models` directory.

1. Customize the `config/config.json` file so the database name lines up with the database you created above:

  ```
  {
    "development": {
      "username": null,
      "password": null,
      "database": "my-db",
      "host": "127.0.0.1",
      "dialect": "postgres"
    },
    "test": {
      "username": null,
      "password": null,
      "database": "my-db-test",
      "host": "127.0.0.1",
      "dialect": "postgres"
    },
    "production": {
      "use_env_variable": "PROD_DATABASE_URL"
    }
  }
  ```

1. Test that your connection is live. In your `models/index.js` file add the following code after the variable `sequelize` is defined.

  ```js
  sequelize.authenticate()
    .then(() => {
      console.log('Connection has been established successfully.');
    })
    .catch(err => {
      console.error('Unable to connect to the database:', err);
    });
  ```

1. Database setup and connected! Now we just need to define models and migrations. Then use them to read and write to the database.

# Models & Migrations

## Define a Model and it's Migration

To define a proper model, you also need to create a migration. For simplicity use a the `sequelize-cli` generators to get started:

```bash
$ sequelize model:create --name Todo --attributes title:string,desc:text
```

This creates a `models/todo.js` model file and a corresponding migration in the `migrations` folder.

```js
// models/todo.js
'use strict';
module.exports = (sequelize, DataTypes) => {
  const Todo = sequelize.define('Todo', {
    title: DataTypes.STRING,
    desc: DataTypes.TEXT
  }, {});

  Todo.associate = function(models) {
    // associations can be defined here
  };

  return Todo;
};
```

```js
// migrations/20181206223919-create-todo.js
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Todos', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      title: {
        type: Sequelize.STRING
      },
      desc: {
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
    return queryInterface.dropTable('Todos');
  }
};
```

## Running & Rolling Back Migrations

```bash
$ sequelize db:migrate
```

I don't recommend rolling back migrations at all, but if you feel that you have to, only roll back one and only before you've shipped it to production.

```bash
$ sequelize db:migrate:undo
```

## Define a Custom Migration

Migrations are used in multiple cases:

1. Creating a table for new model
1. Dropping a table
1. Changing data already in the database (called a "data migration")
1. Adding, updating, or deleting columns of an existing table

Let's look at how to create a custom migration for the case of adding some attributes to our existant `Todos` table.

We'll use the command line to generate the migration:

```bash
$ sequelize migration:generate --name add-attr-to-todo
```

Which generates this boilerplate:


```js
// migrations/20181206230335-add-attr-to-todo.js

'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {

  },

  down: (queryInterface, Sequelize) => {

  }
};
```

The `up` method is the functions run when the migration is run, and the `down` method is run when the migration is undone, reversed, or rolledback. Some migration are irreversible, for example many data migrations. But as much as possible, you should write your migrations to be 100% reversible. We use the `queryInterface` methods to make changes to our tables in the `up` section, and the inverse changes in the `down` section

```js
// migrations/20181206230335-add-attr-to-todo.js

'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    queryInterface.addColumn('Todos', 'summary', { type: Sequelize.TEXT });
    queryInterface.addColumn('Todos', 'completedAt', { type: Sequelize.DATE });
  },

  down: (queryInterface, Sequelize) => {
    queryInterface.removeColumn('Todos', 'summary');
    queryInterface.removeColumn('Todos', 'completedAt');
  }
};
```

And then we'll have to update our models:

```js
// models/todo.js
'use strict';
module.exports = (sequelize, DataTypes) => {
  const Todo = sequelize.define('Todo', {
    title: DataTypes.STRING,
    summary: DataTypes.TEXT,
    completedAt: DataTypes.DATE
  }, {});

  Todo.associate = function(models) {
    // associations can be defined here
  };

  return Todo;
};
```

## Data Migrations

TODO


## QueryInterface Methods

Here are an exhaustive list of the [queryInterface methods](http://docs.sequelizejs.com/class/lib/query-interface.js~QueryInterface.html). The most common ones are:

* `addColumn(table: String, key: String, attribute: Object, options: Object)`
* `renameColumn(tableName: String, attrNameBefore: String, attrNameAfter: String, options: Object)`
* `changeColumn(tableName: String, attributeName: String, dataTypeOrOptions: Object, options: Object)` (changes the column's type)
* `removeColumn(tableName: String, attributeName: String, options: Object)`
* `addIndex(tableName: String, options: Object)`

## Sequelize Data Types
Here are an exhaustive list of [Sequelize Types](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#data-types) you can use. The most common are as follows:

```js
Sequelize.STRING                      // VARCHAR(255)
Sequelize.TEXT                        // TEXT
Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.FLOAT                       // FLOAT
Sequelize.DECIMAL                     // DECIMAL
Sequelize.DATE                        // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
Sequelize.BOOLEAN                     // TINYINT(1)
```

# CRUD w/ Models

Now that we've defined our models
All the query helpers return promises, so either use the `then`/`catch` pattern or the `async`/`await` pattern.

> GOTCHA - you can't just call `user.fullName` on the result like you might expect. You have to add `raw: true` to the query and then access it like this: `user[0].fullName`.

We'll include our models into any file using this line of code:

```js
const models  = require('../db/models');
```

## Index

`.findAll()` with `where`

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

`.findById()` (if you have the id)

```js
models.User.findById(userId).then(user => {

}).catch(err => ({

})

// OR WITH ASYNC/AWAIT

try {
  let user = await models.User.findById(userId);
} catch (err) {

}
```

`findOne` (if you do not have the id)

```js
models.User.findOne({ where: { attribute: value } }).then(user => {

}).catch(err => ({

})

// OR WITH ASYNC/AWAIT

try {
  let user = await models.User.findById(userId);
} catch (err) {

}
```


## Create

`.create()`

```js
models.Task.create(req.body).then(task => {
  // you can now access the newly created task via the variable task
}).catch(err => ({

})

// OR USE ASYNC/AWAIT

let task = await models.Task.create(req.body);
```

Or if you prefer the railsy way — `.build()` then `.save()`

```js
models.Task.build(req.body).then(task => {
  task.save().then(task => {
  }).catch(err => ({

  });
  // you can now access the newly created task via the variable task
}).catch(err => ({

})

// OR WITH ASYNC/AWAIT

try {
  let user = await models.Task.build(req.body);
  user = await user.save();
} catch (err) {

}
```

## Update

`.update()`

```js
models.Task.findById(taskId).then(task => {
  task.update(req.body).then(task => {

  }).catch(err => ({

  })
}).catch(err => ({

})

// OR USE ASYNC/AWAIT

try {
  let task = await models.Task.findById(taskId);
  task = await task.update(req.body);
} catch (err) {

}
```

## Destroy

`.destroy()`

```js
Task.findById(taskId).then(task => {
  task.destroy()
}).catch(err => ({

})

// OR USE ASYNC/AWAIT

try {
  let task = await models.Task.findById(taskId);
  task.destroy();
} catch (err) {

}
```

## Find or Create

A nice thing to have! `.findOrCreate()`

```js
let tag = models.Tag.findOrCreate(req.body)
```

# Associations: One to Many  

TODO

## Defining Association

First you'll have to add a foreign key through a migration. The only way to do this is to chain a `addColumn` function and an `addConstraint` function afterwards.

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn('Tweets', 'UserId', Sequelize.INTEGER).then(() => {
      return queryInterface.addConstraint('Tweets', ['UserId'], {
        type: 'foreign key',
        name: 'user_tweets',
        references: { //Required field
          table: 'Users',
          field: 'id'
        },
        onDelete: 'CASCADE',
        onUpdate: 'CASCADE'
      });
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.removeColumn('Tweets', 'UserId');
  }
};

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

## Include, or Eager Loading

```js
// Find all the pugs, and include their owner
const pugs = await Pug.findAll({ include: [{ model: Owner }] })
```

**GOTCHA** - the included records will be available only if you capitalize their name `pug.Owner`.

You can add a where query to this and also

You can include either one model as the example above or use the option `all: true` to include all associations. There does not seem to be a way to pick and choose which associated models to include.

```js
// Find one event, and include all its associated models
const event = await Event.findById(eventId, { include: [{ all: true }] });
```

You can also add other options to the include request:

```js
{
  model: User,
  attributes: ['name', 'image'],
  as: "Author",
  where: { isPublished: true }
}
```

## Getters and Setters

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

TODO

## onDelete

`onDelete` TODO


# Associations: Many to Many  

## Associating Two Tables

```js
Event.belongsToMany(User, {through: 'rsvps'})
User.belongsToMany(Event, {through: 'rsvps', foreignKey: 'GuestId', as: 'Guests'})
```

Create a `rsvps` table with two foreign keys

```
createdAt | updatedAt | GuestId | EventId
```

```js
User.getEvents();
Event.getGuests();
```

## Self-Referential Many-to-Many Association

Standard - Use `as`

```js
Item.hasMany(Item, {as: 'Subitems'})

Item.find({where:{name: "Coffee"}, include:[{model: Item, as: 'Subitems'}]})


```

```
createdAt | updatedAt | ItemId | SubItemId
```

Bi-Directional - Use `foreignKey` & `through`

```js
User.hasMany(User, {
    as: 'Friends',
    foreignKey: 'FriendId',
    through: 'friends'
})

User.getFriends()
```

Create a `friends` table

```
createdAt | updatedAt | UserId | FriendId
```

```js
User.hasMany(User, { // who follows you
  as: 'Followers',
  foreignKey: 'FollowId',
  through: 'follows'
})

User.hasMany(User, { // who you follow
  as: 'Follows',
  foreignKey: 'FollowerId',
  through: 'follows'
})

User.getFollows();
User.getFollowers();
```

Create a `follows` table

```
createdAt | updatedAt | FollowId | FollowerId
```

## Fetching Associated Records

```js
pug.getFriends() // returns a promise for the array of friends for that pug
pug.addFriend(friend) // creates a new row in the friendship table for the pug and the friend, returns a promise for the friendship (NOT the pug OR the friend - the "friendship")
pug.addFriends(friendsArray) // creates a new row in the friendship table for each friend, returns a promise for the friendship
pug.removeFriend(friend) // removes the row from the friendship table for that pug-friend, returns a promise for the number of affected rows (as if you'd want to destroy any friendships...right?)
pug.removeFriends(friendsArray) // removes the rows from the friendship table for those pug-friend pairs, returns a promise for the number affected rows

// analogous to above ^
friend.getPugs()
friend.addPug(pug)
friend.addPugs(pugsArray)
friend.setPugs(pugsArray)
friend.removePug(pug)
friend.removePugs(pugsArray)
```

# Configure db Connection

There are some verbose configuration settings you can use as your implementation gets more sophistocated:

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

# Paranoid Setting

If the `paranoid` options is true, the object will not be deleted, instead the `deletedAt` column will be set to the current timestamp. To force the deletion, you can pass force: true to the destroy call:

```js
task.destroy({ force: true })
```

# Validations

Sequelize.js uses [validator.js](https://github.com/chriso/validator.js) to add **Validators** to your models.

```js
// models/todo.js
'use strict';
module.exports = (sequelize, DataTypes) => {
  const Todo = sequelize.define('Todo', {
    title: {
      DataTypes.STRING,
      allowNull: false,
      defaultValue: "Gotta Do it!",
      validate: { min: 0, max: 180 }
    }
  }, {});

  Todo.associate = function(models) {
    // associations can be defined here
  };

  return Todo;
};
```

## Custom Validators

You can use the `validate` option to add custom validators.

```js
// models/todo.js
'use strict';
module.exports = (sequelize, DataTypes) => {
  const Todo = sequelize.define('Todo', {
    title: {
      DataTypes.STRING,
      allowNull: false,
      defaultValue: "Gotta Do it!",
      validate: { min: 0, max: 180 }
    },
    latitude: {
      type: Sequelize.INTEGER,
      allowNull: true,
      defaultValue: null,
      validate: { min: -90, max: 90 }
    },
    longitude: {
      type: Sequelize.INTEGER,
      allowNull: true,
      defaultValue: null,
      validate: { min: -180, max: 180 }
    },
  }, {
    validate: {
      bothCoordsOrNone() {
        if ((this.latitude === null) !== (this.longitude === null)) {
          throw new Error('Require either both latitude and longitude or neither')
        }
      }
    }
  });

  Todo.associate = function(models) {
    // associations can be defined here
  };

  return Todo;
};
```

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

## Limit Attributes

```js
let tasks = await models.Task.findAll({ attributes: ['title', 'createdAt'] });
```

## Exclude Attributes

```js
let tasks = await models.User.findAll({ attributes: { exclude: ['email', 'password'] });
```

# Virtuals

TODO

# Seeds

## Generate New Seeder File

```bash
$ sequelize seed:generate --name demo-user
```

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert('Users', [{
        firstName: 'John',
        lastName: 'Doe',
        email: 'demo@demo.com'
      }], {});
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete('Users', null, {});
  }
};
```

## Run Seeds

```bash
$ sequelize db:seed:all
```

# Stuff You'll Use Less Often

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

See the [Sequelize docs](http://docs.sequelizejs.com/manual/tutorial/scopes.html) how to use scopes.

## Raw SQL Queries

It can be done!

## Sync (don't use this)

Don't use sync, just use migrations.


todo

adding raw: true to flatten response to an array => user[0].attribute
