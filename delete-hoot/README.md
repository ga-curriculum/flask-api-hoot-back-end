# ![Flask API - Hoot Back-End - Delete Hoot](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able build a route that removes a single hoot from the database by id.

## Overview

In this section, we will create a new route to delete a single hoot. This route will be a `DELETE` request to `/hoots/<hoot_id>`, returning a JSON response with the deleted hoot.

We will be following these specs when building the route:

- CRUD Action: DELETE
- Method: `DELETE`
- Path: `/hoots/<hoot_id>`
- Response: JSON
- Success Status Code: `200` Ok
- Success Response Body: A JSON object with the deleted hoot.
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an error key and a message describing the error.

## Define the route

Our route will listen for `DELETE` requests on `/hoots/<hoot_id>`:

```
DELETE /hoots/<hoot_id>
```

We'll make use of our `@token_required` decorator, as only a logged in user should be able to delete a hoot. 

Add the following:

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['DELETE'])
@token_required
```

> 🏆 Any route that requires auth should include the `@token_required` decorator. These are our **protected routes**.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL query:

```sql
DELETE FROM hoots WHERE hoots.id = %s"
```

This should resemble our update route, since we'll need to check for ownership before allowing a user to delete a hoot.

Add the following to `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['DELETE'])
@token_required
def delete_hoot(hoot_id):
    try:
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM hoots WHERE hoots.id = %s", (hoot_id,))
        hoot_to_update = cursor.fetchone()
        if hoot_to_update is None:
            return jsonify({"error": "hoot not found"}), 404
        connection.commit()
        if hoot_to_update["author"] is not g.user["id"]:
            return jsonify({"error": "Unauthorized"}), 401
        cursor.execute("DELETE FROM hoots WHERE hoots.id = %s", (hoot_id,))
        connection.commit()
        connection.close()
        return jsonify({"message": "hoot deleted successfully"}), 200
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Test the route in Postman

Now that we have finished the route let's test it with Postman. We'll do this by sending a `DELETE` request to `tktk`.