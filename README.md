# MLTSHP

## Status

[![Build status](https://badge.buildkite.com/a86854c6272f21c9b46b8b6aafd3a4fb99bcfabe6e611bc370.svg)](https://buildkite.com/mltshp-inc/mltshp-web-service) [![Coverage Status](https://coveralls.io/repos/github/MLTSHP/mltshp/badge.svg?branch=master)](https://coveralls.io/github/MLTSHP/mltshp?branch=master)

## Development Environment

MLTSHP is a Dockerized application. This greatly simplifies running the
application for local development. We also deploy the application using
[Docker](https://www.docker.com/), so it makes for a more consistent
environment to build and test against.

With Docker and a git client installed on your computer, clone the MLTSHP
code from Github. If you intend to develop features for MLTSHP, you should
clone from your own fork of the code. Once you have a copy checked out
locally, use this command to create a `settings.py` and `celeryconfig.py`
file suitable for local development (edit these as needed, but the defaults
should be okay):

    $ make init-dev

You should be able to start the app itself using:

    $ make run

This will do a lot of things. But ultimately, you should end up with a
copy of MLTSHP running locally. It expects to be accessed via a hostname
of `mltshp.localhost` and `s.mltshp.localhost`. Add these entries to your `/etc/hosts`
file:

    127.0.0.1   mltshp.localhost s.mltshp.localhost

The web app itself runs on port 8000. You should be able to reach it via:

[http://mltshp.localhost:8000/](http://mltshp.localhost:8000/)

Subsequent invocations of `make run` should be faster, once you have
the dependency images downloaded.

You can login as the `admin` user using the password `password`. You
can also register new user accounts.

While running the service, you can open an editor to the git checkout and
make updates to the code. Changes should be reflected as you save your
files (no need to restart the service).

The MySQL instance that is launched will be accessible on 127.0.0.1:3306
if you want to look at the database directly (since this is using the
default MySQL port, you will probably need to shutdown any existing MySQL
server you may have running). The login for the database is `root` with
no password. If you want to mark any of your user accounts as paid users,
find them in the `user` table and set their `is_paid` value to `1` and
their `stripe_plan_id` column value to `mltshp-double`.

## Logs and Data

When you run the application, it launches it into a background process.
But if you want to watch the realtime logs emitted by each service,
just use this command:

    $ docker-compose logs -f

In addition to that, the web app produces some log files that are
captured under the "mounts/logs" folder of your git repository.
The directory structure looks like this:

    mounts/
        uploaded/
            (transient uploaded file storage)
        logs/
            access.log - nginx access log file
            error.log - nginx error log file
            main-8000.log - python app log file
            celeryd-01.log - celery worker log file
        fakes3/
            (local S3 storage)
        mysql/
            (mysql data files)

## Database Migrations

Occassionally, a database migration will need to be performed to
bring an older database schema up to date. This will be necessary
if you have a MySQL database you use locally for development and
testing and keep it versus using the `destroy` and `init-dev`
commands to make a new one. To update your database, just do this:

    $ make shell
    docker-shell$ cd /srv/mltshp.com/mltshp; python migrate.py

That should do it.

## Tests

With your copy of MLTSHP running, you may want to run unit tests.

Some tests rely on accessing AWS S3. We have a dummy S3 server in
place to mock those API requests. But it requires a license key
in order to use it. Visit [this page](https://supso.org/projects/fake-s3)
to obtain a license key. For individual developers and small organizations,
there is no cost. Add the following to a local `.env` file in the root
of the project:

    FAKES3_LICENSE_KEY=your-license-key-here

Some of the unit tests actually test against Twitter itself, so you'll
want to generate a custom Twitter app with your own set of keys.
Configure the `test_settings` in your `settings.py` file appropriately:

    "twitter_consumer_key" : "twitter_consumer_key_here",
    "twitter_consumer_secret" : "twitter_consumer_secret_key_here",
    "twitter_access_key" : "twitter_access_key_here",
    "twitter_access_secret" : "twitter_access_secret_here",

Then, just run:

    $ make test

Which will invoke a Docker process to run the unit test suite.

## Connecting to the MLTSHP shell

If you ever need to access the Docker image running the application,
you can use this command to create a shell (specifically, a shell
to the Python web application container):

    $ make shell

This should place you in the /srv/mltshp.com/mltshp directory as the
root user. You can use `apt-get` commands to install utilities you
may need.

## Cleanup

If you ever want to _wipe your local data_ and rebuild your Docker
containers, just use this command:

    $ make destroy

If you just wish to rebuild the Docker container, use the Docker
compose command:

    $ docker-compose down

Then, run another `make run`.

## Relationship with MLTSHP-Patterns

The CSS in this repo is just the compiled version of the styles from the MLTSHP
pattern library, which can be found in the
[mltshp-patterns](https://github.com/MLTSHP/mltshp-patterns) repo. Please do
not edit the CSS in this repo, since any changes will be lost the next time we
update from the pattern library.

## About

MLTSHP is open-source software, ©2017 the MLTSHP team and released to the public under the terms of the Mozilla Public License. A copy of the MPL can be found in the LICENSE file.

[![Fastly logo](/static/images/fastly-logo.png)](https://www.fastly.com) MLTSHP is proudly powered by Fastly.
