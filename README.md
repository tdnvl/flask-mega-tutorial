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


## Templates

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







