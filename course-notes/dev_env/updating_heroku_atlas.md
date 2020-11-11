# Updating Heroku Stack and MongoDB Atlas

This page documents some notes relevant to updating the Capstone DemoApp environment 
from Heroku Cedar-14 Stack and mLab MongoDB.

Two issues to keep in mind

* the Capstone DemoApp has been previously checked in with no reference
to a Ruby version so that you did not have to edit that value
in the event that it was used in an alternate environment
* Heroku Ruby stacks have a default version and might not be 
compatible with what is being deployed and may need an explicit
version expressed. However, I will try to leave it unspecified.
* the current Ruby version for the Capstone DemoApp is 2.3.8 and 
the most recent Heroku stack with Ruby 2.3 support is heroku-16.
It is advertsied to be supported thru April 2021.
https://devcenter.heroku.com/articles/stack
* Mongo 3.x is not available for free on Mongo Atlas. Must update
database_cleaner and mongo to a version that supports MongoDB 4.2.

## Establish a new Free Account and MongoDB instance on Mongo Altas

1. Create a free user, project, and cluster with Mongo Altas.
  * https://www.mongodb.com/cloud/atlas

2. create a dbuser with read/write access and remember the password assigned
3. obtain the URL to the database
  * e.g., mongodb://dbuser:(pwd)@capstone-dev-shard-00-00.ue4he.mongodb.net:27017...

## Create a new environment using heroku-16

Follow the posted instructions on how to create a new environment and 
transfer plugins and settings to the new environment

* https://devcenter.heroku.com/articles/upgrading-to-the-latest-stack

You should finish with a rew git remote named `heroku-16`. Use that remote
from this point forward.

Add the following new properties to turn off `database_cleaner` safety checks
during the rake DB managemen scripts.

  ```text
  heroku config:set --remote heroku-16 export DATABASE_CLEANER_ALLOW_REMOTE_DATABASE_URL=true
  heroku config:set --remote heroku-16 DATABASE_CLEANER_ALLOW_REMOTE_DATABASE_URL=true
  ```

## Gemfile Updates

1. Designate a version for Ruby
2. Update `database_cleaner` to 1.8.5
3. Update `mongo` to 2.13.1
  * https://docs.mongodb.com/ruby-driver/master/reference/driver-compatibility/

    ```text
    ruby '2.3.8'
    gem 'mongoid', '~>5.1.5'
    gem 'mongo', '~>2.13.1'
    gem 'database_cleaner', '~>1.8.5'
    ```

4. update the Gemfile.lock

  ```text
  $ bundle update mongo
  ```

5. Commit those changes to a branch (e.g., `ruby238`)


## Upload Application

  ```text
  $ git push heroku-16 ruby238:master
  ```

## Initialize the Database

  ```text
  $ heroku heroku16 run rake db:migrate
  $ heroku run bundle exec rake ptourist:reset_all --remote heroku-16
  ```

## Update an existing application

Once you have confidence in your new test environment, you can try updating existing
environments. The following commands are being executed against a Heroku environment
aliased as `dev` in git remotes.

  ```text
  $ heroku stack:set heroku-16 -a (app name)
  $ heroku config:set --remote dev MLAB_URI='mongodb://.../dev...
  $ git commit --allow-empty -m "Upgrading to heroku-16"
  $ git push dev ruby238:master
  $ 
  $ heroku config:set --remote dev DATABASE_CLEANER_ALLOW_REMOTE_DATABASE_URL=true
  $ heroku config:set --remote dev DATABASE_CLEANER_ALLOW_PRODUCTION=true
  $ heroku run bundle exec rake ptourist:reset_all --remote dev
  ```

*NOTE*:  I shared my free project/cluster between `heroku-16`, `dev`, and `staging` using a 
different database name at the end of the URL
