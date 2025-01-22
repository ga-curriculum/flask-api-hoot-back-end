<h1>
  <span class="headline">Hoot Back-End</span>
  <span class="subhead">Setting up the SQL Database</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able create a `hoots` table in PostgreSQL.

## Creating the `hoots` table

Let's create the table for the main resource in our app: `hoots`.

When building our `hoots` table, we'll want to associate its author field with `users`.

Begin by activating the `psql` shell:

```bash
psql
```

Connect to the database:

```sql
\c flask_hoot
```

Run the following command:

```SQL
CREATE TABLE hoots (
    id SERIAL PRIMARY KEY,
    author INTEGER REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255),
    text VARCHAR(280),
    category VARCHAR(50)
);
```

## Querying the table

Assuming you've created a user with the id of `1` during your testing of the application, let's run some test queries to ensure things are set up correctly. We'll create a sample hoot associated with the first user to sign-up in our application.

Run the following command:

```SQL
INSERT INTO hoots (author, title, text, category) VALUES (1, 'hello world', 'greetings to you all', 'news');
```

If the command was successful, you should see the following response:

```SQL
INSERT 0 1
```

Now try inserting a hoot with a non-existent author:

```SQL
INSERT INTO hoots (author, title, text, category) VALUES (1066, 'Prepare for Glory', 'Harold Godwinson sends his regards', 'history');
```

This should produce the following error:

```SQL
ERROR:  insert or update on table "hoots" violates foreign key constraint "hoots_author_fkey"
```

The foreign key constraint ensures that the `author` value in the `hoots` table must exist in the `users` table. Since author `1066` does not exist in the `users` table, the constraint prevents the insertion. Excellent!

Let's test the association with a nice `JOIN` query to include usernames on our hoots:

```sql
SELECT * FROM hoots JOIN users ON hoots.author = users.id;
```

The above command should produce something like the following output:

```sql
 id | author |    title    |         text         | category | id | username |                           password
----+--------+-------------+----------------------+----------+----+----------+--------------------------------------------------------------
  1 |      1 | hello world | greetings to you all | news     |  1 | cool_user_name   | $2b$12$oCYf0q64HuyANz3NX0JPYeq9pld3jUKGUKlw5zBB1JvRl/9xm4/za
```

> 💡 These results might differ if you have created multiple hoot records. Remember, `SELECT *` returns all of the rows that exist on the specified table.

Looks great, but we don't really want our password hashes being sent with this query. Let's modify the `SELECT` section:

```SQL
SELECT hoots.*, users.username FROM hoots JOIN users ON hoots.author = users.id;
```

That's better!

Let's also test out our `CASCADE` functionality by deleting user 1 and checking if their hoots still exist:

```SQL
DELETE FROM users WHERE users.id = 1;
SELECT * FROM hoots;
```

You should see 0 hoots remaining in the database, since their author was deleted.

## Creating the `comments` table

Next we'll need to create the `comments` table.

Run the following command:

```sql
CREATE TABLE comments (
id SERIAL PRIMARY KEY,
author INTEGER REFERENCES users(id) ON DELETE CASCADE,
hoot INTEGER REFERENCES hoots(id) ON DELETE CASCADE,
text VARCHAR(280)
);
```

> 🏆 Use this opportunity to try out some SQL queries, testing the connections and cascades between them. Be sure to leave a few comment records in the database, they'll prove useful in the following lessons.
