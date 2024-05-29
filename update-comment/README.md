# ![Flask API - Hoot Back-End - Update Comment](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able build a route that updates a comment.

## Overview

In this section, we will make a route to create a new comment. This route will be a `PUT` request to `/hoots/<hoot_id>/comments/<comment_id>`, and will return a JSON response with the new comment. The purpose of this route is to handle data that is sent along with a **form submission**.

We will be following these specs when building the route:

- CRUD Action: CREATE
- Method: `PUT`
- Path: `/hoots/<hoot_id>/comments/<comment_id>`
- Response: JSON
- Success Status Code: `201` Created
- Success Response Body: A new hoot object
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an `error` key and a message describing the error

## Define the route

Next we'll define a route that listens for `PUT` requests on `/hoots/<hoot_id>/comments/<comment_id>`:

```
PUT /hoots/<hoot_id>/comments/<comment_id>
```

We'll make use of our `@token_required` decorator, as only a logged in user should be able to update a comment.

Add the following to `comments_blueprint.py`:

```python
@comments_blueprint.route('/hoots/<hoot_id>/comments/<comment_id>', methods=['PUT'])
@token_required
```

> 🏆 Any route that requires auth should include the `@token_required` decorator. These are our **protected routes**.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL query:

```sql
UPDATE comments SET text = %s WHERE id = %s RETURNING *
```

This query updates the text of an existing comment in the `comments` table. It sets the `text` column to a new value for the comment with the specified `id`. The `RETURNING *` clause ensures that all columns of the updated comment are returned, allowing you to retrieve and verify the updated data immediately.

As we saw when updating a hoot, we'll need to determine whether the user making the request is, in fact, the comment's original creator.

This will require **two** separate queries: 

- One to get the commment from the database and check who the author is,

- And the second to actually update the comment.

Add the following to `comments_blueprint.py`:

```python
@comments_blueprint.route('/hoots/<hoot_id>/comments/<comment_id>', methods=['PUT'])
@token_required
def update_comment(hoot_id, comment_id):
    try:
        updated_comment_data = request.json
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM comments WHERE id = %s", (comment_id,))
        comment_to_update = cursor.fetchone()
        if comment_to_update is None:
            return jsonify({"error": "Comment not found"}), 404
        if comment_to_update["author"] != g.user["id"]:
            return jsonify({"error": "Unauthorized"}), 401
        cursor.execute("UPDATE comments SET text = %s WHERE id = %s RETURNING *", 
                       (updated_comment_data["text"], comment_id))
        updated_comment = cursor.fetchone()
        connection.commit()
        connection.close()
        return jsonify({"comment": updated_comment}), 201
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Test the route in Postman

Now that we have the route, let's test it with Postman. 

We'll do this by sending a `PUT` request to the following URL:

```plaintext
http://127.0.0.1:5000/hoots/4/comments/1
```

> 💡 Be sure to update this with hoot and comment `id` values that exists in your database!

The body of your request should look like the following:

```json
{
    "text": "Updated comment text"
}
```

If the request is successful, you should see a response like the one below:

```json
{
    "comment": {
        "author": 4,
        "hoot": 4,
        "id": 1,
        "text": "Updated comment text"
    }
}
```