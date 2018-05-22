# The Flask Mega-Tutorial

This repo covers Miguel Grinberg's _Flask Mega-Tutorial_. I am adding my personal notes as I'm going through it—how I understand things.

## Microblog

Create directory, move to directory.

Create the virtual environment:

`python3 -m venv venv`

And activate it:

`source venv/bin/activate`

Install Flask:

`pip install flask`

The application will exist in a package. In Python, a sub-directory that includes a __init__.py file is considered a package, and can be imported.

Create app folder:

`mkdir app`

Create __init__.py file:

```
from flask import Flask

app = Flask(__name__)

from app import routes
```

Flask needs to be told how to import it, by setting the FLASK_APP environment variable:

`export FLASK_APP=microblog.py`

To start the app, execute `flask run`. Make sure the be in the virtual environment; start it with `source venv/bin/activate`.


## Chapter 2: Templates

Templates are stored under `app/templates/` as html files. Templates are returned by functions in `routes.py`.

Dynamic content in templates is enclosed in `{{ ... }}`.

The templates are rendered in `routes.py` by the `render_template()` function. This funtion accepts arguments—the dynamic content mentioned above. Don't forget to import the function!

`from flask import render_template`

The templates are rendered by Jinja2, the template engine for Python that comes with Flask.

### Conditional statements

Jinja2 supports conditional statements so we can do more interesting stuff such as:

```
{% if title %}
<title>{{ title }} - Microblog</title>
{% else %}
<titile>Welcome to Microblog</title>
{% endif %}
```


### Loops

Let's use some more dummy data to loop through posts:

```
posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Portland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The Avengers movie was so cool!'
        }
    ]
```

### Template inheritance

I'm very familiar with this by now. I liked the title logic in the `<head>`:

```
<head>
    {% if title %}
    <title>{{ title }} - Microblog</title>
    {% else %}
    <title>Welcome to Microblog</title>
    {% endif %}
</head>
```

## Chapter 3: Web forms

The tutorial uses "the Flask-WTF extension, which is a thin wrapper around the WTForms package that nicely integrates it with Flask."

In the venv:

`pip install flask-wtf`

### Configuration

Miguel Grinberg uses a class to store configuration variables. A bit unusual (?) but very extensible. The class lives in `config.py` in the top-level directory. The configuration settings are defined as class variables inside the `Config` class.

We need to tell Flask to read the config and apply it; in `__init__.py`:

`from config import Config`

The lowercase “config” is the name of the Python module config.py, the one with the uppercase “C” is the actual class.

Still in `__init__.py`:

`app.config.from_object(Config)`

### User login form

> The Flask-WTF extension uses Python classes to represent web forms. A form class simply defines the fields of the form as class variables.

The `forms.py` module will store the web form(s) classes.

We import `wtf_flask` and the type fields and validators. More info on [the field types that WTForms offers.](http://wtforms.simplecodes.com/docs/0.6/fields.html#basic-fields0)

The form module is created, Flask-WTF is imported, the form class is created and each field is declared as a class variable. Cool. Now we need to build a new template.

>The form.hidden_tag() template argument generates a hidden field that includes a token that is used to protect the form against CSRF attacks. All you need to do to have the form protected is include this hidden field and have the SECRET_KEY variable defined in the Flask configuration. If you take care of these two things, Flask-WTF does the rest for you.

The `form.submit()` syntax is convenient when one wants to attach CSS classes or IDs to form fiels:

`{{ form.submit(class="btn-primary") }}`

The last steps are to:
+ import the form to `routes.py`: `from app.forms import LoginForm`
+ add an `@app.route` to `routes.py`

>The form=form syntax may look odd, but is simply passing the form object created in the line above (and shown on the right side) to the template with the name form (shown on the left).

Right now we're going to rely on flashed messages to show the form interaction:

>When you call the `flash()` function, Flask stores the message, but flashed messages will not magically appear in web pages. The templates of the application need to render these flashed messages in a way that works for the site layout.

>An interesting property of these flashed messages is that once they are requested once through the `get_flashed_messages` function they are removed from the message list, so they appear only once after the `flash()`` function is called.

>The form.validate_on_submit() method does all the form processing work.

#### Improving form validation

>As a general rule, any fields that have validators attached will have any error messages that result from validation added under `form.<field_name>.errors`. This is going to be a list, because fields can have multiple validators attached and more than one may be providing error messages to display to the user.

Lalith Polepeddi implements form validation in Flask a bit differently. The `{% for errors in form.<field_name>.errors %}` is nested in an `{% if form.<field_name>.errors %}` which feels a bit cleaner to me. Polepeddi also relies on an `error-message` CSS class instead of inlining some style.

#### Generating links

Instead of relying on actual links in templates and source files, it is better to rely on Flask's `url_for()` function. So instead of:

```
<a href="/index">Home</a>
<a href="/login">Login</a>
```

we'll write:

```
<a href="{{ url_for('index') }}">Home</a>
<a href="{{ url_for('login') }}">Login</a>
```

## Chapter 4: Database

We'll be using Flask-SQLAlchemy and Flask-Migrate (built by Grinberg himself.)

```
(venv) $ pip install flask-sqlalchemy
(venv) $ pip install flask-migrate
```

We'll be using SQLite during development. Thanks to Flask-Migrate + Flask-Alchemy, we should be able to migrate to MySQL or PostgreSQL without having to change the app.

### Migration repository

> Alembic (the migration framework used by Flask-Migrate) maintains a migration repository, which is a directory in which it stores its migration scripts. Each time a change is made to the database schema, a migration script is added to the repository with the details of the change. To apply the migrations to a database, these migration scripts are executed in the sequence they were created.

Let's run:

`flask db init`

The new repository should be added to version control—it's an integral part of the project.

To generate the automatic migrations, run:

`flask db migrate -m "users table"`

A migration script was generated. In my case, it is called `1cc24ddbf5ec_users_table.py` under `migrations/versions`.

>The flask db migrate command does not make any changes to the database, it just generates the migration script. To apply the changes to the database, the flask db upgrade command must be used.

`flask db upgrade`

The upgrade command will detect that we're building with SQLite and will create and app file: `app.db`.

>When working with database servers such as MySQL and PostgreSQL, you have to create the database in the database server _before_ running `upgrade`.

We are creating a second `posts` table with a post `id`, `body`, `timestamp`, and `user_id`, the _foreign key_ that will connect the two tables in a one-to-many relationship.

I like what the author is doing with the `timestamp`, calling the `DateTime` module and setting the default as follows:

`default=datetime.utcnow`

Interesting note about timestamps:

> In general, you will want to work with UTC dates and times in a server application. This ensures that you are using uniform timestamps regardless of where the users are located. These timestamps will be converted to the user’s local time when they are displayed.

We've made changes to the schema, so let's trigger a new update to the database migration script:

`flask db migrate -m "posts table"`

### Shell context

> The `shell` command is the second “core” command implemented by Flask, after `run`.

More about [the Flask CLI.](http://flask.pocoo.org/docs/1.0/cli/#cli)


## Chapter 5: User logins

### Password hashing

We use the password hashing implemented in [Werkzeug](http://werkzeug.pocoo.org/) (part of Flask installation).

We compare a password hash previously generated and the one generated on the fly when the user enters their password. The function returns a boolean.










