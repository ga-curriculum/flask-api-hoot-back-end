# ![Flask API - Hoot Back-End - Hoot Blueprint](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to implement full RESTful routes with related data in Flask

## Hoot Blueprint

Before we start creating our routes, let's get modular and set up a Blueprint for our Hoots routes.

Create a file named `hoots_blueprint.py` and set it up similar to our starting point for the `auth_blueprint`:

```bash
touch hoots_blueprint.py
```

```python
from flask import Blueprint, jsonify, request, g
from db_helpers import get_db_connection
import psycopg2, psycopg2.extras
from auth_middleware import token_required

hoots_blueprint = Blueprint('hoots_blueprint', __name__)
```

Note the import of `token_required` for routes that will require a user, such as creating a hoot or editing/deleting one's own hoots.

Let's create a simple placeholder for the index, register this blueprint in the `app.py`, and test it out:

In `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots', methods=['GET'])
def hoots_index():
    return jsonify({"message": "hoots index lives here"})
```

In `app.py`:

```python
# add to list of imports
from hoots_blueprint import hoots_blueprint

app.register_blueprint(hoots_blueprint)
```

Send a GET request in Postman to `localhost:5000/hoots` to receive the expected response.