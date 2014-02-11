# migration

  Abstract migration framework for node, support javascript and any js preprocessor

## Origin

This project is based on TJ's [node-migrate](https://github.com/visionmedia/node-migrate).

The main difference:

- js preprocessor support (using `--compiler <ext>:<module>` flag)
- env flag
- migrate.opts default flag setting
- timestamp instead of sequnece number in file name when creating migrate file

## Installation

    $ npm install migration

## Usage

    Usage: migrate [options] [command]

    Options:

       -c, --chdir <path>           change the working directory
       -e, --env                    set NODE_ENV, default is development
       --compiler <ext>:<module>    use the given module to create or compile files

    Commands:

       down   [name]    migrate down till given migration
       up     [name]    migrate up till given migration (the default command)
       create [title]   create a new migration file with optional [title]

## Creating Migrations

To create a migration, execute `migrate create` with an optional title. `node-migrate` will create a node module within `./migrations/` which contains the following two exports:

    exports.up = function(next){
      next();
    };

    exports.down = function(next){
      next();
    };

All you have to do is populate these, invoking `next()` when complete, and you are ready to migrate!

For example:

    $ migrate create add-pets
    $ migrate create add-owners

The first call creates `./migrations/20130601000000000-add-pets.js`, which we can populate:

    var db = require('./db');

    exports.up = function(next){
      db.rpush('pets', 'tobi');
      db.rpush('pets', 'loki');
      db.rpush('pets', 'jane', next);
    };

    exports.down = function(next){
      db.rpop('pets');
      db.rpop('pets', next);
    };

The second creates `./migrations/20130601000001000-add-owners.js`, which we can populate:

    var db = require('./db');

    exports.up = function(next){
      db.rpush('owners', 'taylor');
      db.rpush('owners', 'tj', next);
    };

    exports.down = function(next){
      db.rpop('owners');
      db.rpop('owners', next);
    };

## Running Migrations

When first running the migrations, all will be executed in sequence.

    $ migrate
    up : migrations/20130601000000000-add-pets.js
    up : migrations/20130602000000000-add-jane.js
    migration : complete

Subsequent attempts will simply output "complete", as they have already been executed in this machine. `node-migrate` knows this because it stores the current state in `./migrations/.migrate` which is typically a file that SCMs like GIT should ignore.

    $ migrate
    migration : complete

If we were to create another migration using `migrate create`, and then execute migrations again, we would execute only those not previously executed:

    $ migrate
    up : migrates/20130603000000000-coolest-owner.js

You can also run migrations incrementally by specifying a migration.

    $ migrate up 20130605000000000-coolest-pet.js
    up : migrations/20130604000000000-add-pets.js
    up : migrations/20130605000000000-coolest-pet.js
    migration : complete

This will run up-migrations upto (and including) `002-coolest-pet.js`. Similarly you can run down-migrations upto (and including) a specific migration, instead of migrating all the way down.

    $ migrate down 20130601000000000-add-jane.js
    down : migrations/20130602000000000-add-owners.js
    down : migrations/20130601000000000-add-jane.js
    migration : complete

## License 

Copyright (c) 2013 Jarvis Ao Ieong   
Licensed under the MIT license.
