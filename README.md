# Chicago Justice Project backend app

## Local Installation

### Install uv

This project uses [uv](https://docs.astral.sh/uv/).
Follow the [installation directions](https://docs.astral.sh/uv/getting-started/installation/) from uv's website.
If Python is already installed on your system uv will detect and use it. If not, uv will download Python automatically.

### Install PostgreSQL

#### macOS

The easiest way to install PostgreSQL for Mac is with a prebuilt Postgres
installation, like [Postgres.app](http://postgresapp.com/).

Alternatively, you may use [Homebrew](https://brew.sh/):

```shell
brew install postgresql
brew services start postgresql
```

#### GNU/Linux

The version of PostgreSQL provided in most distros' repositories should be
adequate and can be installed through your distro's package manager.

Ubuntu 16.04:

```shell
sudo apt-get update
sudo apt-get install postgresql
```

Arch Linux:

```shell
sudo pacman -S postgresql
sudo -u postgres initdb --locale $LANG -E UTF8 -D '/var/lib/postgres/data'
sudo systemctl start postgresql.service
```

### Configure the database

Once PostgreSQL is installed and running, you can create the database you'll
use locally for this app.

As a user with Postgres database privileges:

```shell
createdb cjpdb
```

The name of the database (e.g., `cjpdb`) may be anything you choose, but
keep track of what you name it along with the user and password we're about to
create. You'll need these for setting up your virtual environment.

Create the Postgres user and give it a password:

```shell
createuser --interactive --pwprompt
```

Finally, grant privileges on the database you just created to the user you just
created. For instance, if we created database `cjpdb` and the user `cjpuser`:

```shell
psql -d postgres -c "GRANT ALL ON DATABASE cjpdb TO cjpuser;"
```

### Configure the environment variables

Some of the app configuration is provided by environment variables.
These environment variables are specified in an `.env` file in the project's root directory.
This file is read by uv automatically.

An example `.env` file is provided. You should copy it:

```shell
cp .env-example .env
```

Then edit the file, substituting `cjpdb`, `cjpuser`, and `cjppassword` with the values appropriate for your setup.

If desired, you may generate a new secret key [here](http://www.miniwebtool.com/django-secret-key-generator/) or by running:

```shell
uv run python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### Create the Django admin user

To use the [Django admin site](https://docs.djangoproject.com/en/5.2/ref/contrib/admin/) you will need to create a superuser:

````shell
uv run manage.py createsuperuser
````

### Initialize Django models and start the server

```shell
uv run manage.py migrate
# the news_source fixture contains data already present in the migrations
# truncate to prevent duplicates in newsarticles_newsource
psql -U cjpuser -d cjpdb -c "TRUNCATE newsarticles_newssource CASCADE;"
uv run manage.py loaddata category news_source
uv run manage.py runserver
```

## Using the `article-tagging` Package Locally

For development purposes, changes to the [article-tagging](https://github.com/chicago-justice-project/article-tagging) package may be required.
This requires packaging the project locally to be used in this project:

````shell
uv pip install -e PATH_TO_ARTICLE_TAGGING_REPO
````

## Running News Scrapers

Before running scrapers for the first time be sure to download the [NLTK Data](https://www.nltk.org/data.html).

```shell
uv run manage.py downloadcorpus .venv/nltk_data
```

Then you can run all the scrapers with:

```shell
uv run manage.py runscrapers
```

or a single scraper by providing its name:

```shell
uv run manage.py runscrapers propublica-illinois
```

----

## Deployment

### CLI setup

The app runs on AWS Elastic Beanstalk. In order to manage the production app, a project maintainer must grant
you an AWS login and access key.

The Elastic Beanstalk CLI is separate from the main AWS CLI. Install it as described
[in the docs](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html).

The most reliable way to configure your credentials is to set the key ID and secret as environment
variables. If you use a different AWS account normally, you can create a file that sets the
envvars to the CJP account, and only source the file when working on the project.

Create a file `cjp-aws.env` with the following lines, or add them to your shell configuration.
**Make sure that these values don't get checked into version control!**

```
export AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

If you create a standalone file, you can enable the CJP credentials in your current
terminal session with `source cjp-aws.env`.

Test that you have the CLI configured correctly by running the following from the
`chicago-justice` project directory:

```
eb status
```

### Deploying

To deploy to production, run `eb deploy` from the project directory. It will deploy
whatever is on your local filesystem, even if it isn't checked into git. To maintain
consistency between production and git, it's recommended to merge changes to master
and then `git checkout master && git pull` before deploying.

Elastic Beanstalk will run any database migrations as part of the deployment. You can check on the
status of the deployment with `eb status`, or `eb logs` for the most recent logs from
various important logfiles.

Environment variables can also be configured with the CLI or from the AWS web interface.
