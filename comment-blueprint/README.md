# ![Flask API - Hoot Back-End - Comments Blueprint](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to implement full RESTful routes with related data in Flask

## Comments Blueprint

Keeping with the modular approach to our application, let's create a new blue print to store routes related to comments.

> 💡 This will be structured much like our previous blueprints.

Run the following command in your terminal:

```bash
touch comments_blueprint.py
```

Add the following to `comments_blueprint.py`

```python
from flask import Blueprint, jsonify, request, g
from db_helpers import get_db_connection
import psycopg2.extras
from auth_middleware import token_required

comments_blueprint = Blueprint('comments_blueprint', __name__)

# Routes here
```

Add the following to `app.py`:

```python
# add to list of imports
from comments_blueprint import comments_blueprint

app.register_blueprint(comments_blueprint)
```

> 💡 Note the import of `token_required` for protected routes, such as creating a comment or editing/deleting one's own comments.