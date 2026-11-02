<h1>
  <span class="headline">Hoot Back-End</span>
  <span class="subhead">Setup</span>
</h1>

## Setup

Open your Terminal application and navigate to your `~/code/ga/lectures` directory:

```bash
cd ~/code/ga/lectures
```

## Cloning the auth boilerplate

This lecture uses the [Flask JWT Auth Template](https://github.com/ga-curriculum/flask-jwt-auth-template.git) as starter code. The template includes code to authenticate users with JWT tokens.

Navigate to the [Flask JWT Auth Template](https://github.com/ga-curriculum/flask-jwt-auth-template.git) and clone the repository to your machine:

```bash
git clone https://github.com/ga-curriculum/flask-jwt-auth-template.git
```

Once we have the repository on our machines, we can change the name of the directory to `flask-api-hoot-back-end`:

```bash
mv flask-jwt-auth-template flask-api-hoot-back-end
```

Next, `cd` into your renamed directory:

```bash
cd flask-api-hoot-back-end
```

Finally, remove the existing `.git` information from this template:

```bash
rm -rf .git
```

> 💡 Removing the `.git` info is important as this is just a starter template provided by GA. You do not need the existing git history for this project.

Open the project's folder in your code editor:

```bash
code .
```

## GitHub setup

To add this project to GitHub, initialize a new Git repository:

```bash
git init
git add .
git commit -m "Initial commit"
```

Make a new repository on [GitHub](https://github.com/) named `flask-api-hoot-back-end`.

Link your local project to your remote GitHub repo:

```bash
git remote add origin https://github.com/<github-username>/flask-api-hoot-back-end.git
git push origin main
```

> 🚨 Do not copy the above command. It will not work. Your GitHub username will replace `<github-username>` (including the `<` and `>`) in the URL above.

## Install dependencies

Next, you will want to install all of the packages listed in the `Pipfile`:

```bash
pipenv install
```

Activate the virtual environment:

```bash
pipenv shell
```

## Create your `.env`

Run the following command in your terminal:

```bash
touch .env
```

Add the following secret keys to your `.env` file:

```
JWT_SECRET=<your-personal-secret-string>
POSTGRES_USERNAME=<your-username>
POSTGRES_PASSWORD=<your-password>
POSTGRES_DATABASE="flask_hoot"
```

> 💡 Notice the `POSTGRES_DATABASE` variable included here.

## Run the application

Add the following route to `app.py`:

```python
# Index route:
@app.route('/')
def index():
  return "Hello, world!"
```

Start the application with the following command:

```bash
python app.py
```

In your browser, send a `GET` request to `http://127.0.0.1:5000/` to hit the index route.

To deactivate the virtual environment when you're done, simply type:

```bash
exit
```

## Creating the SQL Database for this project

Let's take a moment to setup our initial database.

We'll need to execute SQL commands to create a `flask_hoot` database and `users` table.

Activate the `psql` shell with the following command:

```bash
psql
```

Create the database:

```sql
CREATE DATABASE flask_hoot;
```

Connect to the database:

```sql
\c flask_hoot
```

And add the `users` table:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

> 😎 Remember, you can exit the `psql` shell by running `\q` in your terminal.

Next, we'll modify the connection code in `app.py` to connect to the `flask_hoot` database. We can accomplish this with the use of our `POSTGRES_DATABASE` environment variable.

Update `app.py` with the following:

```python
def get_db_connection():
    connection = psycopg2.connect(
        host='localhost',
        database=os.getenv('POSTGRES_DATABASE'),
        user=os.getenv('POSTGRES_USERNAME'),
        password=os.getenv('POSTGRES_PASSWORD')
    )
    return connection
```
