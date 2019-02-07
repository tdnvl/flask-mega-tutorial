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

The application will exist in a package. In Python, a sub-directory that includes a `__init__.py` file is considered a package, and can be imported.

Create app folder:

`mkdir app`

Create `__init__.py` file:

```
from flask import Flask

app = Flask(__name__)

from app import routes
```

Flask needs to be told how to import it, by setting the FLASK_APP environment variable:

`export FLASK_APP=microblog.py`

The python-dotenv package lets us register environment variables:

`pip install python-dotenv`

Create an `.flaskenv` file and declare the FLASK_APP environment variable:

`FLASK_APP=microblog.py`

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

We'll be using [Flask-Login](https://flask-login.readthedocs.io/en/latest/).

`pip install flask-login`

### Logging users in

We're querying the database to see if the user exists. I like the use of `first()`, here:

> The `first()` method is another commonly used way to execute a query, when you only need to have one result.

When the username and password are both correct:

> I call the `login_user()` function, which comes from Flask-Login. This function will register the user as logged in, so that means that any future pages the user navigates to will have the `current_user` variable set to that user.

### Requiring users to log in

Flask-Login offers a clever way to require users to log in while keeping in mind _where_ users were trying to log in (through a query string argument, `URL /login?next=/index` for example). Just add the `@login_required` decorator to a view function under the `@app.route` decorator(s):

```
from flask_login import login_required

@app.route('/')
@app.route('/index')
@login_required
def index():
    # ...
```

### Users registration

The registration form uses form validators. The password is required twice and on the second field, the author uses `EqualTo()` to make sure that the same password was entered twice.

The author added some custom validators (`validate_<field_name>`) to make sure that the `username` and `email` were unique in the database.

# Chapter 6: Profile Page and Avatars

## User profile page

`@app.route('/user/<username>')` is going to be for logged in users. We are passing a dynamic component in the URL: `<username>`.

The function `user()` queries the database to retrieve the user found in the URL. Instead of using `.first()` to return the first result, the author uses `first_or_404()` which returns the first result if there's one or a 404 if there isn't.

## Gravatar

> The Gravatar service is very simple to use. To request an image for a given user, a URL with the format https://www.gravatar.com/avatar/<hash>, where <hash> is the MD5 hash of the user’s email address.

We'll handle the md5 hashed by importing:

`from hashlib import md5`

in `models.py`.

> [...] because the MD5 support in Python works on bytes and not on strings, I encode the string as bytes before passing it on to the hash function.

So:

`digest = md5(self.email.lower().encode('utf-8')).hexdigest()`

## Using Jinja2 Sub-Templates

Sub-templates use a `_` prefix as a naming convention. They are, in a way, modules that one can invoke in a template.

The author decided to turn posts in the `user.html` templates into a sub-template. It is called this way:

```
{% for post in posts %}
    {% include '_post.html' %}
{% endfor %}
```

### More interesting profiles

The author decided to add two pieces of information on users profiles: an about me section and the last time seen.

We add two fields in the `User` class in `models.py`: `about_me` and `last-seen`, but we also have to generate a database migration:

`flask db migrate -m "new fields in user model"`

Flask-Migrate will automagically see that two columns need to be added to the database. The output looks like:

```
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added column 'user.about_me'
INFO  [alembic.autogenerate.compare] Detected added column 'user.last_seen'
```

Now to apply those changes, we must run:

`flask db upgrade`

Finally, we add these two fields to the user profile template:

#### Recording the last visit time for a user

This feature relies on a new decorator: `@before_request`. 

>  This is extremely useful because now I can insert code that I want to execute before any view function in the application, and I can have it in a single place.

If the user is authenticated, we pull the `datetime.utcnow()` time and commit it in the database under `last_seen`.

### Profile editor

We're adding a new form to let users enter some info about themselves that will be stored under `about_me`.

We start by writing a class for the form. We're importing the new `TextAreaField` field from `wtforms`, as well as the `Lenth` validator. We then add a new template (`edit_profile.html`) and a view function in `routes.py`.

Don't forget to import the form in `routes.py`!

`from app.forms import LoginForm, RegistrationForm, EditProfileForm`

# Chapter 7: Error handling

## Flask debug mode

To turn the debugger mode on, close the application and type:

`export FLASK_DEBUG=1`

> Never run a Flask application in debug mode on a production server!

>As an additional security measure, the debugger running in the browser starts locked, and on first use will ask for a PIN number, which you can see in the output of the `flask run` command.

On top of a more verbose, in the browser messages, the Flask Debug mode turns on the _reloader_; it will automatically restart the application when a source file is modified. It's really convenient compared to restarting the app manually when debugging.

## Custom error pages

Let's add an `errors.py` module and declare custom error handlers using the `@errorhandler` decorator.

For these error handlers to be registered with Flask, the new `app/errors.py` module needs to be imported after the application instance is created:

`from app import routes, models, errors` in `__init__.py`

## Sending errors by email

This obviously requires some server, email, and password settings. I don't want to go through this and I'll just use the debugger for now.

## Logging to a file

For this feature, the author relies on a RotatingFileHandler, a file-based log:

> The RotatingFileHandler class is nice because it rotates the logs, ensuring that the log files do not grow too large when the application runs for a long time. In this case I’m limiting the size of the log file to 10KB, and I’m keeping the last ten log files as backup.

# Chapter 8: Followers

# Chapter 9: Pagination

The author mentions the Post/Redirect/Get pattern: 

> It is a standard practice to respond to a `POST` request generated by a web form submission with a redirect. This helps mitigate an annoyance with how the refresh command is implemented in web browsers. 

## Explore page

The author adds a new Explore page that acts as a global stream of posts, to help users discover and follow other users. The author uses the same `index.html` template but omits the `form` argument in the `render_template()` call. As a result, the form will not display—only the feed of posts.

## Pagination of blog posts

Flask-SQLAlchemy supports pagination natively with the `paginate()` query method:

`>>> user.followed_posts().paginate(1, 20, False).items`

The arguments are:
+ page number starting form 1
+ number of items per page
+ an error flag (`True` will return a 404, `False`, an empty list)

The return value from `paginate` is a `Pagination` object.

## Page navigation

The author uses other attributes of the `Pagination` object to generate previous/next links. I like that he uses some logic to generate these URLs and passes them to the template through `render_template`.


# Chapter 10: Email support

## Flask-Mail and JSON Web Tokens

We installed the two modules.

> If you want to use an emulated email server [...] you can start in a second terminal with the following command: 

`(venv) $ python -m smtpd -n -c DebuggingServer localhost:8025`

> To configure for this server you will need to set two environment variables: 

```
(venv) $ export MAIL_SERVER=localhost
(venv) $ export MAIL_PORT=8025
```

## A simple email framework

### Password Reset Tokens

To authenticate the user trying to change their password, we are going to use a "very popular token standard for this type of process: the JSON Web Token, or JWT."

The author shows in the Shell how this works:

```
>>> import jwt
>>> token = jwt.encode({'a': 'b'}, 'my-secret', algorithm='HS256')
>>> token
b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhIjoiYiJ9.dvOo58OBDHiuSHD4uW88nfJik_sfUHq1mDi4G0'
>>> jwt.decode(token, 'my-secret', algorithms=['HS256'])
{'a': 'b'}
```

The token in itself is not secure. The hash goes both ways and it can be decoded easily (see the [JWT debugger](https://jwt.io/#debugger-io)). The key (we'll use `SECRET_KEY`) is... key.

> Since these tokens belong to users, I'm going to write the token generation and verification functions as methods in the User model

### Asynchronous Emails 

Running asynchronous tasks can be achieved with the `threading` or `multiprocessing` module.

# Chapter 11: Facelift

## Bootstrap for Flask

We'll use Bootstrap through a Flask extension:

`(venv) $ pip install flask-bootstrap`

> Flask-Bootstrap needs to be initialized like most other Flask extensions.

So we import and instantiate:

```
from flask_bootstrap import Bootstrap

...

bootstrap = Bootstrap(app)
```

The author then nests the `base.html` template into a `boostrap/base.html` (so the former extends the latter).

> The block named `content` is used by Flask-Bootstrap, so I renamed my content block as `app_content`

We had to do this for all templates.

# Chapter 12: Date and time

We're going to be looking into timestamps for posts. We'll rely on UTC but we're going to have to convert timezones. Fun 🎉

We'll use Flask-Moment, an extension written by the author and that makes it easy to use moment.js in Flask.

`(venv) $ pip install flask-moment`

We initialize in `__init__.py`:

```
# ...
from flask_moment import Moment

app = Flask(__name__)
# ...
moment = Moment(app)
```

The extension does not add the library to the pages so we're going to have to modify the base template ourselves, adding a block:

```
{% block scripts %}
    {{ super() }}
    {{ moment.include_moment() }}
{% endblock %}
```

`super()` is a way to _keep what is in the parent block without overriding it_. Base extends the Bootstrap base that already has some scripts. `super()` guarantees that these scripts are kept intact.

#  Chapter 13: I18n and L10n 

I18n and L10n are abbreviations for internationalization and localization. We are going to use [Flask-Babel](https://pythonhosted.org/Flask-Babel/) for that.

This is a pain. I just skipped this section. I hope that I won't have copy all the files to keep on moving...

# Chapter 14: Ajax

## Using a Third-Party Translation Service 

We're generating some Azure Translator Text keys and adding them as environmenrt variables and calling them in `config.py`:

>As always with configuration values, I prefer to install them in environment variables and import them into the Flask configuration from there. This is particularly important with sensitive information such as keys or passwords that enable access to third-party services. You definitely do not want to write those explicitly in the code. 

```
class Config(object):
    # ...
    MS_TRANSLATOR_KEY = os.environ.get('MS_TRANSLATOR_KEY')
```









