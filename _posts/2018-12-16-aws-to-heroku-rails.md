---
layout: post
title: Moving your Rails app from AWS EC2 to Heroku.
tags: [aws, rails, heroku]
bigimg: /img/posts/2018-12-16-aws-to-heroku-rails/bigimg.png
---

AWS offers free tier for one year only, after that you may pay or find an alternative. And it is better to find a alternative in case your app is a hobby app, and not for large scale or commercial use.

In this post, I write about how to move your rails application which uses Postgres, Puma and Nginx to Heroku. Apps on Heroku run using dyno hours. Heroku offers 550 dyno hours which are free for each month, after which they will charge you. In case your app is not used much you may consider this option. And if you have credit card you may use it to get an additional of 450 dyno hours giving you 1000 hours which are enough for hobby projects.

## Backing up Data from AWS

Before we close our AWS instance we should backup our data. We can use **pg_dump** command to do this. Follow the steps below -

1. ssh to your AWS EC2 instance - 
    ```
        $ ssh [user]@[public-ip or public-dns]
    ```
2. Then navigate to the folder where you want to create a backup.
3. Now use pg_dump command to create a backup
    ```
        $ pg_dump -U [user-name] -h localhost [database-name] >> [backup-file-name.sql]
    ```

If there is a prompt then give your database password. And you can now see you backup in the current directory.

## Getting your backup file from AWS to your local machine
We use scp command for this purpose -

```
    $ scp -i [EC2key.pem] [username]@[public-ip or public-dns]:[/[path-of-backup-file-on-EC2]/[backup-file-name.sql]] . 
```

Here - 
1. EC2key.pem is your PEM key. Which you get while creating the instance.
2. username is the username you ssh with
3. public-ip or public-dns is the IP or DNS alias of the instance
4. path-of-backup-file-on-EC2 is the location where the file is stored on EC2

This will copy the file into the current folder on the local machine.

## Creating an App on Heroku 

Follow the steps below - (In case your app needs storage you can use a S3 bucket for that and use **fog** gem to connect to cloud services) 


1. First clone you Rails app on you local machine. And navigate to it.
2. Now you need to login to your heroku account from command line using heroku command line toolbelt
    ```
        $ heroku login        
    ```
3. Then create a app on heroku using following command. The name given to the app will be random you can change it if you need.
    ```
        $ heroku apps:create
    ```
This will add a git remote to your app, and also give you a web link to your app. You can also open your app using command ```$ heroku open```. You can check remotes using ```$ git remote -v```
4. Since we were using postgres for AWS settings for postgres DB are already in database.yml file. But you don't need to bother about any changes in it as postresql on heroku will be managed by a heroku addon.
5. Now we deploy, by pushing our code onn heroku using 
    ```
        $ git push heroku master
    ```
    The above command will raise two warnings one of missing ruby version in your gemfile and the other of missing Procfile, which is needed to manage Puma. Next we fix these.
    
    > You may get a warning of security vulnerability in case you have done ***config.assets.compile = true***. You may overcome this issue by setting this to false in enviornments/production.rb or by upgrading sprockets gem to version 3.7.2 using ```bundle update sprockets```. After this update you need to push again as this changes Gemfile.lock file. For more information read [here](https://blog.heroku.com/rails-asset-pipeline-vulnerability).
    
6. We add the ruby version our app uses in the gemfile as - ```ruby "2.x.x"```
7. We create a Procfile on the root of our project folder. And add following into it - 
    ```
        web: bundle exec puma -t 5:5 -p ${PORT:-3000} -e ${RACK_ENV:-development}
    ```
8.  Make sure the gem "puma" is added in your gem file. Then we need to change the **puma.rb** file in our config folder which contained configurations for working on AWS. Replace the file with the configurations below -
    ```
        # Puma can serve each request in a thread from an internal thread pool.
        # The `threads` method setting takes two numbers a minimum and maximum.
        # Any libraries that use thread pools should be configured to match
        # the maximum value specified for Puma. Default is set to 5 threads for minimum
        # and maximum, this matches the default thread size of Active Record.
        #
        threads_count = ENV.fetch("RAILS_MAX_THREADS") { 5 }.to_i
        threads threads_count, threads_count
        
        # Specifies the `port` that Puma will listen on to receive requests, default is 3000.
        #
        port        ENV.fetch("PORT") { 3000 }
        
        # Specifies the `environment` that Puma will run in.
        #
        environment ENV.fetch("RAILS_ENV") { "development" }
        
        # Specifies the number of `workers` to boot in clustered mode.
        # Workers are forked webserver processes. If using threads and workers together
        # the concurrency of the application would be max `threads` * `workers`.
        # Workers do not work on JRuby or Windows (both of which do not support
        # processes).
        #
        # workers ENV.fetch("WEB_CONCURRENCY") { 2 }
        
        # Use the `preload_app!` method when specifying a `workers` number.
        # This directive tells Puma to first boot the application and load code
        # before forking the application. This takes advantage of Copy On Write
        # process behavior so workers use less memory. If you use this option
        # you need to make sure to reconnect any threads in the `on_worker_boot`
        # block.
        #
        # preload_app!
        
        # The code in the `on_worker_boot` will be called if you are using
        # clustered mode by specifying a number of `workers`. After each worker
        # process is booted this block will be run, if you are using `preload_app!`
        # option you will want to use this block to reconnect to any threads
        # or connections that may have been created at application boot, Ruby
        # cannot share connections between processes.
        #
        # on_worker_boot do
        #   ActiveRecord::Base.establish_connection if defined?(ActiveRecord)
        # end
        
        # Allow puma to be restarted by `rails restart` command.
        plugin :tmp_restart
    ```
9.  Then run the following commands to deploy again with the changes done -
    ```
        $ git add .
        $ git commit -m "Add ruby version, Procfile and settings for Puma"
        $ git push
        $ git push heroku master
    ```

Now your app is deployed but it still won't run as database is not configured yet. We do this next.

## Creating a Postgres DB on heroku

Since we want to restore the database backup created we do not run our migrations to create a new database on heroku. Instead we do the following -

1. Go to your app dashboard on heroku and add a new addon of **Heroku Postgres**(Choose the free plan). This will create a new postgres DB for your application.
2. Now we need to add our **backup-file.sql** to our heroku database which can be done using
    ```
        $ heroku pg:psql DATABASE_URL < [backup-file.sql]
    ```
DATABASE_URL is a constant containing URL of our heroku database.

## Managing mailer for Devise confirmation and recoverable

You need to add enviornment variables specifying the **EMAIL** and **PASSWORD** or other configurations for mailer, in the settings section of your app dashboard, in **Config Vars** as key value pairs.

In case using gmail add email and password of your gmail account. 

When using gmail we need to **Allow access to less secure apps** to make mailer settings working, otherwise you may get unauthorized error. In case you get a **Net::SMTPAuthentication** error [this](https://stackoverflow.com/questions/18124878/netsmtpauthenticationerror-when-sending-email-from-rails-app-on-staging-envir) can help.

*Feel free to provide any feedback or report any errors. I have created this post after deploying my own rails application and taking reference from the different sources and the links mentioned in the post.*

