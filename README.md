# Coding Adventures

This repo is for a wordpress blog about coding located here - [https://coding-adventures.herokuapp.com](https://coding-adventures.herokuapp.com). I'll be writing about my journey in coding, useful tools, events and how-tos.  As I'm using the free sandbox tier on Heroku, the page takes a while to initially spin up.

It'll also be my first foray into using Wordpress.

## Setup

The setup was cloned/based on this repo - [https://github.com/PhilippHeuer/wordpress-heroku](https://github.com/PhilippHeuer/wordpress-heroku) with some minor adjustments.

The below steps are based on Windows 10 and for deployment to Heroku. I have yet to work on this locally so further steps for setting up for local development will be added in future.

### PHP and Composer

Download and unzip [PHP](https://windows.php.net/download/). Make sure the folder containing php.exe is added to the PATH environment variable.

Download and install [Composer](https://getcomposer.org/download/). Also make sure it's added to the PATH.

Composer requires PHP. It's not strictly necessary to setup Wordpress but is required if you want to make any changes (e.g. upgrade Wordpress version).

### Heroku

Setup an account on [Heroku](https://www.heroku.com/). I used the free tiers for Heroku and all add-ons. Install Heroku and [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

Create an application via the CLI with the app name (or don't include a name to get a random name). A warning will appear if that name is already taken.


```
$ heroku create example-app
Creating ⬢ example... done
https://example-app.herokuapp.com/ | https://git.heroku.com/example-app.git
```

The following config values need to be set:
```
AUTH_KEY='SECRET_VALUE'
SECURE_AUTH_KEY='SECRET_VALUE'
LOGGED_IN_KEY='SECRET_VALUE'
NONCE_KEY='SECRET_VALUE'
AUTH_SALT='SECRET_VALUE'
SECURE_AUTH_SALT='SECRET_VALUE'
LOGGED_IN_SALT='SECRET_VALUE'
NONCE_SALT='SECRET_VALUE'
```
Values can be generated here: https://api.wordpress.org/secret-key/1.1/salt/. There were too many errors setting up the values via the CLI that I entered the key value pairs in Heroku itself (Project > Setting > Config Vars). This was possibly because the values generated included a lot of special characters.

Once entered in Heroku, all config values can be listed using:
```
$ heroku config
```
