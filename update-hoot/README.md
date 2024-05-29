# ![Flask API - Hoot Back-End - Update Hoot](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able build a route that updates a single hoot in the database before issuing this updated object as a JSON response to the client.

## Overview

In this section, we will create an update route to find and update a single hoot. This route will be a `PUT` request on `/hoots/<hoot_id>`, returning a JSON response with a single updated hoot from the database.

We will be following these specs when building the route:

- CRUD Action: UPDATE
- Method: `PUT`
- Path: `/hoots/<hoot_id>`
- Response: JSON
- Success Status Code: `200` Ok
- Success Response Body: A JSON object with the updated hoot.
- Error Status Code: `500` Internal Server Error
- Error Response Body: A JSON object with an error key and a message describing the error.

## Define the route

Our route will listen for `PUT` requests on `'/hoots/<hoot_id>'`:

```
PUT /hoots/<hoot_id>
```

We'll make use of our `@token_required` decorator, as only a logged in user should be able to update a hoot. 

Add the following:

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['PUT'])
@token_required
```

> 🏆 Any route that requires auth should include the `@token_required` decorator. These are our **protected routes**.

## Code the controller function

Let's breakdown what we'll accomplish inside our controller function.

Our function will use the connection `cursor` to `execute` the following SQL query:

```sql
UPDATE hoots SET title = %s, text = %s, category = %s WHERE hoots.id = %s RETURNING *
```

The new twist in this route will be determining whether the user making the request is, in fact, the hoot's original creator!

This will require **two** separate queries: 

- One to get the hoot from the database and check who the author is,

- And the second to actually update the hoot.

Add the following to `hoots_blueprint.py`:

```python
@hoots_blueprint.route('/hoots/<hoot_id>', methods=['PUT'])
@token_required
def update_hoot(hoot_id):
    try:
        updated_hoot_data = request.json
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM hoots WHERE hoots.id = %s", (hoot_id,))
        hoot_to_update = cursor.fetchone()
        if hoot_to_update is None:
            return jsonify({"error": "hoot not found"}), 404
        connection.commit()
        if hoot_to_update["author"] is not g.user["id"]:
            return jsonify({"error": "Unauthorized"}), 401
        cursor.execute("UPDATE hoots SET title = %s, text = %s, category = %s WHERE hoots.id = %s RETURNING *", 
                        (updated_hoot_data["title"], updated_hoot_data["text"], updated_hoot_data["category"], hoot_id))
        updated_hoot = cursor.fetchone()
        connection.commit()
        connection.close()
        return jsonify({"hoot": updated_hoot}), 200
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

## Test the route in Postman

Now that we have the route, let's test it with Postman. 

We'll do this by sending a `PUT` request to the following URL:

```plaintext
http://127.0.0.1:5000/hoots/2
```

> 💡 Be sure to update this with a hoot `id` value that exists in your database!

You should see the updated hoot as a response.