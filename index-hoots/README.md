<h1>
  <span class="headline">Hoot Back-End</span>
  <span class="subhead">Index Hoots</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to create an index route to retrieve all hoots from the database and send this data as a JSON response to the client.

## Overview

In this section, we will create a new route to find all hoots. This route will be a `GET` request to `/hoots`, and will return a JSON response with all the `hoots` in the database.

We will be following these specs when building the route:

- CRUD Action: READ
- Method: `GET`
- Path: `/hoots`
- Response: JSON
- Success Status Code: `200` Ok
- Success Response Body: An array of all the hoots in the database named hoots. The array will be empty if there are no hoots in the database.
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an `error` key and a message describing the error.

## Define the route

Our route will listen for `GET` requests on `'/hoots'`:

```
GET /hoots
```

Add the following to `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots', methods=['GET'])
```

> 🚨 If you recall, we added an example `hoots_index` route in a previous lesson. Be sure to remove or update that existing route while following these steps.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL query:

```sql
SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username
FROM hoots h
INNER JOIN users u_hoot ON h.author = u_hoot.id
LEFT JOIN comments c ON h.id = c.hoot
LEFT JOIN users u_comment ON c.author = u_comment.id;
```

This query retrieves our hoots, including their `id`, `author`, `title`, `text`, and `category` properties. It also fetches the `username` of each hoot's author, and the comments associated with each hoot.

Notice the **aliasing** in this query to create a more readable response. Instead of `hoot.username`, which sounds like the username belongs to the hoot, we renamed that field to be accessible with `hoot.author_username` so the application can use a clearer property name.

> 💡 The use of `INNER JOIN` ensures each hoot has an associated author, while `LEFT JOIN` ensures all hoots and comments are included, even if some hoots don't have comments or some comments don't have matching users.

Let's try out this query within the controller.

Add the following to `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots', methods=['GET'])
def hoots_index():
    try:
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("""SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username
                            FROM hoots h
                            INNER JOIN users u_hoot ON h.author = u_hoot.id
                            LEFT JOIN comments c ON h.id = c.hoot
                            LEFT JOIN users u_comment ON c.author = u_comment.id;
                       """)
        hoots = cursor.fetchall()
        connection.commit()
        connection.close()
        return jsonify({"hoots": hoots}), 200
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

In Postman, make a `GET` request to the following:

```plaintext
http://127.0.0.1:5000/hoots
```

When we test this in Postman, we _do_ get a response!

But check out what happens if you query for a hoot with multiple comments: you end up with a list of hoots with duplicates!

This is because the SQL response sends a new row for each individual comment. We'll need logic to zip these together into the more expected format: `comments` should be a list within the `hoot` object, and it should contain all comments associated with that hoot, like so:

```python
sampleHoot = {
    # other hoot properties
    "comments": [
        {"comment_author": "user2", "text": "I commented on this hoot"},
        {"comment_author": "user3", "text": "I, too, commented!"}
    ]
}
```

> 💡 At this point, you may see an empty comments array if you did not use SQL commands to create comments previously.

To accomplish this, we'll write a helper function named `consolidate_comments_in_hoots` that accepts the SQL response dictionaries as input, and returns the expected format of nested comments within hoots. Fortunately, we're back in vanilla python world for this problem, but this will be a tricky algorithm to write!

Remember, not every hoot has an associated comment, but some may have multiple associated comments. Since this one feels like an interview programming puzzle, there will be many possible answers.

You can see our approach below, with readability emphasized over advanced python syntax such as generators, lambdas, and other functional programming shortcuts.

Add the following to `db_helpers.py`:

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

Next, in `hoots_blueprint.py`, update the following import with `consolidate_comments_in_hoots`:

```python
# hoots_blueprint.py
from db_helpers import get_db_connection, consolidate_comments_in_hoots
```

And update `hoots_index` as shown below:

```python
@hoots_blueprint.route('/hoots', methods=['GET'])
def hoots_index():
    try:
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("""SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username
                            FROM hoots h
                            INNER JOIN users u_hoot ON h.author = u_hoot.id
                            LEFT JOIN comments c ON h.id = c.hoot
                            LEFT JOIN users u_comment ON c.author = u_comment.id;
                       """)
        hoots = cursor.fetchall()

        # Update:
        consolidated_hoots = consolidate_comments_in_hoots(hoots)

        connection.commit()
        connection.close()
        return jsonify({"hoots": consolidated_hoots}), 200
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

Try it out in Postman. Our responses should now be formatted in a more consumable format.
