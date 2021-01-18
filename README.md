# knex-project-example
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
