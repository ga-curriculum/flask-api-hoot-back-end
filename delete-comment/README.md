# ![Flask API - Hoot Back-End - Delete Comment](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able build a route that deletes embedded subdocuments inside a single hoot.

## Overview

In this section, we will create a delete route to find and delete a single comment within a hoot.

We will be following these specs when building the route:

- CRUD Action: DELETE
- Method: `DELETE`
- Path: `/hoots/<hoot_id>/comments/<comment_id>`
- Response: JSON
- Success Status Code: `200` Ok
- Success Response Body: A JSON status message.
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an error key and a message describing the error

## Define the route

Next we'll define a route that listens for `DELETE` requests on `/hoots/<hoot_id>/comments/<comment_id>`:

```
DELETE /hoots/<hoot_id>/comments/<comment_id>
```

We'll make use of our `@token_required` decorator, as only a logged in user should be able to delete a comment.

Add the following to `comments_blueprint.py`:

```python
@comments_blueprint.route('/hoots/<hoot_id>/comments/<comment_id>', methods=['DELETE'])
@token_required
```

> 🏆 Any route that requires auth should include the `@token_required` decorator. These are our **protected routes**.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL queries:

```sql
SELECT * FROM comments WHERE id = %s
```

```sql
DELETE FROM comments WHERE id = %s
```

This should resemble our previous update and delete routes, since we'll need to check for ownership before allowing a user to delete a comment.

Add the following to `comments_blueprint.py`:

```python
@comments_blueprint.route('/hoots/<hoot_id>/comments/<comment_id>', methods=['DELETE'])
@token_required
def delete_comment(hoot_id, comment_id):
    try:
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM comments WHERE id = %s", (comment_id,))
        comment_to_delete = cursor.fetchone()
        if comment_to_delete is None:
            return jsonify({"error": "Comment not found"}), 404
        if comment_to_delete["author"] != g.user["id"]:
            return jsonify({"error": "Unauthorized"}), 401
        cursor.execute("DELETE FROM comments WHERE id = %s", (comment_id,))
        connection.commit()
        connection.close()
        return jsonify({"message": "Comment deleted successfully"}), 200
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Test the route in Postman

Now that we have the route, let's test it with Postman. 

We'll do this by sending a `DELETE` request to the following URL:

```plaintext
http://127.0.0.1:5000/hoots/4/comments/1
```

> 💡 Be sure to update this with hoot and comment `id` values that exists in your database!

If the request is successful, you should see a response like the one below:

```json
{
    "message": "Comment deleted successfully"
}
```