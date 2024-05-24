# ![Express API - Hoot Back-End - Setting the Stage](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to associate SQL queries to each of the RESTful routes in the Hoot application.

## SQL Queries

Before creating all of this in our Flask application, however, it can be helpful to prepare the SQL queries themselves first. This ensures we can isolate problems when they arise, and will make writing the routes much easier when the heavy cognitive lifting of creating the associated SQL queries is already done.

Review the sample queries below to get a better sense of what we'll implment in our application.

> 🏆 If you wish, you can test these out in your `postgres` shell.

## Hoots Queries

### Create

```sql
INSERT INTO hoots (author, title, text, category) VALUES (2, 'testing hoot creation', 'of course the 2 would be a variable from the route', 'news') RETURNING *;
```

### Index

```sql
SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
FROM hoots h 
INNER JOIN users u_hoot ON h.author = u_hoot.id 
LEFT JOIN comments c ON h.id = c.hoot 
LEFT JOIN users u_comment ON c.author = u_comment.id;
```

Notice the **aliasing** in this query to create a more readable response. Instead of `hoot.username`, which sounds like the username belongs to the hoot, we renamed that field to be accessible with `hoot.author_username` so the application can use a clearer property name.

We also had to join the user table in multiple places, so we aliased those tables to `u_hoot` when joining with hoots and `u_comment` when joining with comments respectively.

This was quite the query!

### Show

```sql
SELECT h.id, h.author AS hoot_author_id, h.title, h.text, h.category, u_hoot.username AS author_username, c.id AS comment_id, c.text AS comment_text, u_comment.username AS comment_author_username 
FROM hoots h 
INNER JOIN users u_hoot ON h.author = u_hoot.id 
LEFT JOIN comments c ON h.id = c.hoot 
LEFT JOIN users u_comment ON c.author = u_comment.id
WHERE h.id = 4;
```

Use any number for the id that exists in your database for now. If 4 doesn't exist, try something else!

### Update

```sql
UPDATE hoots SET title = 'updated hoot!', text = 'updated text!', category = 'updated category!' WHERE hoots.id = 4 RETURNING *;
```

### Delete

```sql
DELETE FROM hoots WHERE hoots.id = 4;
```

## Comments Queries

This is where we can really limit the number of RESTful routes, since comments are so deeply tied in with hoots.

We won't need an index, since no social media sites ever list out every comment on their own, independent of the posts they're commenting on!

### Create
```sql
INSERT INTO comments (author, hoot, text) VALUES (2, 5, 'I believe this is the finest post that was ever written') RETURNING *;
```

You may have needed to create more hoots to keep going after testing the delete hoots queries!

### Update

```sql
UPDATE comments SET text = 'updated comment!' WHERE comments.id = 2 RETURNING *;
```

### Delete

```sql
DELETE FROM comments WHERE comments.id = 2;
```

With our SQL queries all ready to go, we'll just need to properly parameterize these queries in the route handlers using information gathered from the user's request.