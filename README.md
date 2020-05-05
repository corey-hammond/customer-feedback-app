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

### Configure Database Connection

I created a new local development database with PostgreSQL using pgAdmin4 and connected it to the application using Flask SQLAlchemy.

In app.py:

```
ENV = 'dev'

if ENV == 'dev':
    app.debug = True
    app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://postgres:postgres@localhost/feedback_app'
else:
    app.debug = False
    app.config['SQLALCHEMY_DATABASE_URI'] = ''

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

Next, create the database model:

```
class Feedback(db.Model):
    __tablename__ = 'feedback'
    id = db.Column(db.Integer, primary_key=True)
    customer = db.Column(db.String(200))
    dealer = db.Column(db.String(200))
    rating = db.Column(db.Integer)
    comments = db.Column(db.Text())

    def __init__(self, customer, dealer, rating, comments):
        self.customer = customer
        self.dealer = dealer
        self.rating = rating
        self.comments = comments
```

Once this is configured in app.py, stop the app server, start a python cli, and run the following commands to populate the table in the database:

```
from app import db
db.create_all()
exit()
```

The 'feedback' table is now created in our database

### Make DB Queries

In our submit(), we want to take the form data and submit it to the database:

```
if db.session.query(Feedback).filter(Feedback.customer == customer).count() == 0:
            data = Feedback(customer, dealer, rating, comments)
            db.session.add(data)
            db.session.commit()

            return render_template('success.html')
        return render_template('index.html', message='You have already submitted feedback')
```

### Use MailTrap.io to Stage Emails

We're setting up a feature that allows an email to be sent with the feedback information. With each submission, the feedback data gets stored in our database and it gets sent in an email. Mailtrap.io acts as a staging area for the emails to assist us with the development process. 

Create a new file, send_mail.py, which will use the smtplib and MIMEtext libraries to send emails. Create a function that uses these libraries to capture the data and sends to mailtrap.io and the desired email recipient. 

### Deploy to Heroku

Create a Heroku PostgreSQL database, in the Heroku CLI (after logging in and creating a Heroku app):

```
heroku addons:create heroku-postgresql:hobby-dev --app APP_NAME_HERE
```

Obtain your new database url:

```
heroku config --app APP_NAME_HERE
```

Copy the url and paste into the production database uri in app.py. Change our ENV variable to 'prod'.

Create Procfile:

```
touch Procfile
```

And enter the following code:

```
web: gunicorn app:app
```

Create a runtime.txt file and add:

```
python-3.7.2
```

Create requirements.txt:

```
pip freeze > requirements.txt
```

Create the Feedback table on the remote database