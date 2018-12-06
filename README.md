# Sequelize It

> Don't criticize it.

# Getting Started

```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('make-twitter', '', '', {
  host: 'localhost',
  dialect: 'postgres'
});

const User = sequelize.define('user', {
  firstName: { type: Sequelize.STRING },
  lastName: { type: Sequelize.STRING }
});
```

```bash
$ createdb make-twitter
```


# Models

## Defining Models
## CRUD w/ Models

```

User.findAll()
User.findOne({ where: { attribute: value } })
User.findById(userId)
User.create(obj)

user.update()

User.findOrCreate()

user[0].dataValues.id
```


# Migrations

## Create Tables
## Update/Drop Tables
## Data Migrations


# Associations: One to Many  


# Associations: Many to Many  

# Validations



# Seeds

# Sync (don't use this)

Don't use sync.

// force: true will drop the table if it already exists {force: true}
// User.sync().then(() => {
  // Table created
  // return User.create({
  //   firstName: 'John',
  //   lastName: 'Hancock'
  // });
// });
