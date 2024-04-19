# ![JWT Authentication in Flask APIs - Sign In Route](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to implement a sign in route with Flask and JWTs.

## Sign In Route

Now that a user can sign up, next let's create a route that allows users to sign in to our application. As with the sign up route, we'll be using `bcrypt` - this time to handle checking the user's credentials. 

We'll use the built in Bcrypt `checkpw()` method to check the user's password. This method takes two arguments - an unhashed password and a hashed password - and checks if they match. If so, the method returns `true`, otherwise it will return `false`. 

For now, we'll just return a boilerplate message in the instance that the passwords match, return a different message for invalid credentials, and add our standard error handling in the `except` block:  

```python
@app.route('/auth/signin')
def signin():
    try:
        sign_in_form_data = request.get_json()
        con = get_db_connection()
        cursor = con.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute("SELECT * FROM users WHERE username = %s;", (sign_in_form_data["username"],))
        existing_user = cursor.fetchone()
        if existing_user is None:
            return jsonify({"error": "Invalid credentials."}), 401
        password_is_valid = bcrypt.checkpw(sign_in_form_data["password"], existing_user["password"])
        if not password_is_valid:
            return jsonify({"error": "Invalid credentials."}), 401
        return jsonify({"message": "Successful credentials."})
    except:
        return jsonify({"error": "Invalid credentials."}), 401
    finally: 
        cursor.close()
```

Next, we want to send a token back to the client in the instance that they are authorised to sign in. We'll create a new `token` variable, and assign it to the result of signing the token by passing the valid user's username and id as the payload, and the `JWT_SECRET` from our `.env` file as the secret key. Finally, we send this `token` to the client, giving it a status of `200` (OK). 

```python
token = jwt.encode({"username": existing_user["username"], "id": existing_user["id"]}, os.getenv('JWT_SECRET'))
return jsonify({"token": token}), 200
```

Fantastic! We now have a working Sign In route that will check the user's credentials, and then create and supply a JWT back to the user if their password matches the hashed password on the server. 

## Refactoring the Sign Up Process

In most applications, signing up for a new account will automatically authenticate the new user. Let's implement that by adding our fancy new `jwt` code to the response from our `/auth/signup` route, and including the token in our response as well:

```python
token = jwt.encode(created_user, os.getenv('JWT_SECRET'))
return jsonify({"token": token, "user": created_user}), 201
```

Next, we'll work on using the token to protect routes in our app! 