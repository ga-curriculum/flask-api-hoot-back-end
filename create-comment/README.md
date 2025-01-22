<h1>
  <span class="headline">Hoot Back-End</span>
  <span class="subhead">Create Comment</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able build a route that creates a comment associated with a specific hoot.

## Overview

In this section, we will make a route to create a new hoot. This route will be a `POST` request to `/hoots/<hoot_id>/comments`, and will return a JSON response with the new comment. The purpose of this route is to handle data that is sent along with a **form submission**.

We will be following these specs when building the route:

- CRUD Action: CREATE
- Method: `POST`
- Path: `/hoots/<hoot_id>/comments`
- Response: JSON
- Success Status Code: `201` Created
- Success Response Body: A new hoot object
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an `error` key and a message describing the error

## Define the route

Next we'll define a route that listens for `POST` requests on `/hoots/<hoot_id>/comments`:

```
POST /hoots/<hoot_id>/comments
```

We'll make use of our `@token_required` decorator, as only a logged in user should be able to create a comment.

Add the following to `comments_blueprint.py`:

```python
@comments_blueprint.route('/hoots/<hoot_id>/comments', methods=['POST'])
@token_required
```

> 🏆 Any route that requires auth should include the `@token_required` decorator. These are our **protected routes**.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL query:

```sql
INSERT INTO comments (hoot, author, text)
VALUES (%s, %s, %s)
RETURNING *
```

This query inserts a new comment into the `comments` table. The columns `hoot`, `author`, and `text` will be populated with the provided values. The `RETURNING *` clause ensures that all columns of the newly inserted comment are returned.

After inserting the new row in the `comments` table, our function will respond to the request with the `created_comment` as JSON data.

Add the following to `comments_blueprint.py`:

```python
@comments_blueprint.route('/hoots/<hoot_id>/comments', methods=['POST'])
@token_required
def create_comment(hoot_id):
    try:
        new_comment_data = request.get_json()
        new_comment_data["author"] = g.user["id"]

        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)

        cursor.execute("""
                        INSERT INTO comments (hoot, author, text)
                        VALUES (%s, %s, %s)
                        RETURNING *
                        """,
                        (hoot_id, new_comment_data['author'], new_comment_data['text'])
        )
        created_comment = cursor.fetchone()
        connection.commit()
        connection.close()
        return jsonify({"comment": created_comment}), 201
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Test the route in Postman

Now that we have the route, let's test it with Postman.

We'll do this by sending a `POST` request to the following URL:

```plaintext
http://127.0.0.1:5000/hoots/4/comments
```

> 💡 Be sure to update this with a hoot `id` value that exists in your database!

The body of your request should look like the following:

```json
{
  "text": "Comment text"
}
```

If the request is successful, you should see a response like the one below:

```json
{
  "comment": {
    "author": 4,
    "hoot": 4,
    "id": 1,
    "text": "Comment text"
  }
}
```
