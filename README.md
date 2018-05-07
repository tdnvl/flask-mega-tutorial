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


## Chapter2: Templates

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





