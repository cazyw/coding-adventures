# Coding Adventures

This repo is for a wordpress blog about coding located here - [https://coding-adventures.herokuapp.com](https://coding-adventures.herokuapp.com). I'll be writing about my journey in coding, useful tools, events and how-tos. As I'm using the free sandbox tier on Heroku, the page takes a while to initially spin up.

It'll also be my first foray into using Wordpress.

## Setup

The setup was cloned/based on this repo - [https://github.com/PhilippHeuer/wordpress-heroku](https://github.com/PhilippHeuer/wordpress-heroku) with some minor adjustments as detailed below.

The first step was to clone the repo.

The below steps are for Windows 10 and for deployment to Heroku. I have yet to work on this locally so further steps for setting up for local development will be added in future.

Any modifications made to the `wordpress-heroku` cloned repo are in this repo.

### PHP and Composer

#### Windows

Download and unzip [PHP](https://windows.php.net/download/). Make sure the folder containing php.exe is added to the PATH environment variable.

Download and install [Composer](https://getcomposer.org/download/). Also make sure it's added to the PATH. Composer requires PHP. It's not strictly necessary to setup Wordpress but is required if you want to make any changes (e.g. upgrade Wordpress version).

**Amendment to the cloned code**

I found I had issues with cropping images in Wordpress unless the `ext-gd` package (bundled with php) was also added.

Navigate to the PHP installation folder and open `php.ini` in an editor. Then uncomment the following line.

```ini
; From this
;extension=php_gd2.dll

; To this
extension=php_gd2.dll
```

Add the following to `composer.json`

```
"require": {
  "php": ">=7.0",
  "ext-gd": "*", <== line added
  ...

```

Then run composer update to update the `composer.lock` file.

```
$ composer install <== may need to run this if not previously run
$ composer update
```

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

### Sendgrid Mail

This is an add-on with Heroku (credit card details required). The `starter` plan is free and should be adequate. Even if you don't think you'll be sending any emails, this is a good add-on to include so that emails can be sent to recover lost passwords.

Add it with:

```
$ heroku addons:create sendgrid:starter
```

This will automatically add `SENDGRID_USERNAME` and `SENDGRID_PASSWORD` to the app configuration.

However, you will also need an API key and this has to be manually set up. Log into Sendgrid either through their website using the login and password in your config values, or via Heroku. Under Settings, create an API key and save this in Heroku under the key `SENDGRID_API_KEY`.

**Changes to the cloned code**

Heroku/Sendgrid now requires authentication via the API key rather than username/password.

The cloned code originally checked first for the username and password config values. This no longer works and the code in `.\config\plugins\wordpress\wordpress-sendgrid.php` has been replaced with the following code (uses the API Key `else` block)

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

_when setting up Wordpress, I forgot my password and was locked out of my account. A frustrating hour was spent trying to work out why Sendgrid wasn't sending emails (to reset the password) but instead saying the host had disabled mail()_

### Scheduling wp-cron

Wordpress runs wp-cron (scheduled tasks) every time a page loads. This can mean tasks are run too often (if a site has a lot of traffic) or too little (if a site gets no traffic). In order to disable the default settings and use a consistent schedule, use the heroku scheduler:

```
$ heroku addons:create scheduler:standard
$ heroku config:set DISABLE_WP_CRON='true'
$ heroku addons:open scheduler
```

With the following values:

```
Dyno Size = Free
Frequency = Daily
Command = bin/cron/wordpress.sh
```

### Jaws MySQL Database

Used MySQL via the JawsDB add-on. The kitefin plan is free.

```
$ heroku addons:create jawsdb:kitefin
```

### AWS S3 Storage

Because Heroku's filesystem is ephemeral, images and other media must be more permanently stored elsewhere, in this case, on Amazon S3.

1. Create an AWS account
1. Create an S3 bucket that will hold the media content
1. Create a user specifically to access the bucket

A user was created with `AmazonS3FullAccess` permission policy.

S3 Bucket policy used so only a particular user can access the bucket:

```json
# accountnumber = the root account number
# user = the user who will have access
# bucketname = the name of the S3 bucket

{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<accountnumber>:user/<user>"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::<bucketname>/*"
        }
    ]
}
```

Once the bucket and user has been set up, the following key value pair needs to be added to the Heroku app config vars:

```
# access key id = the user's access key (under Security Credentials)
# access secret key = the user's secret key (under Security Credentials)
# bucket region = region (e.g. ap-southeast-2)
# bucket name = the name of the S3 bucket

AWS_S3_URL=s3://<ACCESS_KEY_ID>:<ACCESS_SECRET_KEY>@s3-<BUCKET_REGION>.amazonaws.com/<BUCKET_NAME>
```

Region codes can be found here: https://docs.aws.amazon.com/general/latest/gr/rande.html

### Heroku Redis Caching

To use Heroku Redis for caching run the following command:

```
$ heroku addons:create heroku-redis:hobby-dev
```

### Deploying to Heroku

Add and commit the changes and then deploy to Heroku.

```
$ git push heroku master
```

Access the site with either the URL (e.g. `https://your-app.herokuapp.com`) or

```
$ heroku open
```

The first time you open the application you'll be prompted to setup a Wordpress account. You'll be able to log in and change your settings.

## Updating WordPress Version

The WordPress version in the cloned repo was 4.7.2 and at the time this project was set up, the Wordpress version was 4.9.8. A notification was visible in the admin tab that a new version was available. Currently upgraded to Wordpress version 5.3.2.

To upgrade to a new wordpress version:

In `.\composer.json`, change the wordpress version.

```json
"johnpbloch/wordpress": "5.3.2",
```

Run the following

```
$ composer update
```

And then commit the changes and push to Heroku.

```
$ git push heroku master
```

## Updating Themes

The cloned repo came with the Theme `Twenty Seventeen`. This was changed to `Dyad` and then `Baskerville` (the current theme). These are on the Official WordPress Themes site.

First find a theme on the Official [WordPress Themes](https://wordpress.com/themes) site. Then find it on the [Wordpress Packagist](https://wpackagist.org/s).

Add the package as a requirement to the `require` section in `composer.json`

```
"require": {
  "php": ">=7.0",
  ...
	"wpackagist-theme/dyad":"1.0.10",  <== line added
    "wpackagist-theme/baskerville":"1.26"   <== line added
  },

```

Then also add the theme name under the installer-paths section.

```
 "extra": {
   "installer-paths": {
      ...
      "web/app/themes/{$name}/": [
      "type:wordpress-theme",
      "wpackagist-theme/baskerville", <== line added
      "wpackagist-theme/dyad" <== line added
      ]
    },

```

Run the following:

```
$ composer update
```

Add and commit the changes and then deploy to Heroku.

```
$ git push heroku master
```

When you next log in, the theme will be listed in the Customization section and can be activated.
