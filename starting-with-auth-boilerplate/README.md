# ![Flask API - Hoot Back-End - Starting with Auth Boilerplate](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to bootstrap Flask applications with their JWT authentication starter code.

## Overview

Take a moment to review some of the starter code included in the [Flask JWT Auth Template](https://git.generalassemb.ly/modular-curriculum-all-courses/flask-jwt-auth-template).

In this lesson, we'll expand upon this code by adding modularity to accommodate additional functionality beyond user authentication.

## Modularizing our Flask application

Now that we're building a larger application with multiple resources and routes, let's start the process of modularizing our code by splitting off the authentication functionality into a separate file.

We'll leverage Flask's [blueprints pattern](https://flask.palletsprojects.com/en/2.3.x/blueprints/) and create a Blueprint for authentication routes.

> 💡 Blueprints in Flask are a way to organize your application into smaller and more manageable components. They allow you to split your application into distinct modules, each with its own routes, templates, and static files. This makes it easier to maintain and scale your codebase, especially as your application grows. By using Blueprints, you can keep related functionality together and encapsulate routes and views in a clean and modular way.

Let's create our `auth_blueprint.py` file.

Run the following command in your terminal:

```bash
touch auth_blueprint.py
```

Add the following to `auth_blueprint.py`:

```python 
from flask import Blueprint

authentication_blueprint = Blueprint('authentication_blueprint', __name__)
```

This provides us with a new **blueprint object** called `authentication_blueprint`. This object can be used much like the existing `app` object in `app.py`. 

Remember, we use decorators to mount routes to our `app` object. This allows us to define what code should execute when specific URLs are accessed. For example, the decorator below tells Flask to map the root URL `/` to the function that follows it, which in this case is the `index` function:

```python
@app.route('/')
def index():
  return "Hello, world!"
```

Similarly, **blueprint objects** can be used to define routes in the same way.  

Instead of using `@app.route` to define all routes in the application, we can utilize `@authentication_blueprint.route`. This allows us to keep authentication-related routes and logic encapsulated in a separate file. 

> 🏆  This organization helps maintain a clean code structure and makes it easier to manage and scale our application as it grows.

To plug it into the main application, we need to **register** the blueprint.

To register the blueprint, first add the following import to `app.py`:

```python
# Add to stack of imports
from auth_blueprint import authentication_blueprint
```

Next, add the following line below the initialization of `app`:

```python
app = Flask(__name__)
# New line:
app.register_blueprint(authentication_blueprint)
```

> 🤯 With that, we have registered our blueprint, and any routes defined within `authentication_blueprint` will now be part of our Flask application.

## Refactoring `app.py`

Now we need to move our existing auth routes out of `app.py` and into the new `auth_blueprint.py` (replacing all instances of `@app` with `@authentication_blueprint`).

We'll begin by cleaning up `app.py`. Here we'll want to remove the `signup`, `signin`, and `vip_lounge` routes and our `get_db_connection` function.

Our `app.py` should look like the following:

```python
from flask import Flask
from dotenv import load_dotenv
from auth_blueprint import authentication_blueprint

load_dotenv()

app = Flask(__name__)
app.register_blueprint(authentication_blueprint)

app.run()
```

## Adding a database connection module

Next we'll want to include our `get_db_connection` function in its own module. This approach will make sense for our application, as many pieces of our app will want this function.

First we'll create a `db_helpers.py` file.

Run the following command in your terminal:

```bash
touch db_helpers.py
```

Add the following to `db_helpers.py`:

```python
import os
import psycopg2

def get_db_connection():
    connection = psycopg2.connect(
        host='localhost',
        database=os.getenv('POSTGRES_DATABASE'),
        user=os.getenv('POSTGRES_USERNAME'),
        password=os.getenv('POSTGRES_PASSWORD')
    )
    return connection
```

With the module built out, we should now be able to implement it in `auth_blueprint.py`.

## Building the `auth_blueprint.py` blueprint

Add the following imports to the top of `auth_blueprint.py`,:

```python
import os
import jwt
import bcrypt
import psycopg2, psycopg2.extras
from flask import Blueprint, jsonify, request
from db_helpers import get_db_connection
```

> 💡 Notice the inclusion of our `get_db_connection`.

In `auth_blueprint.py`, find the initialization of the `authentication_blueprint`. We'll add our routes directly below it.

Add the following to `auth_blueprint.py`:

```python
authentication_blueprint = Blueprint('authentication_blueprint', __name__)

# Routes here:

@authentication_blueprint.route('/auth/signup', methods=['POST'])
def signup():
    try:
        new_user_data = request.get_json()
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM users WHERE username = %s;", (new_user_data["username"],))
        existing_user = cursor.fetchone()
        if existing_user:
            cursor.close()
            return jsonify({"error": "Username already taken"}), 400
        hashed_password = bcrypt.hashpw(bytes(new_user_data["password"], 'utf-8'), bcrypt.gensalt())        
        cursor.execute("INSERT INTO users (username, password) VALUES (%s, %s) RETURNING id, username", (new_user_data["username"], hashed_password.decode('utf-8')))
        created_user = cursor.fetchone()
        connection.commit()
        token = jwt.encode(created_user, os.getenv('JWT_SECRET'))
        return jsonify({"token": token, "user": created_user}), 201
    except Exception as error:
        return jsonify({"error": str(error)}), 401
    finally:
        connection.close()

@authentication_blueprint.route('/auth/signin', methods=["POST"])
def signin():
    try:
        sign_in_form_data = request.get_json()
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM users WHERE username = %s;", (sign_in_form_data["username"],))
        existing_user = cursor.fetchone()
        if existing_user is None:
            return jsonify({"error": "Invalid credentials."}), 401
        password_is_valid = bcrypt.checkpw(bytes(sign_in_form_data["password"], 'utf-8'), bytes(existing_user["password"], 'utf-8'))
        if not password_is_valid:
            return jsonify({"error": "Invalid credentials."}), 401
        token = jwt.encode({"username": existing_user["username"], "id": existing_user["id"]}, os.getenv('JWT_SECRET'))
        return jsonify({"token": token}), 201
    except Exception as error:
        return jsonify({"error": "Invalid credentials."}), 401
    finally:
        connection.close()  
```

You should now be able to test this out in Postman and receive valid responses when making requests to `http://127.0.0.1:5000/auth/signin` and `http://127.0.0.1:5000/auth/signup`.

> 💡 You might have some errors to work through after this much refactoring, so keep at it until your Postman requests are error-free, and we'll see you in the next step.