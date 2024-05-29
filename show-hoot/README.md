# ![Flask API - Hoot Back-End - Show Hoot](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able build a route to retrieves a single hoot from the database and sends this data as a JSON response to the client.

## Overview

In this section, we will create a show route to find a single hoot. This route will be a `GET` request to `/hoots/<hoot_id>`, returning a JSON response with a single hoot from the database.

We will be following these specs when building the route:

- CRUD Action: READ
- Method: `GET`
- Path: `/hoots/:hootId`
- Response: JSON
- Success Status Code: `200` Ok
- Success Response Body: A JSON object with the hoot that matches the hootId parameter.
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an error key and a message describing the error.

## Define the route

Our route will listen for `GET` requests on `'/hoots/<hoot_id>'`:

```
GET /hoots/<hoot_id>
```

Add the following to `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['GET'])
```

Note, for this route, we'll have to *parameterize* the query. The term *parameterize* in the context of a SQL query refers to safely inserting the value of `hoot_id` into the SQL query using placeholders, which helps prevent SQL injection attacks. When using parameterized queries, the placeholder `%s` is used in the SQL statement and the actual value is provided as a separate argument to the `cursor.execute` method.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL query:

```sql
SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
        FROM hoots h 
        INNER JOIN users u_hoot ON h.author = u_hoot.id 
        LEFT JOIN comments c ON h.id = c.hoot 
        LEFT JOIN users u_comment ON c.author = u_comment.id
        WHERE h.id = %s;"""
```

The query fetches a specific hoot's details, including its author and associated comments, along with the usernames of both the hoot's author and the comment authors. The `INNER JOIN` ensures that each hoot has an associated author, while the `LEFT JOIN` ensures all related comments and their authors are included, even if some comments don't exist. The query is filtered to return data for a specific hoot based on its `id`.

From there, we have to process it to consolidate any comments into a list property, similar to our index route's processing. The response from SQL will be a list of rows, even though they only relate to a single hoot, so we can still use our processing function from earlier.

Add the following to `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['GET'])
def show_hoot(hoot_id):
    try:
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
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
            connection.close()
            return jsonify({"hoot": processed_hoot}), 200
        else:
            connection.close()
            return jsonify({"error": "Hoot not found"}), 404
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Test the route in Postman

Now that we have the route let's test it with Postman. 

We'll do this by sending a `GET` request to following URL:

```plaintext
http://127.0.0.1:5000/hoots/2
```

> 💡 Be sure to update this with a hoot `id` value that exists in your database!

If the request is successful, you should see a response like the following:

```json
{
    "hoot": {
        "author_username": "cool-username",
        "category": "news",
        "comments": [],
        "hoot_author_id": 4,
        "id": 2,
        "text": "Lorem ipsum",
        "title": "Big News"
    }
}
```
