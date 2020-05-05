# Python Feedback App

A Python application to receive customer feedback from a form submission and sent via email - (Mock application for a Lexus Dealer)

## Notes

### Install packages

Pipenv install flask, psycopg2 and psycopg2-binary, flask-sqlalchemy, gunicorn

### Set up html templates and app.py with routes and server config

Create your html templates

Create app.py; import Flask, create '/' route, and set up server; run python app.py to run server:

```
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route('/')
def index():
    return render_template('index.html')


if __name__ == '__main__':
    app.debug = True
    app.run()
```

Create /submit route to handle the submit action from index.html:

```
@app.route('/submit', methods=['POST'])
def submit():
    if request.method == 'POST':
        customer = request.form['customer']
        dealer = request.form['dealer']
        rating = request.form['rating']
        comments = request.form['comments']
        print(customer, dealer, rating, comments)
        return render_template('success.html')
```

Set up a message in index.html

app.py:

```
if customer == '' or dealer == '':
            return render_template('index.html', message='Please enter all required fields')
```

index.html:

```
{% if message %}
      <p class="message">{{ message | safe }}</p>
      {% endif %}
```

### Set up database

