
# knex-project-example

#### A. Creating a Basic Knex sqlite3
1. <code>$ npm init -y</code>
2. <code>$ npm i knex sqlite3</code>
3. <code>$ knex init</code>

<code>package.js</code>
```js
module.exports = {

  // ....

  "scripts": {
    "dev": "nodemon src/main.js",
    "dbu": "knex migrate:latest",
    "dbd": "knex migrate:rollback",
    "dbi": "npm run dbd && npm run dbu",
    "seed": "knex seed:run"
  },

  // ....

}
```


<code>knexfile.js</code>
```js
module.exports = {

    development: {
      client: 'sqlite3',
      connection: {
        filename: './src/db/dev.sqlite3'
      },
      migrations: {
        tableName: 'knex_migrations',
        directory: './src/db/migrations'
      },
      seeds: {
        directory: './src/db/seeds'
      },
      useNullAsDefault: true
    }, 
}
```
4. <code>$ knex migrate:make create_tables</code>

<code>xxxxx_create_tables.js</code>
```js
const migrations = require('./../migrations')

exports.up = async (knex, Promise) => {
    for(key in migrations)
        await knex.schema.createTable(key, migrations[key])
    return knex
}

exports.down = async (knex, Promise) => {
    for(key in migrations)
        await knex.schema.dropTable(key)
    return knex
}
```
5. <code>$ touch src/db/migrations.js</code>

<code>migrations.js</code>
```js
module.exports = {
    users: (table) => {
        table.increments().primary()
        
        table.timestamps(true, true)
    },
}
```

5. <code>Instantiate Knex Object</code>
```js
const { development } = require('./../../knexfile')
const knex = require('knex')(development)
```

```cmd
$ npm init -y && npm i knex sqlite3 && knex init && touch knexfile.js
```

### Seeds

#### A. Creating Seeds
1. <code>$ knex seed:make 01-seed_name</code>

<code>01-seed_name.js</code>

```js
exports.seed = function(knex) {
  // Deletes ALL existing entries
  return knex('table_name').truncate()
    .then(function () {
      // Inserts seed entries
      return knex('table_name').insert([
        {id: 1, colName: 'rowValue1'},
        {id: 2, colName: 'rowValue2'},
        {id: 3, colName: 'rowValue3'}
      ]);
    });
};
```
2. <code>$ knex seed:run</code>

### A+. Add Objection
6. <code>$ npm i objection</code>
7. <code>$ mkdir src/models</code>
8. <code>$ touch src/models/Model.js</code>

```cmd
$ knex migrate:make create_tables && touch src/db/migrations.js && npm i objection && mkdir src/models && touch src/models/Model.js
```

5. <code>Instantiate Knex Object</code>

<code>Model.js</code>
```js
const { development } = require('./../../knexfile')
const knex = require('knex')(development)
const { Model } = require('objection')
Model.knex(knex)


class BaseModel extends Model {

}

module.exports = BaseModel
```

6. <code>Example Class</code>
```js
const Model = require('./Model')
const bcrypt = require('bcrypt')
const BCRYPT_ROUNDS = 12

class User extends Model {
    
    async verifyPassword(password){
        return await bcrypt.compare(password, this.password)    
    }

    async $beforeInsert(context){
        await super.$beforeInsert(context)
        if(this.password) this.password = await bcrypt
            .hash(this.password, BCRYPT_ROUNDS)
    }

    async $beforeUpdate(context){
        await super.$beforeInsert(context)
        if(this.password) this.password = await bcrypt
            .hash(this.password, BCRYPT_ROUNDS)
    }

    static get relationMappings(){
        
        const Role = require('./Role')

        return {
            roles: {
                relation: Model.ManyToManyRelation,
                modelClass: Role,
                join: {
                    from: 'users.id',
                    to: 'roles.id',
                    through: {
                        from: 'user_roles.user_id',
                        to: 'user_roles.role_id'
                    }
                }
            }
        }
    }
    
}

module.exports = User
```

#### B. Creating a Basic Knex HTTP Server

1. create a new folder.
2. open this folder in VSCode.
3. initialize your node-js project
<code>$ knex init</code>
4. install koa
<code>$ npm i koa </code>
5. create a main.js file

```js
const Koa = require('koa')
const app = new Koa()
const port = 4000

app.listen(port)
```
#### Adding Controllers and Middleware
A koa server support middleware and controller functions. 
- middleware: modifies the request then passes it on to the next middleware.
- controller: modifies the request and responds to the user. 

```js
const middleware = async (ctx, next) => {
    // modify the request object here.
    await next()
}

const controller = async (ctx) => {
    // modify the request object here.
    ctx.body = 'add your response here'
}
```
We can add add middleware and controllers by doing the following:

```js
app.use(middleware, controller)
```
<code>$ knex migrate:latest</code>
5. drop db_table from migrations
<code>$ knex migrate:rollback</code>

#### Project
```cmd
/Users/david/Desktop/db-test
├── knexfile.js
├── package-lock.json
├── package.json
└── src
   ├── db
   |  ├── db.sqlite3
   |  ├── migrations
   |  |  └── 20210118121642_create_migrations.js
   |  └── migrations.js
   ├── main.js
   ├── models
   └── utils
```

#### Setup
1. installation
<code>$ npm i knex sqlite3</code>
2. create knexfile.js
<code>$ knex init</code>
3. create migration_table.js
<code>$ knex migrate:make create_table_name</code>
4. create db_table from migrations
<code>$ knex migrate:latest</code>
5. drop db_table from migrations
<code>$ knex migrate:rollback</code>


#### Migrations
```js

const users = (table) => {
    table.increments('id')
    table.string('first_name', 255).notNullable()
    table.string('last_name', 255).notNullable()
    table.string('birth_date', 255).notNullable()
    table.timestamps(true, true)
}

const roles = (table) => {
    table.increments('id')
    table.string('name', 255).notNullable()
    table.string('description', 255).notNullable()
    table.timestamps(true, true)
}

const user_roles = (table) => {
    table.increments('id')
    table.string('name', 255).notNullable()
    table.string('description', 255).notNullable()
    table.timestamps(true, true)
}

exports.up = (knex) => knex.schema
    .createTable('users', users)
    .createTable('roles', roles)
    .createTable('user_roles', user_roles)

exports.down = (knex) => knex.schema
    .dropTable('users')
    .dropTable('roles')
    .dropTable('user_roles')

```

#### Auto Generated Migrations
```js
const migrations = {

    users: (table) => {
        table.increments().primary()
        table.string('userId').unique().notNullable()
        table.string('username').notNullable().unique()
        table.date('birth_date', 255).notNullable()
        table.string('email').notNullable().unique()
        table.string('password').notNullable()
        table.boolean('confirmed').defaultTo(false)
        table.boolean('suspended').defaultTo(false)
        table.boolean('disabled').defaultTo(false)
        table.timestamps(true, true)
    }, 

    roles: (table) => {
        table.increments().primary()
        table.string('name').unique().notNullable()
        table.text('description')
        table.timestamps(true, true)
    },

    user_roles: (table) => {
        table.increments().primary()
        table.unique(['user_id', 'role_id'])
        table.integer('user_id').references('id').inTable('users').onDelete('CASCADE').index().notNullable()
        table.integer('role_id').references('id').inTable('roles').onDelete('CASCADE').index().notNullable()
        table.timestamps(true, true)
    } 
}

exports.up = async (knex, Promise) => {
    for(key in migrations)
        await knex.schema.createTable(key, migrations[key])
    return knex
}

exports.down = async (knex, Promise) => {
    for(key in migrations)
        await knex.schema.dropTable(key)
    return knex
}
```

#### Files

##### package.json

```json
"scripts": {
    "dev": "nodemon src/main.js",
    "dbup": "knex migrate:latest",
    "dbdown": "knex migrate:rollback",
    "init": "npm run dbdown && npm run dbup"
},
```

##### knexfile.js

```js
// Update with your config settings.

module.exports = {

  development: {
    client: 'sqlite3',
    connection: {
      filename: './src/db/db.sqlite3'
    },
    migrations: {
      tableName: 'knex_migrations',
      directory: './src/db/migrations'
    }
  }
};

```

##### main.js
```js
global.c = (val) => console.log(val)
const knexfile = require('./../knexfile')
const knex = require('knex')(knexfile.development)

const main = async () => {
    const res1 = await knex('users').insert({
        first_name: 'david',
        last_name: 'chen',
        birth_date: '1990-09-23'
    })
    const data = await knex.select('*').from('users')
    c({res1, data})
}

const run = async () => {
    try {
        await main()
    } catch (err) {
        c(err)
    }   
}

run()
```
