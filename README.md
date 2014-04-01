Air Mozilla
===========

Live: [https://air.mozilla.org](https://air.mozilla.org)
Stage: [https://air.allizom.org](https://air.allizom.org)
Dev: [https://air-dev.allizom.org](https://air-dev.allizom.org)

Air Mozilla is the Internet multimedia presence of Mozilla, with live and
pre-recorded shows, interviews, news snippets, tutorial videos, and
features about the Mozilla community.

Wiki page: https://wiki.mozilla.org/Air_Mozilla

How to get it running locally from scratch
------------------------------------------

This section assumes you know about and are using a
[virtualenv](http://www.virtualenv.org/).
If you're not familiar with `virtualenv`, that's fine. You can use your "system python"
but a virtualenv is advantageous because you get a self-contained python system
that doesn't affect and isn't affected by any other python projects on your
computer.

Also, these instructions are **geared towards developers** working on the code.
Not system people deploying the code for production.

**Step 1 - The stuff you need**

You're going to need Git, Python 2.6 or Python 2.7, a PostgreSQL database
(partially works with MySQL and SQLite too) and the necessary python dev
libraries so you can install binary python packages. On a mac, that means you
need to install XCode and on Linux you'll need to install `python-dev`.

[This article on pyladies.com](http://www.pyladies.com/blog/Get-Your-Mac-Ready-for-Python-Programming/)
has a lot of useful information.

**Step 2 - Get the code**

Note: We're assuming you have already activated a `virtualenv` which will
have its own `pip`.

```
git clone --recursive https://github.com/mozilla/airmozilla.git
cd airmozilla
pip install -r requirements/compiled.txt
```

**Step 3 - Create a database**
First you need to install PostgreSQL on your computer. Please follow the
related instructions for your Operating System on
the PostgreSQL website](http://www.postgresql.org/download/).

To create a database in PostgreSQL there are different approaches. The simplest
is the `createdb` command. How you handle credentials, roles and permissions is
generally out of scope for this tutorial.

```
createdb -E UTF8 airmozilla
```

You might at that point need to supply a specific username and/or password.
Whichever it is, take a note of it because you'll need it to set up your settings.

**Step 4 - Set up the settings**

The first thing to do is to copy the settings "template".

```
cp airmozilla/settings/local.py-dist airmozilla/settings/local.py
```
Now open that `airmozilla/settings/local.py` in your editor and we'll go
through some essential bits.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'airmozilla',
        'USER': 'username',
        'PASSWORD': 'password',
        'HOST': '',
        'PORT': '',
    },
    # 'slave': {
    #     ...
    # },
}
```
Here, replace `username` and `password` accordingly.
If you want to use MySQL, which should work except the Search, you replace
the `ENGINE` setting with `django.db.backends.mysql`.

If you're a developer and intend to run tests make sure you have this line uncommented:

```
INSTALLED_APPS = base.INSTALLED_APPS + ['django_nose']
```
For local development, make sure the following lines are uncommented:
```
DEBUG = TEMPLATE_DEBUG = True
```
Since Air Mozilla uses a lot of AJAX, it's sometimes useful to not
let the browser show the errors, when they happen, but instead have
all errors happen in the terminal console. This is a matter of personal
taste but if you want all errors in the terminal add:
```
DEBUG_PROPAGATE_EXCEPTIONS = True
```

And, since you're probably going to run the local server NOT on HTTPS
you uncomment this line:
```
SESSION_COOKIE_SECURE = False
```
For security, you need to enter something into the `HMAC_KEYS`.
The same is true for `SECRET_KEY`.
```
HMAC_KEYS = {
    'any': 'thing',
}

SECRET_KEY = 'somethingnotempty'
```

By default, you need a `Memcache` server up and running. The connection
settings for that is not entered by default. So if you have a Memcache running
on the default port you need to enter it for the `LOCATION` setting like this:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'KEY_PREFIX': 'airmoz',
        'TIMEOUT': 6 * 60 * 60,
        'LOCATION': 'localhost:11211'
    }
}
```

If you want to use a local in-memory cache instead
by setting up the `CACHES` setting like this:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake'
    }
}
```
By default, all the default settings are geared towards production
deployment. Not local development. For example, the default way of handling
emails is to actually send them with SMTP. For local development we don't want
this. So change `EMAIL_BACKEND` to:
```
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

The majority of the remaining settings are important security details for
third-party services like AWS, Vid.ly, Mozillians, Twitter, Bit.ly and
Sentry.

Getting actual details there is a matter of trust and your relationship to the
project. Some things will obviously not work without these secrets, the file
upload for example won't work without AWS credentials. But things aren't
necessary should be something you can just go around. If something doesn't work
at all without having certain security settings, it's considered a bug.

**Step 5 - Running for the first time**

First you need to install all the required packages using this command:
```pip install -r requirements/compiled.txt```

After that, you need to run these commands: 

```
./manage.py syncdb
./manage.py migrate
```
The `syncdb` command will ask you to set up a first default superuser. Make
sure you use an email address that you can log in to Persona with.

And last but not least:
```
./manage.py runserver
```

Now you should be able to open `http://localhost:8000`.


Getting help
------------

The best place to get help on development is to go the the `#airmozilla-dev`
IRC channel on `irc.mozilla.org`.


Tests and test coverage
-----------------------
Included is a set of comprehensive tests, which you can run by:
``./manage.py test``

To see the tests' code coverage, use:
``./manage.py test --with-coverage --cover-erase --cover-html --cover-package=airmozilla``


Migrations
----------
We're using [South][south] to handle database migrations.
To generate a schema migration, make changes to models.py, then run:

``./manage.py schemamigration airmozilla.main --auto``

or

``./manage.py schemamigration airmozilla.comments --auto``

To generate a blank data migration, use:

``./manage.py datamigration airmozilla.main data_migration_name``

Then fill in the generated file with logic, fixtures, etc.

To apply migrations:

``./manage.py migrate airmozilla.main airmozilla.comments airmozilla.uploads``

In each command, replace airmozilla.main with the appropriate app.


Requirements
------------
See the ``requirements/`` directory for installation dependencies.
This app requires a working install of PIL with libjpeg and libpng.


First run
-----------------------
```
./manage.py syncdb
./manage.py migrate
```

If you'd like to create a default set of example groups with useful permissions
(Event Organizers, Experienced Event Organizers, PR, Producer):

``./manage.py create_mozilla_groups``

Since we're using BrowserID for log-in, you'll need to manually set up your
account as a superuser.  Log in to the site, then run the shell command:
```
./manage.py shell
>>> from django.contrib.auth.models import User
>>> my_user = User.objects.get(email='my@email.com')
>>> my_user.is_superuser = True
>>> my_user.is_staff = True
>>> my_user.save()
```


Contributing
------------

See CONTRIBUTING.md


IRC
---
irc://irc.mozilla.org/airmozilla-dev

[django]: http://www.djangoproject.com/
[gh-playdoh]: https://github.com/mozilla/playdoh
[south]: http://south.aeracode.org/


Cron jobs
---------

All cron jobs are managed by two files: First
``airmozilla/manage/crons.py`` where you kick things off. Then, later
the crontab file is compiled and installed by
``bin/crontab/crontab.tpl``. These are the two files you need to edit
to change, add or remove a cron execution.


Twitter
-------

To test tweeting locally, what you need to do is set up some fake
authentication credentials and lastly enable a debugging backend. So,
add this to your settings/local.py:

    TWITTER_USERNAME = 'airmozilla_dev'
    TWITTER_CONSUMER_KEY = 'something'
    TWITTER_CONSUMER_SECRET = 'something'
    TWITTER_ACCESS_TOKEN = 'something'
    TWITTER_ACCESS_TOKEN_SECRET = 'something'

Now, to avoid actually using HTTP to post this message to
api.twitter.com instead add this to your settings/local.py:

    TWEETER_BACKEND = 'airmozilla.manage.tweeter.ConsoleTweeter'

That means that all tweets will be sent to stdout instead of actually
being attempted.

To send unsent tweets, you need to call:

    ./manage.py cron send_unsent_tweets

This can be called again and again and internally it will take care of
not sending the same tweet twice.

If errors occur when trying to send, the tweet will be re-attempted
till the error finally goes away.


Deployment
----------

Deployment of dev, stage and prod is all done using Chief. More will
be written about it soon.


Bit.ly URL Shortener
--------------------

To generate a Bit.ly access token you need the right email address and
password. If you have access you can go to
https://bugzilla.mozilla.org/show_bug.cgi?id=870385#c2

To generate it use this command:

    curl -u "EMAIL:PASSWORD" -X POST "https://api-ssl.bitly.com/oauth/access_token"

That will spit out a 40 character code which you set in
settings/local.py for the ``BITLY_ACCESS_TOKEN`` setting.


About the database
------------------

Even though we use the Django ORM which is database engine agnostic,
we have to have PostgreSQL because we rely on its ability to do full
text index searches.
