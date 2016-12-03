# Simple WordPress on App Engine Starter Project

***Note: This repo is a fork of the Google WordPress for AppEngine starter project that is way too painful for them to maintian. This project ensures you always have the lastest versions of plugins, and makes the entire process of updating and installing plugins for AppEngine painless. From your pals at Redux.  ;)

## Prerequisites

1. Install [PHP Composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx)
2. Install [Google's Cloud SDK (gcloud)](https://cloud.google.com/sdk/) command prompt utility.
3. Install the [PHP SDK for Google App Engine](https://developers.google.com/appengine/downloads#Google_App_Engine_SDK_for_PHP) 
```
$ gcloud components list
# Find the gcloud app PHP Extensions in the list
$ gcloud components install app-engine-php-*
``` 
4. Install [MySQL](http://dev.mysql.com/downloads/)

5. [Sign up](http://cloud.google.com/console) for a Google Cloud Platform project, and
set up a Cloud SQL instance, as described [here](https://cloud.google.com/sql/docs/getting-started#create), and a
Cloud Storage bucket, as described [here](https://developers.google.com/storage/docs/signup). You'll want to name
your Cloud SQL instance "wordpress" to match the config files provided here. To keep costs down, we suggest signing up for a D0 instance with package billing. 
6. Visit your project in the
[Google Cloud Console](http://cloud.google.com/console), going to the App Engine section's **Application Settings**
area, and make a note of the **Service Account Name** for your application, which has an e-mail address
(e.g. `<PROJECT_ID>@appspot.gserviceaccount.com`). Then, visit the Cloud Storage section of your project,
select the checkbox next to the bucket you created in step 3, click
**Bucket Permissions**, and add your Service Account Name as a **User** account that has **Writer** permission.

## Getting Started

### Step 1: Clone

Clone this git repo:
```
$ git clone https://github.com/reduxframework/appengine-wordpress
```

### Step 2: Run composer install


```
$ php composer install
```

This will grab all of the plugins/repos defined install them properly. It will also build your `public.built` directory.

**Note, if you are a PC/Windows user, you MUST use the following command, and only edit `composer.windows.json` to your needs.**

```
$ php composer require composer.win.json
```

### Step 2: Edit public/wp-config.php

Edit `public/wp-config.php` primarily the `DB_HOST` declaration for cloud-sql. Also be sure to update the section labeled `Authentication Unique Keys and Salts`.

## Running WordPress locally

>First, edit `public/wp-config.php` to have all the correct database login information.

**If you're using MAMP locally on MacOS, there are instructions in `public/wp-config.php`. Search for MAMP (two places).**

On Mac and Windows, the default is to use the PHP binaries bundled with the SDK:

    $ APP_ENGINE_SDK_PATH/dev_appserver.py path_to_this_directory

On Linux, or to use your own PHP binaries, use:

    $ APP_ENGINE_SDK_PATH/dev_appserver.py --php_executable_path=PHP_CGI_EXECUTABLE_PATH path_to_this_directory

Now, with App Engine running locally, visit `http://localhost:8080/wp-admin/install.php` in your browser and run
the setup process, changing the port number from 8080 if you aren't using the default.
Or, to install directly from the local root URL, define `WP_SITEURL` in your `wp-config.php`, e.g.:

    define( 'WP_SITEURL', 'http://localhost:8080/');

You should be able to log in, and confirm that your app is ready to deploy.

### Deploy!

If all looks good, you can upload your application by using this command:

    $ APP_ENGINE_SDK_PATH/appcfg.py update APPLICATION_DIRECTORY

Just like you had to do with the local database, you'll need to set up the Cloud SQL instance. The SDK includes
a tool for doing just that:

    google_sql.py <PROJECT_ID>:wordpress

This launches a browser window that authorizes the `google_sql.py` tool to connect to your Cloud SQL instance.
After clicking **Accept**, you can return to the command prompt, which has entered into the SQL command tool
and is now connected to your instance. Next to `sql>`, enter this command:

    CREATE DATABASE IF NOT EXISTS wordpress_db;

You should see that it inserted 1 row of data creating the database. You can now type `exit` -- we're done here.

Now, just like you did when WordPress was running locally, you'll need to run the install script by visiting:

    http://<PROJECT_ID>.appspot.com/wp-admin/install.php

Or, to install directly from the root URL, you can define WP_SITEURL in your `wp-config.php`, e.g.:

    define( 'WP_SITEURL', 'http://<YOUR_PROJECT_ID>.appspot.com/');

### Activating the plugins, configuring email, and hooking up WordPress to your Cloud Storage

**The following steps should be performed on your hosted copy of WordPress on App Engine**

#### Activating the plugins

Now, we just need to activate the plugins that were packaged with your app. Log into the WordPress
administration section of your blog at `http://<PROJECT_ID>.appspot.com/wp-admin`, and visit the
Plugins section. Click the links to activate **Batcache Manager** and **Google App Engine for WordPress**.

#### Configuring email and hooking WordPress up to your Cloud Storage

Now visit **Settings > App Engine**. Enable the App Engine mail service - this will use the App Engine Mail
API to send notifications from WordPress. Optionally, enter a valid e-mail address that mail should be sent
from (if you leave this blank, the plugin will determine a default address to use). The address of the account
you used to the create the Cloud Console project should work.

Stay on this page, because in order to be able to embed images and other multimedia in your WordPress content,
you need to enter the name of the Cloud Storage bucket you created when going through all the Prequisites earlier
under **Upload Settings**.

Hit **Save Changes** to commit everything.

## That's all! (PHEW)

Congratulations! You should now have a blog that loads rapidly, caches elegantly,
sends email properly, and can support adding images and other media to blog posts! Most importantly,
it will take advantage of Google's incredibly powerful infrastructure and scale gracefully to
accomodate traffic that is hitting your blog.

### Maintaining

You'll want to keep your local copy of the application handy because that's how you install other plugins and update
the ones that are packaged with this project. Due to the tight security of the
App Engine sandbox, you can't directly write to files in the application area -- they're static. That's
also why we hooked your uploads up to Cloud Storage. So, to install plugins, you log into the admin area
of your local WordPress instance, install or update any plugins you want there, and
redeploy. Then go into the admin area for your hosted WordPress instance to activate the plugins.

## Warning
Anything inside `public.built/` directory will be erased at every composer update. So I'd suggest updating your code inside the `public/` folder and run `composer update` to deploy the changes. 

## Updating WordPress, Plugins, and Themes
This is SUPER easy. This is it:
```
php composer update
```

## Installing a Private Git Repo
Inside your composer.json file, modify the following:
```
    "repositories": [
        {
            "type":"composer",
            "url":"https://wpackagist.org"
        },
        {
            "type":"package",
            "package": {
                "name": "reduxframework/custom-repo",
                "version":"master",
                "source": {
                    "url": "https://user:pass@github.com/redux-framework/custom-repo.git",
                    "type": "git",
                    "reference": "master"
                }
            }
        }
    ],
```