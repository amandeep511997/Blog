---
layout: post
title: Deploying Rails App on AWS Ubuntu-16.04 using Puma and Nginx.
tags: [aws, rails, puma, nginx, deployment]
bigimg: /img/posts/2017-12-28-aws-puma-nginx/rails.jpg
---

There are many ways to deploy your Rails application on AWS. In this post you can learn how to deploy rails application on AWS EC2 Ubuntu-16.04 using Puma as app server and Nginx as a web server. And we will use PostgreSQL as our production database.

> **It is considered a good practice to use same database for production and development environments**. As using different databases in different environments can be harmful due to the reason that there might be a case when your code will work in development environment but it may create a problem in production, creating a problem for you. So it would be better if you change your development and test database to PostgreSQL by installing PostgreSQL and adding pg gem, to your gemfile. 
PostgreSQL is considered a good choice due to its better performance than Sqlite3(which is a default database in Rails) in many ways, and it also offers some additional functionalities, such as full-text search.  

You may have your Application currently on Github, Bitbucket or any similar online version control platform. 

## Settings on AWS.

1. You need to have an AWS account(AWS offers 12 months free for new users). You need to have a credit/debit card for signing up on AWS. You may also have a look at [Github Student pack](https://education.github.com/pack)(if you are a student), which offers some good plans, promotional codes, and many other things till you are a student, one of which includes AWS also. 

2. Next you may sign in as a normal user or If you want you can setup an IAM user. [This](https://www.youtube.com/watch?v=yJivoQMkdug) video is helpful - if you want to setup an IAM user. And to know more about IAM user you may read [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)

3. Now create a Key Pair and a Security Group. [This](https://www.youtube.com/watch?v=_7tVUHlbPBM) video is helpful - Follow all steps for setting .pem file for key pair and creating a group.  

4. You may also change the region of your server, before creating an instance. 

## Creating a EC2 Instance.

1. Now, we will create an EC2 instance. Select EC2 from services option.

2. Then click on Launch Instance to setup configuration of your instance.

![aws-puma-nginx-1](/img/posts/2017-12-28-aws-puma-nginx/aws-puma-nginx-1.jpg)

3. Select the Ubuntu 16.04 LTS, from the free-tier eligible filter.

![aws-puma-nginx-2](/img/posts/2017-12-28-aws-puma-nginx/aws-puma-nginx-2.png)

4. Choose instance type to be t2.micro(the only type available in free-tier) and click on Review and Launch.

![aws-puma-nginx-3](/img/posts/2017-12-28-aws-puma-nginx/aws-puma-nginx-3.png)

5. Now, you will be redirected to the last option-7 of Review. You may select configuration in between if you want or you can skip them, and let them be as default setup. 

6. Then, click on Edit security group option. And select the security group you created above while Setting up AWS.

![aws-puma-nginx-4](/img/posts/2017-12-28-aws-puma-nginx/aws-puma-nginx-4.jpg)

7. Finally, Launch the instance. Here a prompt will ask you to choose a key pair, so choose the one you created above while Setting up AWS, and whose .pem file has been setup. And click Launch Instance.

**You may have more than one instance using the same key-pair and security group**

## Configuring our EC2 Instance.

Now we need to setup our new EC2 instance so it is ready for deployment. First we will setup ssh to make it easy for entering the server rather than entering passwords each time we want to enter.

### SSH into Server.

1. Copy the public DNS of the instance.

2. Use the **ssh** command to connect to instance. Now, we use **ubuntu** as the user to enter in server, by running following command.

*Your Terminal:*
```
	$ ssh -i ~/.ssh/name-of-key-pair.pem ubuntu@public-dns 
```

You can find the public-dns in description of your instance 

### Setting up a Deploy User.
Now we will set up a new user named **deploy**(You may use any name). We will set up our application as a deploy user not as root user, which is **ubuntu**.

*Server Terminal:*
1. Create a deploy user
```
	$ sudo adduser deploy
```

2. Then Enter a password. *It will be great if you write all the passwords at a safe place as there will be many passwords and there is a chance that you forget them*.

3. Enter your name but leave rest options as blank(just ENTER).

4. Enter "Y" that information entered by you is correct.

5. Add deploy user to the sudo group.
```
	$ sudo adduser deploy user
```

6. Add deploy user to the sudo group, by following command.
```
	$ sudo su - deploy
```

7. Create a .ssh directory.
```
	$ mkdir .ssh
```

8. change file permissions of .ssh directory to 700.
```
	$ chmod 700 .ssh
```

9. Create a file named *authorized_keys* in .ssh directory.
```
	$ touch .ssh/authorized_keys
```

	> The touch command is the easiest way to create new, empty files. It is also used to change the timestamps (i.e., dates and times of the most recent access and modification) on existing files and directories.

10. Change file permissions of the authorized_keys file to 600.
```
	$ chmod 600 .ssh/authorized_keys
```

11. Edit the authorized_keys file with nano(text-editor) and paste the public key of your local machine's key-pair in that file.
*(for generating new key pair on your machine and copying it for this step follow the next section, or else you can skip it)*
```
	$ nano ~/.ssh/authorized_keys
```

12. Then you can save and exit file by **Ctrl + X**, then **"Y"** and then **ENTER**.

13. Now exit the deploy user by **exit** command. And the exit the server again by **exit** command.

*For explanation of the file permissions types used here you may refer to following [link](http://www.thinkplexx.com/learn/article/unix/command/chmod-permissions-flags-explained-600-0600-700-777-100-etc)*.

### Check ssh key-pair in your machine and Copy it.

1. Open New Terminal.

2. Enter following command to see if existing SSH keys are present. They are usually id_rsa.pub or something similar.
```
	$ ls -al ~/.ssh
```

3. If they are present then go to step 5. Else follow the next step to create a new key.

4. Generating and Adding SSH keys - [Github](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

5. Copy the SSH key to your clipboard. *If your SSH key file has a different name than the example code, modify the filename to match your current setup. When copying your key, don't add any newlines or whitespace.*

6. You can copy they key using xclip as follows -
```
	# this downloads and installs xclip.
	$ sudo apt-get install xclip 
	# this copies your public key.
	$ xclip -sel clip < ~/.ssh/id_rsa.pub
```

7. Or you can copy using **cat** command.
```
	# shows the contents of the id_rsa.pub. Then you can copy that.
	$ cat ~/.ssh/id_rsa.pub
```

### Entering into Server as the deploy user
After adding a deploy user and setting up ssh, now we can enter into our server as deploy user using the following command.
```
	$ ssh deploy@[public-ip or public-dns]
```

Now, our deploy user has been setup, So next we will set up Rails environment on our server as deploy user.

### Installing Ruby and Rails

*Server Terminal:*
1. First install updates
```
	$ sudo apt-get update
```

2. We will install Ruby using rbenv. So, first install the rbenv and Ruby dependencies using the following command.
```
	$ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev
```

3. Now we are ready to install rbenv. The easiest way to do that is to run these commands, as the user that will be using Ruby.
```
	$ cd
	$ git clone git://github.com/sstephenson/rbenv.git .rbenv
	$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
	$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
	$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
	$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
	$ source ~/.bashrc
```
This installs rbenv into your home directory, and sets the appropriate environment variables that will allow rbenv to the active version of Ruby.

4. Now you can install Ruby version that you want to install using following commands. 
```
	$ rbenv install -v 2.x.x
	# the global command below sets the default version of Ruby that all of your shells will use. 
	$ rbenv global 2.x.x
```

5. You can now check if Ruby is installed properly by
```
	$ ruby -v
```

6. Now before installing rails disable the option to install documentations for each gem which you will install.
```
	$ echo "gem: --no-document" > ~/.gemrc
```

7. First install bundler gem to manage all the dependencies of your application.
```
	$ gem install bundler
```

8. Now you can install Rails and you may pass a specific version using -v option.
```
	$ gem install rails
```

9. Whenever you install a new version of Ruby or a gem that provides commands, you should run the **rehash** sub-command.
```
	$ rbenv rehash
```

10. You can now check if Rails is installed properly by
```
	$ rails -v
```

11. A few Rails features, such as the Asset Pipeline, depend on a Javascript runtime. We will install Node.js to provide this functionality.
```
	# Add the Node.js PPA to apt-get:
	$ sudo add-apt-repository ppa:chris-lea/node.js
	
	# Then update apt-get and install the Node.js package:
	$ sudo apt-get update
	$ sudo apt-get install nodejs
```

Now we have installed Ruby and Rails on our instance we will move further by installing PostgreSQL.

### Installing PostgreSQL

1. First, update apt-get.
```
	$ sudo apt-get update
```

2. Then install PostgreSQL and its development libraries.
```
	$ sudo apt-get install postgresql postgresql-contrib libpq-dev
```

### Creating Production database user

1. Create a new user for postgres and you can keep username as appname or you can keep of your choice.
```
	$ sudo -u postgres createuser -s appname
```

2. Now we will set the user's password. Enter the postgres console by the command
```
	$ sudo -u postgres psql
```

3. Then set the password for the database user, "appname" using following command
```
	\password appname
```
Enter your desired password and confirm it.

4. Exit the PostgreSQL console with this command
```
	\q
```

### Setting up your App for Production

Now we will set up some files in our application, to make them ready for production environment. And for safety we will keep secrets as environment variables.

1. Open your config/database.yml file, and then set up your production database setting as follows 
```
	production:
  		<<: *default
  		host: localhost
  		adapter: postgresql
  		encoding: utf8
  		database: appname_production
  		pool: 5
  		username: <%= ENV['APPNAME_DATABASE_USER'] %>
  		password: <%= ENV['APPNAME_DATABASE_PASSWORD'] %>
```

2. If you have setup devise for user authentication and the confirmable option is enabled then you should also set up mailer settings in config/environment/production.rb as follows
```
  	config.action_mailer.delivery_method = :smtp
  	config.action_mailer.default_url_options = { :host => 'your-domain-name' }
  	config.action_mailer.smtp_settings = {
    	address:              'smtp.gmail.com',
    	port:                 587,
    	domain:               'gmail.com',
    	user_name:            ENV['APPNAME_EMAIL_USER']
    	password:             ENV['APPNAME_EMAIL_PASSWORD'],
    	authentication:       'plain',
    	enable_starttls_auto: true  
    }
```
*The above mentioned settings are for using gmail for mailer. For any other mail services, such as Sengrid, you can make the appropriate changes. And while using gmail for mailer, you need to provide permission to less secure apps to access your account, which you can do from your gmail account.*

3. Make a commit and push changes to your application.

### Clone your application

1. Now you can do **git clone** to bring your application on the server from, Github, Bitbucket or any other online version control platform.
```
	$ git clone your-repo-url
```
By this you get your entire application on server along with its version history.

2. Now we run bundle to complete you app's dependencies
```
	$ cd appname
	$ bundle
```

### Setting up Environment variables

1. Install rbenv-vars Plugin it is an easy way to manage environment variables, which we can use to load passwords and secrets into our application at runtime. To install the rbenv-vars plugin
```
	# simply change to the .rbenv/plugins directory
	$ cd ~/.rbenv/plugins
	# and clone it from GitHub.
	$ git clone https://github.com/sstephenson/rbenv-vars.git
```
Now the rbenv-vars plugin is installed, we will set up the required environment variables, we used in our application. 

2. First, generate the secret key, which will be used to verify the integrity of signed cookies
```
	$ cd ~/appname
	$ rake secret
```

3. Copy the secret key that is generated, then open the .rbenv-vars file. We will place all our variables in this file, Any environment variables that you set here can be read by your Rails application.
```
	$ vi .rbenv-vars
```

4. First, set the SECRET_KEY_BASE variable like this
```
	SECRET_KEY_BASE=your_generated_secret
```

5. Then, set the APPNAME_DATABASE_USER and APPNAME_DATABASE_PASSWORD variable like this 
```
	APPNAME_DATABASE_USER=appname
	APPNAME_DATABASE_PASSWORD=prod_db_pass
```

6. If you have used devise then set the APPNAME_EMAIL_USER and APPNAME_EMAIL_PASSWORD variable like this 
```
	APPNAME_DATABASE_USER=used_email
	APPNAME_DATABASE_PASSWORD=used_email_pass
```

7. Save and exit.

In order to view which environment variables are set for your application, you can view with the rbenv-vars plugin by running this command
```
	$ rbenv vars
```

*If you change your secret or database password, update your .rbenv-vars file. Be careful to keep this file private, and don't include it any public code repositories.*

### Creating Production database

1. Now that your application is configured with your PostgreSQL database, create the production database by following command
```
	$ RAILS_ENV=production rake db:create
```

2. Set up your database
```
	$ RAILS_ENV=production rake db:migrate
```

3. You should also precompile the assets
```
	$ RAILS_ENV=production rake assets:precompile
```

### Installing Puma

On Release of Rails5 the app server has been changed from Webrick to Puma. So if your app is in Rails5 or you have puma already installed in your app you may skip this step.

To install Puma do the following-

1. Open your gemfile
```
	vi Gemfile
```

2. At the end of the file, add the Puma gem with this line
```
	gem 'puma'
```

3. Save and exit.

4. Run Bundler to install puma and its dependencies
```
	$ bundle
```

Puma is now installed, but we need to configure it.

### Configuring Puma

1. Before configuring Puma, you should look up the number of CPU cores your server has. You can do that with this command
```
	$ grep -c processor /proc/cpuinfo
```

2. We will add our Puma configuration to config/puma.rb
```
	$ vi config/puma.rb
```

3. Copy and paste this configuration into the file
```
	# Change to match your CPU core count
	workers 2

	# Min and Max threads per worker
	threads 1, 6

	app_dir = File.expand_path("../..", __FILE__)
	shared_dir = "#{app_dir}/shared"

	# Default to production
	rails_env = ENV['RAILS_ENV'] || "production"
	environment rails_env

	# Set up socket location
	bind "unix://#{shared_dir}/sockets/puma.sock"

	# Logging
	stdout_redirect "#{shared_dir}/log/puma.stdout.log", "#{shared_dir}/log/puma.stderr.log", true

	# Set master PID and state locations
	pidfile "#{shared_dir}/pids/puma.pid"
	state_path "#{shared_dir}/pids/puma.state"
	activate_control_app

	on_worker_boot do
	  require "active_record"
	  ActiveRecord::Base.connection.disconnect! rescue ActiveRecord::ConnectionNotEstablished
	  ActiveRecord::Base.establish_connection(YAML.load_file("#{app_dir}/config/database.yml")[rails_env])
	end
```
**Change the number of workers to the number of CPU cores of your server, which we found in step-1.**

4. Save and exit. This configures Puma with the location of your application, and the location of its socket, logs, and PIDs. 

5. Now create the directories that were referred to in the configuration file
```
	$ mkdir -p shared/pids shared/sockets shared/log
```

### Create Puma Upstart Script

Let's create an Upstart init script so we can easily start and stop Puma, and ensure that it will start on boot.

1. Download the Jungle Upstart tool from the Puma GitHub repository to your home directory
```
	$ cd ~
	$ wget https://raw.githubusercontent.com/puma/puma/master/tools/jungle/upstart/puma-manager.conf
	$ wget https://raw.githubusercontent.com/puma/puma/master/tools/jungle/upstart/puma.conf
```

2. Now open the provided puma.conf file, so we can configure the Puma deployment user
```
	$ vi puma.conf
```

3. Look for the two lines that specify **setuid** and **setgid**, and replace "apps" with the name of your deployment user and group. Here we have used **deploy**
```
	setuid deploy
	setgid deploy
```

4. Save and exit.

5. Now copy the scripts to the Upstart services directory
```
	$ sudo cp puma.conf puma-manager.conf /etc/init
```

6. The puma-manager.conf script references /etc/puma.conf for the applications that it should manage. Let's create and edit that inventory file now
```
	$ sudo vi /etc/puma.conf
```

7. Each line in this file should be the path to an application that you want puma-manager to manage. Add the path to your application now. For example:
```
	/home/deploy/appname
```

8. Save and exit. 

Now your application is configured to start at boot time, through Upstart. This means that your application will start even after your server is rebooted.

### Start Puma Applications Manually

1. To start all of your managed Puma apps now, run this command
```
	$ sudo start puma-manager
```

	>The following error **'Unable to connect to Upstart: Failed to connect to socket /com/ubuntu/upstart: Connection refused'** may occur while starting your puma-manager. You can either follow the solution given on [this](https://github.com/puma/puma/issues/1211) link, or you may run the commands below to solve this(In my case running these commands worked.)
	```
		$ sudo apt-get install upstart-sysv
		$ sudo update-initramfs -u
		$ reboot
	``` 
	The error above occurs if you are deploying your app on ubuntu 15.04 or higher as mentioned [here](http://codepany.com/blog/rails-5-puma-capistrano-nginx-jungle-upstart/). **Remember, It is necessary to reboot.**

2. You may also start a single Puma application by using the puma Upstart script, like this
```
	$ sudo start puma app=/home/deploy/appname
```

3. You may also use stop and restart to control the application, like so
```
	$ sudo stop puma-manager
	$ sudo restart puma-manager
```
Now your Rails application's production environment is running under Puma, and it's listening on the shared/sockets/puma.sock socket. Before your application will be accessible to an outside user, you must set up the Nginx reverse proxy.

### Install and Configure Nginx

1. Install Nginx using apt-get:
```
	$ sudo apt-get install nginx
```

2. Now open the default server block with a text editor.
```
	$ sudo vi /etc/nginx/sites-available/default
```
	Replace the contents of the file with the following code block. Be sure to replace the paths at two locations **server unix:** and **root** with your user(we have created here **deploy**) and your **appname**.
```
	upstream app {
	    # Path to Puma SOCK file, as defined previously
	    server unix:/home/deploy/appname/shared/sockets/puma.sock fail_timeout=0;
	}

	server {
	    listen 80;
	    server_name localhost;

	    root /home/deploy/appname/public;

	    try_files $uri/index.html $uri @app;

	    location @app {
	        proxy_pass http://app;
	        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header Host $http_host;
	        proxy_redirect off;
	    }

	    error_page 500 502 503 504 /500.html;
	    client_max_body_size 4G;
	    keepalive_timeout 10;
	}
```

3. Save and exit. This configures Nginx as a reverse proxy, so HTTP requests get forwarded to the Puma application server via a Unix socket. 

4. Restart Nginx to put the changes into effect, by the following command
```
	$ sudo service nginx restart
```

Now the production environment of your Rails application is accessible through your server's public IP address. To access your application visit your application server in a web browser:
```
	http://server_public_IP
```

### Conclusion
Congratulations! You have deployed the production environment of your Ruby on Rails application using Nginx and Puma.

Thanks for Reading!

*Feel free to provide any feedback or report any errors. I have created this post after deploying my own rails application and taking reference from the sources below and the links mentioned in the post.*

## References 

1. [Ruby Thursday](https://www.youtube.com/channel/UCgbzly83EZoSVjBIf9sNw5A) .
2. [SSH - Github](https://help.github.com/articles/connecting-to-github-with-ssh/).
3. [Digital Ocean - Deployment a Rails app on ubuntu 14.04 using puma and nginx](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-puma-and-nginx-on-ubuntu-14-04).
