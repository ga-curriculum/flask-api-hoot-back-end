# ![Flask API Related Data - SQL Queries for RESTful Routes](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to associate SQL queries to each of the RESTful routes in the Hoot application.

## RESTful Recap

As a brief refresher, let's look back over the table of RESTful routes to get a sense of the types of operations we'll need SQL queries to match:

### Hoot RESTful Routes

| HTTP Method | Function Name | URI | Use Case |
| --- | --- | --- | --- |
| POST | create | /api/hoots | Create a hoot |
| GET | index | /api/hoots | List hoots |
| GET | show | /api/hoots/:hootId | Get a single hoot |
| PUT | update | /api/hoots/:hootId | Update a hoot |
| DELETE | deleteHoot | /api/hoots/:hootId | Delete a hoot |

With nested RESTful routes for comments, like so:

| POST | createComment | /api/hoots/:hootId/comments | Create a comment |

Note that not all RESTful routes for nested resources need to be supported, depending on the application. Social media sites rarely, if ever, support accessing a page based on a single comment, so the comments routes will be limited.

Think also about what associated data an application would want with each query: the index page, for example, would certainly want the username of authors for each Hoot. JOINs will be very common in this app.

## Routes to Queries

Before creating all of this in our Flask application, however, it can be helpful to prepare the SQL queries themselves first. This ensures we can isolate problems when they arise, and will make writing the routes much easier when the heavy cognitive lifting of creating the associated SQL queries is already done.

See how much you can anticipate the necessary SQL queries before revealing our answer and testing them in your `postgres` shell.

Remember all the JOINs we'll need to support: the application will be expecting comments to come along with hoots on the show page for an individual post!

### Hoots Queries

<details>
    <summary>Hoots: Create</summary>
    INSERT INTO hoots (author, title, text, category) VALUES (2, 'testing hoot creation', 'of course the 2 would be a variable from the route', 'news') RETURNING *;
</details>

<details>
    <summary>Hoots: Index</summary>
SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
FROM hoots h 
INNER JOIN users u_hoot ON h.author = u_hoot.id 
LEFT JOIN comments c ON h.id = c.hoot 
LEFT JOIN users u_comment ON c.author = u_comment.id;

Notice the **aliasing** in this query to create a more readable response. Instead of `hoot.username`, which sounds like the username belongs to the hoot, we renamed that field to be accessible with `hoot.author_username` so the application can use a clearer property name.

We also had to join the user table in multiple places, so we aliased those tables to `u_hoot` when joining with hoots and `u_comment` when joining with comments respectively.

This was quite the query!
</details>

<details>
    <summary>Hoots: Show</summary>
    SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
    FROM hoots h 
    INNER JOIN users u_hoot ON h.author = u_hoot.id 
    LEFT JOIN comments c ON h.id = c.hoot 
    LEFT JOIN users u_comment ON c.author = u_comment.id
    WHERE h.id = 4;

    Use any number for the id that exists in your database for now. If 4 doesn't exist, try something else!
</details>

<details>
    <summary>Hoots: Update</summary>
    UPDATE hoots SET title = 'updated hoot!', text = 'updated text!', category = 'updated category!' WHERE hoots.id = 4 RETURNING *;
</details>

<details>
    <summary>Hoots: Delete</summary>
    DELETE FROM hoots WHERE hoots.id = 4;
</details>

### Comments Queries

This is where we can really limit the number of RESTful routes, since comments are so deeply tied in with hoots.

We won't need an index, since no social media sites ever list out every comment on their own, independent of the posts they're commenting on!

<details>
    <summary>Comments: Create</summary>
    INSERT INTO comments (author, hoot, text) VALUES (2, 5, 'I believe this is the finest post that was ever written') RETURNING *;

    You may have needed to create more hoots to keep going after testing the delete hoots queries!
</details>

<details>
    <summary>Comments: Update</summary>
    UPDATE comments SET text = 'updated comment!' WHERE comments.id = 2 RETURNING *;
</details>

<details>
    <summary>Comments: Delete</summary>
    DELETE FROM comments WHERE comments.id = 2;
</details>

With our SQL queries all ready to go, we'll just need to properly parameterize these queries in the route handlers using information gathered from the user's request.