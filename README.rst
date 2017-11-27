Sentry on Dokku
================

This repository is forked from `Sentry on Heroku`_ and adjusted to run on Dokku.

.. _Sentry on Heroku: https://github.com/fastmonkeys/sentry-on-heroku

    Sentry_ is a realtime event logging and aggregation platform.  At its core
    it specializes in monitoring errors and extracting all the information
    needed to do a proper post-mortem without any of the hassle of the
    standard user feedback loop.

    .. _Sentry: https://github.com/getsentry/sentry



Manual setup
------------

Follow the steps below to get Sentry up and running on Heroku:

1. Create a new Dokku application::

        dokku apps:create sentry

2. Add PostgresSQL to the application::

        sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
        dokku postgres:create sentry
        dokku postgres:link sentry sentry

3. Add Redis to the application::

        sudo dokku plugin:install https://github.com/dokku/dokku-redis.git redis
        dokku redis:create sentry
        dokku redis:link sentry sentry

4. Set Django's secret key for cryptographic signing and Sentry's shared secret
   for global administration privileges::

        dokku config:set --no-restart sentry SECRET_KEY=$(python -c "import base64, os; print(base64.b64encode(os.urandom(40)).decode())")

5. Set the absolute URL to the Sentry root directory. The URL should not include
   a trailing slash. Replace the URL below with your application's URL::

        dokku config:set --no-restart sentry SENTRY_URL_PREFIX=https://sentry.dokku.me

6. Deploy Sentry to Dokku::

        git push dokku master

7. Sentry's database migrations are automatically run as part of the Dokku post deploy::

        dokku run sentry "sentry --config=sentry.conf.py upgrade --noinput"

8. Create a user account for yourself::

        dokku run sentry "sentry --config=sentry.conf.py createuser"

That's it!



SSL Certificate
---------------

Follow the steps below, if you want to add a SSL certificate with Letsencrypt:

1. Install the letsencrypt plugin::

        sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git

2. Set the administrator email address::

        dokku config:set --no-restart sentry DOKKU_LETSENCRYPT_EMAIL=your@email.tld

3. Request a SSL certificate::

        dokku letsencrypt sentry
