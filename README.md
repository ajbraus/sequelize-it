# Sequelize It

> Don't critize it.

> An awesome project.

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

```

User.findOrCreate()
User.findAll()
User.create

user.update()

user[0].dataValues.id
```

# Getting Started


# Sync

Don't use sync.

// force: true will drop the table if it already exists {force: true}
// User.sync().then(() => {
  // Table created
  // return User.create({
  //   firstName: 'John',
  //   lastName: 'Hancock'
  // });
// });

# Create Models
