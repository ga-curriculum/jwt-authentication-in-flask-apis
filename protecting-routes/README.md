# ![JWT Authentication in Flask APIs - Protecting Routes](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to use a token payload to protect a route from unauthorized users.

## Protecting routes with middleware

At this stage, we're authenticating users when they sign in to our app by providing them a token to include with their future requests. Next, we want to use those tokens to protect specific routes from unauthenticated or unauthorized users.

We could write code to handle this in each function that is attached to a protected route, but that would not be very DRY.

Instead, let's create middleware that can be used on any route that should require authentication before proceeding.

We'll use Flasks `decorator` pattern to create our own custom decorator, which will use the same syntax as our `@app.route()` decorator!

Let's [consult the docs](https://flask.palletsprojects.com/en/2.3.x/patterns/viewdecorators/) on the pattern before writing our own.

When first learning how to implement JWTs in Flask, we wrote a function that accepts and verifies a token:

```python
@app.route('/verify-token', methods=['POST'])
def verify_token():
    try:
        token = request.headers.get('Authorization').split(' ')[1]
        decoded_token = jwt.decode(token, os.getenv('JWT_SECRET'), algorithms=["HS256"])
        return jsonify({"user": decoded_token})
    except Exception as error:
       return jsonify({"error": error.message})
```

This function will work well as a middleware, with a few small changes.

Instead of sending a response to a successful token, the function should return _the result of the next function it was passed_ using the decorator pattern.

Let's take this complex maneuver one step at a time, starting with the Flask documentation's decorator example for guidance:

```python
def token_required(f)
```

This line defines our decorator function, `token_required`, and it accepts the next function in the middleware stack as an input parameter. We'll call upon this input function (`f`) to pass the request along to its final destination.

```python
def token_required(f)
    @wraps(f)
    def decorated_function():
        return f(*args, **kwargs)
    return decorated_function
```

This version of the decorator wouldn't accomplish anything, but it's helpful as an example of the pattern we'll use.

Inside our custom decorator, we're using the func_tools `@wraps(f)` to let Python know we're creating a decorator that wraps around another function (`f`).

We then define the actual functionality we want the decorator to have in the form of our `decorator_function` definition.

Our simple example is merely passing the request down the chain by invoking the next function (`f`), with all the same args and kwargs the decorator is being provided.

## Building `token_required` middleware

Let's apply this approach to our Flask app.

First, create a new middleware file called `auth_middleware`:

```bash
touch auth_middleware.py
```

Add the following imports to the top of `auth_middleware.py`:

```python
from functools import wraps
from flask import request, jsonify
import jwt
import os
```

Next, let's add the `token_required` function:

```python
# auth_middleware.py
def token_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        authorization_header = request.headers.get('Authorization')
        # check if there is a header before attempting to decode it
        if authorization_header is None:
            return jsonify({"error": "Unauthorized"}), 401
        try:
            # remove the 'Bearer' portion of the Auth header string
            token = authorization_header.split(' ')[1]
            # decode will throw an error if the token is invalid, triggering the except block automatically
            jwt.decode(token, os.getenv('JWT_SECRET'), algorithms=["HS256"])
        except Exception as error:
            return jsonify({"error": str(error)}), 500
        return f(*args, **kwargs)
    return decorated_function
```

Now our decorator will act as a filter to ensure valid tokens are attached before forwarding the request.

In most applications, however, we want to do a bit more than that. Specifically, if a request requires a signed-in user to proceed, we want the eventual controller function to have access to the user stored in the token.

To add data along the middleware stack, Flask provides us a `g` built-in tool that allows us to propogate data along the request chain.

Let's add that to our growing list of flask imports, and then we'll use it in the eventual routes that get screened by this middleware.

Update the existing import in `app.py` with `g`:

```python
# app.py
from flask import request, jsonify, g
```

And update `token_required` with the following:

```python
def token_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        authorization_header = request.headers.get('Authorization')
        if authorization_header is None:
            return jsonify({"error": "Unauthorized"}), 401
        try:
            token = authorization_header.split(' ')[1]
            token_data = jwt.decode(token, os.getenv('JWT_SECRET'), algorithms=["HS256"])
            g.user = token_data
        except Exception as error:
            return jsonify({"error": str(error)}), 500
        return f(*args, **kwargs)
    return decorated_function
```

## Using the token_required middleware

It's been a long journey getting this far, so let's create the simplest possible route to test our `@token_required` decorator on.

This route will be a VIP Lounge that greets authenticated users by their username, but forbids unauthenticated users.

- Route: `/vip-lounge`
- Method: `GET`
- Response Body: `Welcome to the party, {username}`

First, add the following imports to `app.py`:

```python
# app.py

# Import our custom middleware:
from auth_middleware import token_required

# Add the g object to our list of flask imports:
from flask import Flask, jsonify, request, g
```

Next, add the following route:

```python
@app.route('/vip-lounge')
@token_required
def vip_lounge():
    return f"Welcome to the party, {g.user['username']}"
```

Since we attached the data from the token to the `g` object, we can access anything stored in that token (the user's `id` is also there!).

Now, let's test this out in Postman.

In Postman, sign up or sign in to obtain a token. After obtaining a token, attach it to Postman's Auth section as we've done previously.

With the token set in Postman's Auth headers, make a request to `/vip-lounge`.

It should show a message greeting the user you've just signed in with!

## Authorization

For many routes, authentication is enough. The dashboard of most web applications will be accessible to any signed-in user, but will reject requests from someone not yet signed in.

Some routes, however, are also interested in **authorizing** the user, and only allowing access if the request is coming from a specific user.

Any routes requiring authentication through our `@token_required` middleware will also grant us access to the `id` of the user. We could then use this to determine if that specific user is allowed to fulfill the request or not.

This strategy is applicable to any case where you want to restrict actions to the user who owns a specific resource, like editing or deleting their posts on social media.

A more sophisticated authorization approach would grant users specific roles and permission levels, allowing an application's administrators to have access to any user's resources.

## Conclusion

We have an Flask app that authenticates and authorizes users through a JWT approach. Consider it the start of a re-usable boilerplate that's useful for any Flask API requiring user management, but don't consider it the endpoint of proper user management in an application. For that, you'd need email verification, password management, oAuth considerations, and many other features expected and required of applications managing users.

Besides, there are plenty of authentication management libraries in the Python ecosystem, so creating your own from scratch with JWT like we've done here is hardly necessary. At their foundation, however, all token-based authentication libraries and APIs are employing the same strategy we've implemented in this lesson, and being comfortable with the management and exchange of these tokens is critical no matter which token authentication implementation you're working with.
