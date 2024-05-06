# ![Flask API Related Data - Starting with Auth Boilerplate](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to bootstrap Flask applications with their authentication jwt starter code.

## Pulling in Authentication Boilerplate

Let's start by revisiting the JWT authentication Flask application in [this git repository](https://git.generalassemb.ly/modular-curriculum-all-courses/flask-jwt-auth-template).

We'll want to start our future authenticated Flask apps with basically the same code as this application, only a bit more modularized now that we're going to be adding more functionality than just user authentication.

For now, modify the `app.py` in your app to match the one provided in the template. You should have a `get_db_connection` function, `sign-up` and `sign-in` routes, and the `auth-middleware.py` file used previously.

Think: what else do we need to copy over to make this application run the same as our previous app?

<details>
    <summary>Reveal the next steps</summary>
    Create a .env file with the POSTGRES connection information and JWT_SECRET
    Create a new database for this project
    Create the users table
    Copy over the auth-middleware.py file
</details>

The .env file should be a simple copy-paste from the working version on your machine, but just in case you need it:

```
JWT_SECRET = <your-personal-secret-string>
POSTGRES_USERNAME = postgres
POSTGRES_PASSWORD = postgres
```

## Creating the SQL Database for this project

We'll need to execute the same Database and Table creation SQL commands as we did for the jwt authentication lesson:

```SQL
CREATE DATABASE flask_hoot;

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

Then, modify the connection code in `app.py` to connect to the `flask_hoot` database instead. Better yet, use an environment variable called `POSTGRES_DATABASE` and use it, instead.

```python
def get_db_connection():
    conn = psycopg2.connect(host='localhost',
                            database='os.getenv('POSTGRES_DATABASE')',
                            user=os.getenv('POSTGRES_USERNAME'),
                            password=os.getenv('POSTGRES_PASSWORD'))
    return conn
```

## Modularizing our Flask Application

Now that we're building a larger application that will have many routes, let's start the process of modularizing our code by splitting off the authentication functionality into a separate file.

We'll leverage Flask's [blueprints pattern](https://flask.palletsprojects.com/en/2.3.x/blueprints/) and create a Blueprint for authentication routes.

Create a new file called `auth_blueprint.py` and start with the minimum two lines required to import Blueprint and create one:

In `auth_blueprint.py`:

```python 
from flask import Blueprint

authentication_blueprint = Blueprint('authentication_blueprint', __name__)
```

This provides us a new object, `@authentication_blueprint`, that works exactly the same as `@app` did in our main file.

To plug it into the main application, we need to **register** the blueprint, like so:

In `app.py`:
```python
# Add to stack of imports
from auth_blueprint import authentication_blueprint

# register the blueprint 
app.register_blueprint(authentication_blueprint)
```

Now, we can copy-paste the two routes from our `app.py` into the new `auth_blueprint.py`, replacing all instances of `@app` with `@authentication_blueprint`. There's also a lot of imports to bring over from `app.py` as well!

In `application_blueprint.py`:
```python
from flask import Blueprint, jsonify, request, g
import os
import jwt
import bcrypt
import psycopg2, psycopg2.extras

authentication_blueprint = Blueprint('authentication_blueprint', __name__)

@authentication_blueprint.route('/auth/signup', methods=['POST'])
def signup():
    try:
        new_user_data = request.get_json()
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM users WHERE username = %s;", (new_user_data["username"],))
        existing_user = cursor.fetchone()
        if existing_user:
            cursor.close()
            return jsonify({"error": "Username already taken"}), 400
        hashed_password = bcrypt.hashpw(bytes(new_user_data["password"], 'utf-8'), bcrypt.gensalt())
        cursor.execute("INSERT INTO users (username, password) VALUES (%s, %s) RETURNING username, id", (new_user_data["username"], hashed_password.decode('utf-8')))
        created_user = cursor.fetchone()
        con.commit()
        cursor.close()
        token = jwt.encode(created_user, os.getenv('JWT_SECRET'))
        return jsonify({"token": token, "user": created_user}), 201
    except Exception as error:
        return jsonify({"error": str(error)}), 401
    finally:
        cursor.close()

@authentication_blueprint.route('/auth/signin', methods=["POST"])
def signin():
    try:
        sign_in_form_data = request.get_json()
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
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
        cursor.close()
```

You'll notice something's off, though: where is our `get_db_connection` function?

Let's split that off into a separate file right now as well, since many pieces of our app will want this function.

Create `db_helpers.py` and add the `get_db_connection` function to it:

```python
def get_db_connection():
    conn = psycopg2.connect(host='localhost',
                            database=os.getenv('POSTGRES_DATABASE'),
                            user=os.getenv('POSTGRES_USERNAME'),
                            password=os.getenv('POSTGRES_PASSWORD'))
    return conn
```

Then, add the import to `auth_blueprint.py`:

```python
from db_helpers import get_db_connection
```

You should now be able to test this out in Postman and receive the same responses to `localhost:5000/auth/signin` and `localhost:5000/auth/signup` as you previously did!

In real life, though, you probably have some errors to work through after this much refactoring, so keep at it until your Postman requests are error-free, and we'll see you in the next step.
