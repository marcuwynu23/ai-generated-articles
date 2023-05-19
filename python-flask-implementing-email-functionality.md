# Implementing Email Functionality in Flask

Email functionality is an essential part of many web applications, and Flask provides built-in support for sending emails. In this article, we will look at how to implement email functionality in Flask.

### Setting up Flask-Mail
Flask-Mail is a popular extension for sending emails in Flask. To use Flask-Mail, you need to install it using pip:

```bash
pip install Flask-Mail
```

Next, you need to configure Flask-Mail in your Flask application:

```python
from flask_mail import Mail

app = Flask(__name__)
app.config['MAIL_SERVER'] = 'smtp.gmail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = 'your-email@example.com'
app.config['MAIL_PASSWORD'] = 'your-email-password'

mail = Mail(app)
```

### Sending Emails

With Flask-Mail set up, you can use it to send emails in your Flask application. Here's an example of sending a simple email:

```python
from flask_mail import Message

@app.route('/send-email')
def send_email():
    message = Message('Hello', recipients=['recipient@example.com'])
    message.body = 'This is a test email'
    mail.send(message)
    return 'Email sent!'
```

In this example, we create a Message object with a subject and a recipient, set the body of the email, and then send the email using the mail object.

### Templated Emails

You can also send templated emails in Flask using the render_template function. Here's an example:

```python
from flask import render_template

@app.route('/send-templated-email')
def send_templated_email():
    message = Message('Hello', recipients=['recipient@example.com'])
    message.html = render_template('email.html', name='John')
    mail.send(message)
    return 'Email sent!'
```

In this example, we use the render_template function to render an HTML email template with a variable name. We then set the html attribute of the Message object to the rendered template.

### Conclusion

Flask-Mail provides an easy way to send emails in Flask applications. With Flask-Mail, you can send simple text emails or templated HTML emails with ease. By following the examples above, you can add email functionality to your Flask application in no time.

