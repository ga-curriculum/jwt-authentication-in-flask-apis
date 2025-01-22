<h1>
  <span class="headline">JWT Authentication in Flask APIs</span>
  <span class="subhead">Sign Up Route</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to implement a sign up route with Flask, Postgres, and JWTs.

## User model

This section will set up an Express application to use JSON Web Tokens (JWT) for **authentication** and **authorization**. For this, we will need a user model to store user information. We'll use Postgres to create users with the following fields:

| Field    | Type   | Description            |
| -------- | ------ | ---------------------- |
| id       | Int    | User's id              |
| username | String | User's username        |
| password | String | User's hashed password |

## Create a user table

To set up for this step in our project, we'll need to create a new database for this application called `flask_auth_db`, along with a `user` table, using the `psql` shell.

Open your terminal and run the following command:

```bash
psql
```

Next, we'll initialize a new database for this project:

```sql
CREATE DATABASE flask_auth_db;
```

To connect to the new database, run the following:

```sql
\c flask_auth_db;
```

After running the command above, you should see something like the following in your terminal:

```plaintext
You are now connected to database "flask_auth_db" as user "<your-user-name>"
```

Now that we are connected to the database we can create a table:

```sql
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        username VARCHAR(50) NOT NULL,
        password VARCHAR(255) NOT NULL
    );
```

> 💡 Make sure the `user` table has the `username` and `password` fields as described above.

## Sign up route

Inside our `app.py`, create a `POST` route for `/sign-up`. This route will be responsible for creating a new user in the database, but for now, let's just send back a message to the user to let them know that the route is working.

```python
@app.route('/auth/sign-up', methods=['POST'])
def sign_up():
    return jsonify({"message": "Sign up route reached."})
```

Using Postman, make a POST request to `http://localhost:5000/auth/sign-up` and check if you get the expected response:

```json
{
  "message": "Sign up route reached."
}
```

## bcrypt

We never want to save passwords directly to the database. This is because if our database gets hacked people's passwords would be exposed to the hackers which could lead to other accounts using the same password being stolen.

In this step, we will use `bcrypt` to hash the user's password before saving it to the database.

Bcrypt is a package that will encrypt passwords for us, so that we are not saving them directly to the database. Knowing precisely how Bcrypt works is not essential, but let's review a high-level overview of what it's doing to our passwords:

1. **Hashes the Password**: Applies a one-way hash function to our passwords, converting them into a fixed-length string of characters. This mathematical process produces a unique output for each input, making it practically impossible to reverse-engineer the original password from the hash.
2. **Salting**: "Salting" is where a random string of characters is added to the password before hashing it. This random string of characters is often called the "salt" - adding the "salt" to a password ensures that if two users have the same password, they do not end up with the same hash.
3. **Adjustable Work Factor**: You can adjust the "work factor" or the number of times a password is hashed. However, this will increase the time the hash function runs. As we increase the number of times we hash, we directly increase the time the function will need to run.

Two main methods in the Bcrypt library allow us to hash passwords.

- `hashpw()`: Used to generate a hash for the given string. It returns the hashed string.
- `gensalt()`: Used to generate the salt that ensures hashes are unique.

Install the `bcrypt` library with `pipenv`:

```bash
pipenv install bcrypt
```

At the top of `app.py`, add an import for `bcrypt`:

```python
import bcrypt
```

Inside our `sign_up` route, remove the placeholder return value and include a `try...except` block:

```python
@app.route('/auth/sign-up', methods=['POST'])
def sign_up():
    try:
        return jsonify({"message": "Sign up route reached."})
    except Exception as err:
        return jsonify({"err": err.message})
```

We will create a new user using SQL statements, so let's set up a helper function to manage database connections.

In `app.py` import `psycopg2`:

```python
# Add psycopg2 import
import psycopg2, psycopg2.extras
```

And add the following:

```python
def get_db_connection():
    connection = psycopg2.connect(host='localhost',
                            database='flask_auth_db',
                            user=os.getenv('POSTGRES_USERNAME'),
                            password=os.getenv('POSTGRES_PASSWORD'))
    return connection
```

Add the following variables to your `.env`:

```plaintext
POSTGRES_USER=<your-postgres-username>
POSTGRES_PASSWORD=<your-postgres-password>
```

> 🚨 Make sure you've also defined the variables `POSTGRES_USERNAME` and `POSTGRES_PASSWORD` in your `.env` file to help keep this connection process secure.

For the field `hashedPassword`, we want to call the Bcrypt method `hashpw`. This method intakes two arguments:

- The password to be hashed.
- The salt (generated by bcrypt.gensalt()).

Read the very limited [documentation](https://github.com/pyca/bcrypt/) carefully.

The password input needs to be of type `bytes`, as shown in the example usage:

```python
import bcrypt
password = b"super secret password"
# Hash a password for the first time, with a randomly-generated salt
hashed = bcrypt.hashpw(password, bcrypt.gensalt())
# Check that an unhashed password matches one that has previously been
# hashed
if bcrypt.checkpw(password, hashed):
    print("It Matches!")
else:
    print("It Does not Match :(")
```

When passing the password into the method, therefore, we'll need to use the `bytes()` type conversion function. We'll also need to decode the `bytes` type response from the hashing function into a string before storing it in the database.

Once we create the new user, we will want to send the status of `201` created along with the user to the client.

Of course, it's also vital that we don't end up with two users with the same username, so we'll also add a database call checking whether the username is already taken before creating a new one.

The last thing we have to do for this route is handle any errors that might happen. Let's do that by setting the status code to `401` and sending the error message to the client.

Update your `sign_up` route with the following:

```python
@app.route('/auth/sign-up', methods=['POST'])
def sign_up():
    try:
        new_user_data = request.get_json()
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM users WHERE username = %s;", (new_user_data["username"],))
        existing_user = cursor.fetchone()
        if existing_user:
            cursor.close()
            return jsonify({"err": "Username already taken"}), 400
        hashed_password = bcrypt.hashpw(bytes(new_user_data["password"], 'utf-8'), bcrypt.gensalt())
        cursor.execute("INSERT INTO users (username, password) VALUES (%s, %s)", (new_user_data["username"], hashed_password.decode('utf-8')))
        created_user = cursor.fetchone()
        connection.commit()
        connection.close()
        return jsonify(created_user), 201
    except Exception as err:
        return jsonify({"err": str(err)}), 401
```

In Postman, test this route by making a POST request to `http://localhost:5000/auth/sign-up`.

Make sure to send the correct body in the request. It should be formatted as such:

```json
{
  "username": "<insert test username>",
  "password": "<insert test password>"
}
```

We should receive a response that includes the hashed password like the example below:

```json
{
  "id": 5,
  "password": "$2b$12$A142ODnQDdanYc1QnY21Q.8OKfA6bjKTIHa2gYZPJSLD72TeRc7Mq",
  "username": "myusername"
}
```

## Remove the hashed password from the response

While this is not our plain text password, we still shouldn't send it inside a JSON object. It should live safely inside our database. Let's fix that by adjusting our SQL statement:

```python
cursor.execute("INSERT INTO users (username, password) VALUES (%s, %s) RETURNING id, username", (new_user_data["username"], hashed_password.decode('utf-8')))
```

Test again with Postman, and you can see that the hashed password has been removed.

## Return a token instead of the user

We'll use the same `jwt.encode()` method we used in the `/sign-token` route to create a token. This method takes two arguments:

- The payload: An object that contains the data we want to encode in the token. We'll craft the payload with the exact data we want - just the `username` and the `_id` of a user.
- The secret: A string that is used to sign the token. This is the secret we stored in an environment variable earlier.

> 🧠 The payload can contain any data you want, but it's best to keep it small since the data in the payload is attached to every authenticated request the client makes. You don't want to store data that will change frequently in the token, as it could become stale.

Here's our final `sign-up` route:

```python
@app.route('/auth/sign-up', methods=['POST'])
def sign_up():
    try:
        new_user_data = request.get_json()
        connection = get_db_connection()
        cursor = connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM users WHERE username = %s;", (new_user_data["username"],))
        existing_user = cursor.fetchone()
        if existing_user:
            cursor.close()
            return jsonify({"err": "Username already taken"}), 400
        hashed_password = bcrypt.hashpw(bytes(new_user_data["password"], 'utf-8'), bcrypt.gensalt())
        cursor.execute("INSERT INTO users (username, password) VALUES (%s, %s) RETURNING id, username", (new_user_data["username"], hashed_password.decode('utf-8')))
        created_user = cursor.fetchone()
        connection.commit()
        connection.close()
        # Construct the payload
        payload = {"username": created_user["username"], "id": created_user["id"]}
        # Create the token, attaching the payload
        token = jwt.encode({ "payload": payload }, os.getenv('JWT_SECRET'))
        # Send the token instead of the user
        return jsonify({"token": token}), 201
    except Exception as err:
        return jsonify({"err": str(err)}), 401
```

Test this in Postman by making a POST request to `http://localhost:3000/auth/sign-up`. You should receive a token in the response that looks like this:

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkIjp7InVzZXIiOiJnYXJ5aG9zdCIsIl9pZCI6IjY3N2RmMTg1MGU1ZjM2MWFhNGU2NmViYSJ9LCJpYXQiOjE3MzYzMDcwNzd9.C5ofiUptXCPKPHI5afLHBWCrUk6BLjGMdcBM1nEjRP4"
}
```

You can confirm the token includes the specified payload by pasting the token's value (excluding the quotes) into [jwt.io](https://jwt.io/#debugger-io) and observing the decoded payload.
