# ![Flask API Related Data - Hoots and Comments Routes](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to implement full RESTful routes with related data in Flask

## Hoot Blueprint

Before we start creating our routes, let's get modular and set up a Blueprint for our Hoots routes.

Create a file named `hoots_blueprint.py` and set it up similar to our starting point for the `auth_blueprint`:

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

## Hoots Index

Thankfully, we've done some of the hard part of this route: the multi-join SQL query.

All we have to do is set it up similar to our previous API routes. Since this is an index route, we don't even need parameters, making this a great starting point.

Let's just paste in our SQL query as the response and see how far we get:

```python
@hoots_blueprint.route('/hoots', methods=['GET'])
def hoots_index():
    try:     
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("""SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
                            FROM hoots h 
                            INNER JOIN users u_hoot ON h.author = u_hoot.id 
                            LEFT JOIN comments c ON h.id = c.hoot 
                            LEFT JOIN users u_comment ON c.author = u_comment.id;
                       """)
        hoots = cursor.fetchall()
        con.commit()
        cursor.close()
        return jsonify({"hoots": hoots})
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

Testing this in Postman, we *do* get a response! But check out what happens if you query for a hoot with multiple comments: you end up with a list of hoots with duplicates!

This is because the SQL response sends a new row for each individual comment. We'll need to do some kind of Python algorithm to zip these together into the more expected format: `comments` should be a list within the `hoot` object, and it should contain all comments associated with that hoot, like so:

```python 
sampleHoot = {
    # other hoot properties
    "comments": [
        {"comment_author": "user2", "text": "I commented on this hoot"},
        {"comment_author": "user3", "text": "I, too, commented!"}
    ]
}
```

We'll write a helper function named `conslidated_comments_in_hoots` that accepts the SQL response dictionaries as input, and returns the expected format of nested comments within hoots.

Fortunately, we're back in vanilla python world for this problem, but this will be a tricky algorithm to write! 

Remember, not every hoot has an associated comment, but some may have multiple associated comments.

Since this one feels like an interview programming puzzle, there will be many possible answers.

Here's ours, with readability emphasized over advanced python syntax such as generators, lambdas, and other functional programming shortcuts.

```python
def consolidate_comments_in_hoots(hoots_with_comments):
    print(hoots_with_comments)
    consolidated_hoots = []
    for hoot in hoots_with_comments:
        # Check if this hoot has already been added to consolidated_hoots
        hoot_exists = False
        for consolidated_hoot in consolidated_hoots:
            if hoot["id"] == consolidated_hoot["id"]:
                hoot_exists = True
                consolidated_hoot["comments"].append(                    
                    {"comment_text": hoot["comment_text"],
                     "comment_id": hoot["comment_id"],
                     "comment_author_username": hoot["comment_author_username"]
                    })
                break
        
        # If the hoot doesn't exist in consolidated_hoots, add it
        if not hoot_exists:
            hoot["comments"] = []
            if hoot["comment_id"] is not None:
                hoot["comments"].append(
                    {"comment_text": hoot["comment_text"],
                     "comment_id": hoot["comment_id"],
                     "comment_author_username": hoot["comment_author_username"]
                    }
                )
            del hoot["comment_id"]
            del hoot["comment_text"]
            del hoot["comment_author_username"]
            consolidated_hoots.append(hoot)
    
    return consolidated_hoots
```

Define this function in `db_helpers.py`, import it into the `hoots_blueprint`, and plug it into the response.

Here's the full `hoots_blueprint.py` to make sure we haven't lost track of anything:

```python
from flask import Blueprint, jsonify, request, g
from db_helpers import get_db_connection, consolidate_comments_in_hoots
import psycopg2, psycopg2.extras
from auth_middleware import token_required

hoots_blueprint = Blueprint('hoots_blueprint', __name__)

@hoots_blueprint.route('/hoots', methods=['GET'])
def hoots_index():
    try:     
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("""SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
                            FROM hoots h 
                            INNER JOIN users u_hoot ON h.author = u_hoot.id 
                            LEFT JOIN comments c ON h.id = c.hoot 
                            LEFT JOIN users u_comment ON c.author = u_comment.id;
                       """)
        hoots = cursor.fetchall()
        consolidated_hoots = consolidate_comments_in_hoots(hoots)
        con.commit()
        cursor.close()
        return jsonify({"hoots": consolidated_hoots})
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Hoots Create

Congratulations, we're past the most complex route of the app! 

Creation will be much simpler, with the only twists being parameterizing the query and adding the @token_required middleware to get access to the user creating the hoot.

```python
@hoots_blueprint.route('/hoots', methods=['POST'])
@token_required
def create_hoot():
    try:
        new_hoot = request.json
        new_hoot["author"] = g.user["id"]
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("""
                        INSERT INTO hoots (author, title, text, category) 
                        VALUES (%s, %s, %s, %s)
                        RETURNING *
                        """, 
                        (new_hoot['author'], new_hoot['title'], new_hoot['text'], new_hoot['category'])
        )
        created_hoot = cursor.fetchone()
        con.commit()
        cursor.close()
        return jsonify({"hoot": created_hoot})
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Hoots Show

For the show route, we'll have to parameterize the query we've already prepared for this route to get our SQL response.

From there, we have to process it to consolidate any comments into a list property, similar to our index route's processing.

The response from SQL will be a list of rows, even though they only relate to a single hoot, so we can still use our processing function from earlier.

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['GET'])
def show_hoot(hoot_id):
    try:
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("""
            SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
            FROM hoots h 
            INNER JOIN users u_hoot ON h.author = u_hoot.id 
            LEFT JOIN comments c ON h.id = c.hoot 
            LEFT JOIN users u_comment ON c.author = u_comment.id
            WHERE h.id = %s;""", 
            (hoot_id,))
        unprocessed_hoot = cursor.fetchall()
        if unprocessed_hoot is not None :
            processed_hoot = consolidate_comments_in_hoots(unprocessed_hoot)[0]
            return jsonify({"hoot": processed_hoot})
        else:
            return jsonify({"error": "Hoot not found"}), 404
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Hoots Update

The new twist in this route will be determining whether the user making the request is, in fact, the hoot's original creator!

This will require two separate queries: one to get the hoot from the database and check who the author is, and the second to actually update the hoot.

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['PUT'])
@token_required
def update_hoot(hoot_id):
    try:
        updated_hoot_data = request.json
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM hoots WHERE hoots.id = %s", (hoot_id,))
        hoot_to_update = cursor.fetchone()
        if hoot_to_update is None:
            return jsonify({"error": "hoot not found"}), 404
        con.commit()
        if hoot_to_update["author"] is not g.user["id"]:
            return jsonify({"error": "Unauthorized"}), 401
        cursor.execute("UPDATE hoots SET title = %s, text = %s, category = %s WHERE hoots.id = %s RETURNING *", 
                        (updated_hoot_data["title"], updated_hoot_data["text"], updated_hoot_data["category"], hoot_id))
        updated_hoot = cursor.fetchone()
        con.commit()
        cursor.close()
        return jsonify({"hoot": updated_hoot})
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Hoots Delete

A simpler version of the update route, since we still need to check for ownership, but don't need much data after that.

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['DELETE'])
@token_required
def delete_hoot(hoot_id):
    try:
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM hoots WHERE hoots.id = %s", (hoot_id,))
        hoot_to_update = cursor.fetchone()
        if hoot_to_update is None:
            return jsonify({"error": "hoot not found"}), 404
        con.commit()
        if hoot_to_update["author"] is not g.user["id"]:
            return jsonify({"error": "Unauthorized"}), 401
        cursor.execute("DELETE FROM hoots WHERE hoots.id = %s", (hoot_id,))
        con.commit()
        cursor.close()
        return jsonify({"message": "hoot deleted successfully"})
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

There we have it, our Hoots routes all complete!

## You Do: Comments Routes

Follow this pattern for creating the comments routes, including creating a comments_bluprint to go along with them.

We'll need the following routes:

- Create a comment at `/hoots/<hoot_id>/comments`
- Update a comment at `/comments/<comment_id>`
- Delete a comment at `/comments/<comment_id>`

These SQL queries will require minimal joins, since you won't need the attached hoot in your responses. Take advantage of our pre-prepared queries for these routes.