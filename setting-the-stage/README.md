# ![Express API - Hoot Back-End - Setting the Stage](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will understand the architecture and functional requirements of building a secure Express API for a blogging application, including user authentication and CRUD operations for blog posts and comments.

## Overview

In this module, we'll build an Flask API that serves as the back-end for a full-stack blogging application called **Hoot**.

Within our Flask API, we'll implement full CRUD functionality on a blog post resource `hoots`. Additionally, we will implement `create` functionality on an embedded resource called `comments`.

Our Express API will make use of the [Flask API JWT Auth Template](https://git.generalassemb.ly/modular-curriculum-all-courses/flask-jwt-auth-template.git). This template provides our Flask application with pre configured JWT Authentication. 

## User stories

Our Flask API should include functionality for the following user stories:

- As a guest, I should be able to create an account.
- As a guest with an account, I should be able to log in to my account.
- AAU, I should be able to create a hoot post.
- AAU, I should be able to read a list of all hoots.
- AAU, I should be able to view information about a single hoot post along with its associated comments.
- AAU, I should be able to add a comment to a specific hoot.
- As the author of a hoot, I should be able to update that hoot.
- As the author of a hoot, I should be able to delete that hoot.

## Entity-relationship diagram

Take a look at the ERD below to get a sense of the data we'll be working with:

![ERD](./assets/erd.png)

## Express & Postman

Below you can see a chart outlining the RESTful routes required for this application:

| HTTP Method | Controller    | Response | URI                     | Use Case          |
| ----------- | ------------- | -------- | ----------------------- | ----------------- |
| POST        | create        | 200      | /hoots                  | Create a hoot     |
| GET         | index         | 200      | /hoots                  | List hoots        |
| GET         | show          | 200      | /hoots/:hootId          | Get a single hoot |
| PUT         | update        | 200      | /hoots/:hootId          | Update a hoot     |
| DELETE      | deleteHoot    | 200      | /hoots/:hootId          | Delete a hoot     |
| POST        | createComment | 200      | /hoots/:hootId/comments | Create a comment  |

> ❗ In Postman, remember to double-quote all JSON keys in the data payload (body).

Note that not all RESTful routes for nested resources need to be supported, depending on the application. Social media sites rarely, if ever, support accessing a page based on a single comment, so the comments routes will be limited.

Think also about what associated data an application would want with each query: the index page, for example, would certainly want the username of authors for each Hoot. JOINs will be very common in this app.