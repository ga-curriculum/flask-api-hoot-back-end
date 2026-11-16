<h1>
  <span class="prefix">Flask API</span>
  <span class="headline">Hoot Back-End</span>
</h1>

## About

In this module, students will build a backend for the Hoot blogging platform using Flask, PostgreSQL, and JWT authentication. They will create a RESTful API that supports full CRUD functionality for blog posts and comments. Throughout the module, students will focus on designing RESTful routes, managing data relationships, and testing API endpoints with tools like Postman. This practical experience will equip students with essential backend development skills.

## Content

| Lesson                                                                       | Skills                                                                                                             |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [Setup](./setup/README.md)                                                   | Setting up the development environment.                                                                            |
| [Setting the Stage](./setting-the-stage/README.md)                           | Understand the architecture and functional requirements of building a secure Flask API for a blogging application. |
| [Starting with Auth Boilerplate](./starting-with-auth-boilerplate/README.md) | Bootstrapping an app with our JWT auth template.                                                                   |
| [Setting Up the SQL Database](./setting-up-the-sql-database/README.md)       | Creating SQL databases and tables with foreign keys.                                                               |
| [Hoot Blueprint](./hoot-blueprint/README.md)                                 | Creating a resource blueprint.                                                                                     |
| [Create Hoot](./create-hoot/README.md)                                       | Create a `POST` route to add new hoot blog posts to the database.                                                  |
| [Index Hoots](./index-hoots/README.md)                                       | Create a `GET` route to retrieve all hoot blog posts from the database.                                            |
| [Show Hoot](./show-hoot/README.md)                                           | Create a `GET` route to retrieve a specific hoot blog post from the database by id.                                |
| [Update Hoot](./update-hoot/README.md)                                       | Create a `PUT` route to update a specific hoot blog post in the database by id.                                    |
| [Delete Hoot](./delete-hoot/README.md)                                       | Create a `DELETE` route to delete a specific hoot blog post from the database by id.                               |
| [Comments Blueprint](./comment-blueprint/README.md)                          | Creating a resource blueprint.                                                                                     |
| [Create Comment](./create-comment/README.md)                                 | Create a `POST` request to create a comment associated with a specific hoot.                                       |
| [Update Comment](./update-comment/README.md)                                 | Create a `PUT` route to update a comment by id.                                                                    |
| [Delete Comment](./delete-comment/README.md)                                 | Create a `DELETE` route to delete a comment by id.                                                                 |

## References

📖 [Reference Materials](./references/README.md)

## Internal

### Prerequisites

- Intro to SQL
- Flask Postgres REST API
- JWT Authentication with Flask

### Solution code

🏁 [Solution code](https://github.com/ga-curriculum/flask-api-hoot-back-end-solution)

### Course landing pages

- [SEB - Software Engineering Bootcamp](https://ga-curriculum.github.io/flask-api-hoot-back-end/canvas-landing-pages/seb.html)
- [Fallback](https://ga-curriculum.github.io/flask-api-hoot-back-end/canvas-landing-pages/fallback.html)

### Resources

✏️ [Instructor Guide](./internal-resources/instructor-guide.md)

🎥 [Video Hub](./internal-resources/video-hub.md)

🏗️ [Release Notes](./internal-resources/release-notes.md)

---

**Find a 👾 bug 👾 or have suggestions? [Let us know](https://ga-curriculum.github.io/universal-resources-internal/module-feedback.html)!**
