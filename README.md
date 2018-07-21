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
Values can be generated here: https://api.wordpress.org/secret-key/1.1/salt/. There were too many errors (around special characters) setting up the values via the CLI that I entered the key value pairs in Heroku itself (Project > Setting > Config Vars).

Once entered in Heroku, all config values can be listed using:
```
$ heroku config
```

### Sendgrid

This is an add-on with Heroku (credit card details required). The `starter` plan is free and should be adequate. Even if you don't think you'll be sending any emails, this is a good add-on to include so that emails can be sent to recover lost passwords.

Add it with:

```
$ heroku addons:create sendgrid:starter
```

This will automatically add  `SENDGRID_USERNAME` and `SENDGRID_PASSWORD` to the app configuration.

However, you will also need an API key and this has to be manually set up. Log into Sendgrid either through their website using the login and password in your config values, or via Heroku. Under Settings, create an API key and save this in Heroku under the key `SENDGRID_API_KEY`.

**Amendment to the cloned code**

Heroku/Sendgrid now requires authentication via the API key rather than username/password.

The cloned code checks first for the username and password config values. This no longer works and the code in `.\config\plugins\wordpress\wordpress-sendgrid.php` has been replaced with the following code (use the API Key `else` block)
```php
<?php
/**
 * Configuration - Plugin: Sendgrid
 * @url: https://wordpress.org/plugins/sendgrid-email-delivery-simplified/
 */
if (!empty(getenv('SENDGRID_API_KEY'))) {
    // Auth method ('apikey')
    define('SENDGRID_AUTH_METHOD', 'apikey');
    define('SENDGRID_API_KEY', getenv('SENDGRID_API_KEY'));
}
```

*having been locked out of my account, a frustrating hour was spent trying to work out why Sendgrid wasn't sending emails but instead saying the host had disabled mail()*