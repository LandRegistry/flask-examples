How to structure a Flask app
=================

### Notes to self and others
Tests? How about package called tests. Use pytest  and test_requirements.txt? [pytest](http://pytest.org/latest/contents.html)
or python -m unittest discover?

Don't know, discussion please.

### This example app has the following structure

<pre>
flask-example
+-- Procfile (can be used to run locally and is used on Heroku)
+-- README.md
+-- appname (the actual application)
|       +-- __init__.py  (do your application setup here)
|       +-- models.py (you could if you like keep persitent domain classes here)
|       +-- server.py (this is where your route decorators live. You don't have to call it server, but it's a good a name
|       +-- static (static assets folder)
|       +-- templates (jinja templates)
+-- config.py
+-- manage.py (flask script commands)
+-- migrations (this app has a database hence migration files are a fine idea - this contains an example - delete if you are not using a db)
|        +-- README
|        +-- alembic.ini
|        +-- env.py
|        +-- script.py.mako
|        +-- versions
|        |  +-- 32c89b1892d9_.py
+-- requirements.txt (requirements for this project)
+-- run.sh (this just calls foreman start)
+-- run_dev.py (this is handy for local development as you get reload on file changes so you don't have to start/stop for changes)

</pre>

### To get and play with the example code

**Prerequisites**

1. install [virtualenv](https://virtualenv.pypa.io/en/latest)

2. install [virtualenvwrapper](http://virtualenvwrapper.readthedocs.org/en/latest/)

Note very important the part in virtualenvwrapper install intructions about sourcing the virtualenvwrapper.sh in your .bash_profile, .zshrc or whatever for the shell you use.

On my machine I have the following in my .zshrc

```
export WORKON_HOME=~/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```

So when I call ```mkvirtualenv some-name``` I get a some-name virtual environment directory in ~/.virtualenvs

Now create a virtualenv for the flask-example project

```
mkvirtualenv flask-examples
```

This automatically activates the virtualenv. Once done any pip installs will install into that virtualenv.

Anytime you want to activate the virtualenv from that point on, you just enter

```
workon flask-example
```

**Check the code out**

```
git clone git@github.com:LandRegistry/flask-examples.git
cd flask-examples
workon flask-examples
pip install -r requirements.txt
```

**other requirements**

sub module dependencies - gov uk toolkit

```
git submodule init
git submodule update
```

bundler dependency - css generation from scss

```
gem install sass
```


**Run the app**

```
./run.sh # this runs with foreman
```
or

```
./run.sh dev # this runs the run_dev.py so reloading works
```


## Some suggestions/patterns contained in the example app

### A little bit of security

This is a starter for ten for things to add to all responses

```
@app.after_request
def after_request(response):
    response.headers.add('Content-Security-Policy', "default-src 'self'")
    response.headers.add('X-Frame-Options', 'deny')
    response.headers.add('X-Content-Type-Options', 'nosniff')
    response.headers.add('X-XSS-Protection', '1; mode=block')
    return response
```

Have a look at [List of useful HTTP headers](https://www.owasp.org/index.php/List_of_useful_HTTP_headers)

Any further suggestions welcome

### Use a package for the app

This allows you to initialise the application in the __init__.py  of the appname package.

### Configuration

Create a config.py with a base Config class  and a sub class per environment that  needs over riding of values, or additional values.

Values for configuration keys should be read from the environment. In other words set environment variables for local development and on any deployment target.

For example, for local development

```
export SETTINGS='config.Config'
```

or on Heroku:

Install [Heroku Toolbelt](https://toolbelt.heroku.com/) , then you can set the config to be used as follows.

```
heroku config:set SETTINGS=config.Config
```


### To run the application

The app should have a procfile Procfile which in this app looks like this:

```
web: gunicorn -b 0.0.0.0:$PORT -k eventlet appname.server:app
```

There may be a higher level script to run a number of services. In that case create a run.sh and add that to project run_all.sh

### Databases (NOTE: If you are not using a database - uninstall all the flask-sqlalchemy, flask-script stuff and delete the migrations directory - you will not need it!)

If the app connects to a database then we'll need a means to create and manage db schemas. The suggested approach would be to add Flask-Migrate and Flask-Script to the project requirements.txt.

Flask-Migrate will pull in Flask-SQLAlchemy and Alembic (which manages the actual migration). Flask-Script provides a nice wrapper for constructing commands.

This example project has a simple model in models.py that will be the basis of generated migration files.  Have a look at manage.py as well which imports the models and is configured to use the migration commands. Note because you need to export a SETTINGS value, the commands can be run against different databases dependent on environment.

On first installation (a one time operation), you would run

```
python manage.py db init
```

This creates a migration directory with files needed for Alembic

You can then run

```
python manage.py db migrate
```

That will dump a file into the versions directory in migrations. Remember, that in manage.py script there's an import for all the models in models.py. That's how the tool knows what models to create DDL for.

The file created in versions contains the python code needed to create tables based upon your models.

Below is a snippet from the example migration script in project.

```
def upgrade():
    ### commands auto generated by Alembic - please adjust! ###
    op.create_table('foo',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('fooname', sa.String(length=140), nullable=True),
    sa.PrimaryKeyConstraint('id'),
    sa.UniqueConstraint('fooname')
    )
```

To apply the change to your local db (be that sqlite or postgres or whatever you've configured in the Config class you have set your SETTINGS environment variable) then run.

```
python manage.py db upgrade
```

To run the migration on heroku you just need to have set the correct configuration class using heroku config:set as mentioned above. Push the repo to Heroku
and then use:

```
heroku run ...
```

to run the commands as above.


### Frontend and static assets

**Install the sass gem**

```
gem install sass
```

#### Pre-compiled

Compile the SASS locally with sass:

    cd appname/static/stylesheets
    sass main.scss main.css

...and then check in the changed file.

GOV UK frontend toolkit is a git submodule in this project. After checking out:

```
git submodule init
git submodule update
```
