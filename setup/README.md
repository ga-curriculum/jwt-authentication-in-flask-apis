# ![JWT Authentication in Flask APIs - Setup](./assets/hero.png)

## Setup

Open your Terminal application and navigate to your **`~/code/ga/lectures`** directory:

```bash
cd ~/code/ga/lectures
```

Make a new directory called **`jwt-authentication-in-flask`**, then enter this directory:

```bash
mkdir jwt-authentication-in-flask
cd jwt-authentication-in-flask
```

Initialize a new virtual environment inside your project directory and install Flask and pythons postgres library:

```bash
pipenv install flask psycopg2 python-dotenv
```

This command will create a new `Pipfile` and `Pipfile.lock` in your project directory, specifying Flask as a dependency.

Activate the virtual environment:

```bash
pipenv shell
```

Create an `app.py` file by running the following command in your terminal:

```bash
touch app.py
```

Add the simplest Flask app boilerplate to `app.py`:

```python
# app.py
# Import the 'Flask' class from the 'flask' library.
from flask import Flask

# Initialize Flask
# We'll use the pre-defined global '__name__' variable to tell Flask where it is.
app = Flask(__name__)

# Define our route
# This syntax is using a Python decorator, which is essentially a succinct way to wrap a function in another function.
@app.route('/')
def index():
  return "Hello, world!"

# Run our application, by default on port 5000
app.run()
```

Run the application from the terminal:

```bash
python app.py
```

And send a test request in your browser to [localhost:5000](http://127.0.0.1:5000/) to hit the index route.

Open the contents of the directory in VSCode:

```bash
code .
```

To deactivate the virtual environment when you're done, simply type:

```bash
exit
```
