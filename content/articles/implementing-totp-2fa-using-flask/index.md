---
layout: engineering-education
status: publish
published: true
url: /implementing-totp-2fa-using-flask/
title: Implementing TOTP 2FA in Python and Flask
description: In this tutorial we will learn about the concept of two-factor authentication and discussed different 2FA factors, including possession factor, biometric factor, and other factors.
author: prince-joel
date: 2021-04-27T00:00:00-10:00
topics: [Security, Languages]
excerpt_separator: <!--more-->
images:

  - url: /implementing-totp-2fa-using-flask/hero.jpg
    alt: 2FA in Python example image
---
Two-factor authentication (2FA) is a security protocol that protects users by asking them to verify their identity using two authentication methods. 
<!--more-->
In recent times, most organizations use 2FA techniques to ensure their user’s details and avoid the possibility of hackers gaining unauthorized access.

Two-factor authentication is setup using any of the following factors:
- Possession factor: This factor authenticates users using something only the user has, such as an ID card, mobile gadget for receiving OTP, or a security token.
- Biometric factor: This usually requires the users to be physically present during authentication. It may be in the form of fingerprints or facial recognition.
- Location and time factors: This usually checks the user’s current location using GPS or VPN.

### Prerequisites
To follow and fully understand this tutorial, you will need to have:
- Python 3.6 or a newer version.
- A text editor.
- A basic understanding of Flask.

### How two-factor authentication works
Essentially, the process of two-factor authentication involves the following procedure:
1. The user authenticates themselves using email and password (knowledge factor).
2. The platform confirms the user’s information and asks for a second authentication technique.
3. The platform generates a one-time password (OTP) and sends it to a device that only the user can access (possession factor).
4. The user provides the received OTP to the platform, which validates the information and authorizes the user.

### Importance of two-factor authentication
- It provides users with assured security of their accounts.
- It protects the platform from data breaches.
- It boosts customer’s confidence in an organization.
- It ensures only the intended user can assess the protected information.

### Time-based One-Time Password (TOTP)
Time-based One-Time Password (TOTP) is a common way of implementing two-factor authentication in applications. It works by asking the user for a token usually sent in an SMS, email, or a generated secret pass to the user’s device with an expiry time. It compares the provided token with the actual generated token, then authenticates them if the tokens match.

### Google Authenticator
Google Authenticator uses a software-based authentication technique made by Google that implements 2FA using TOTP and [HMAC-based](https://en.wikipedia.org/wiki/HMAC) one-time password (HOTP) for authenticating users of an application. It is part of [Open Authentication](https://oauth.net/2/) (OATH).

Hash-based message authentication code (HMAC) is a technique that uses hash functions and secret keys to calculate message authentication codes.

### How TOTP authenticator applications work
Essentially, the process of authenticating with authenticators involves the following procedure:
1. The website requests the user provide a one-time password generated by the authenticator application.
2. The website then generates another token using a seed value that both the authenticator application and itself know.
3. The website proceeds to authenticate the user if the newly generated token matches the token provided by the user.

### Implementing TOTP 2FA with Python and Flask
#### Installing required libraries
Two-factor authentication is commonly used in web applications to serve as an extra layer of security when users access a server. Using Python, let us build a Flask application and secure it with two-factor authentication using [Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator).

First, you must install the [Flask](https://palletsprojects.com/p/flask/) web framework, [Flask-Bootstrap](https://pypi.org/project/Flask-Bootstrap4/), and [PyOTP](https://pyauth.github.io/pyotp/) library, which you will use to build the server and implement two-factor authentication. 

In the terminal, type:

```bash
pip install flask
pip install pyotp
pip install flask-bootstrap4
```

#### Building a simple Flask server
You will write the code for setting up the Flask server. 

Start by creating a file named `app.py` and save the code below in it:

```python
# importing needed libraries
from flask import *
from flask_bootstrap import Bootstrap

# configuring flask application
app = Flask(__name__)
app.config["SECRET_KEY"] = "APP_SECRET_KEY"
Bootstrap(app)


# homepage route
@app.route("/")
def index():
    return "<h1>Hello World!</h1>"


# running flask server
if __name__ == "__main__":
    app.run(debug=True)
```

In the code above, you created a Flask server that renders the text `"Hello World!"` when the index page opens. You will get a response similar to the image below after running the server.

![Hello World](/implementing-totp-2fa-using-flask/hello-world.png)

#### One-factor authentication in Flask
Next we will write the code for authenticating users using a username and password. For simplicity’s sake, we will be hardcoding credentials the applications will match. 

Update the `app.py` file by adding the code below:

```python
# login page route
@app.route("/login/")
def login():
    return render_template("login.html")
```

You will also create a file named `login.html` that will be stored in the `templates` directory and save the following code in it:

```html
{% extends "bootstrap/base.html" %}

{% block content %}
<div class="container">
  <div class="row justify-content-center">
    <div class="col-lg-12">
      <div class="jumbotron text-center p-4">
        <h2>Flask + 2FA Demo</h2>
      </div>
    </div>
    <div class="col-lg-6">
      {% with messages = get_flashed_messages(with_categories=true) %}
      {% if messages %}
      {% for category, message in messages %}
      <div class="alert alert-{{ category }}" role="alert">
        {{ message }}
      </div>
      {% endfor %}
      {% endif %}
      {% endwith %}
      <form method="POST">
        <div class="form-group">
          <label for="username">Username</label>
          <input type="text" class="form-control" id="username" name="username" required>
        </div>
        <div class="form-group">
          <label for="password">Password</label>
          <input type="password" class="form-control" id="password" name="password" required>
        </div>
        <div class="text-center">
          <button type="submit" class="btn btn-primary">Authenticate User</button>
        </div>
      </form>
    </div>
  </div>
</div>
{% endblock %}
```

![Login](/implementing-totp-2fa-using-flask/login.png)

You will also write a route to handle `POST` requests made to the login page and authenticate them. 

Update the `app.py` file by adding the code below:

```python
# login form route
@app.route("/login/", methods=["POST"])
def login_form():
    # demo creds
    creds = {"username": "test", "password": "password"}

    # getting form data
    username = request.form.get("username")
    password = request.form.get("password")

    # authenticating submitted creds with demo creds
    if username == creds["username"] and password == creds["password"]:
        # inform users if creds are valid
        flash("The credentials provided are valid", "success")
        return redirect(url_for("login"))
    else:
        # inform users if creds are invalid
        flash("You have supplied invalid login credentials!", "danger")
        return redirect(url_for("login"))
```

You should get an image similar to the one below when invalid credentials are used in the form:

![invalid login](/implementing-totp-2fa-using-flask/invalid-login.png)

And an image similar to the one below when valid credentials are used:

![valid login](/implementing-totp-2fa-using-flask/valid-login.png)

> Please note that the valid credentials created for the application are `username: test` and `password: password`.

#### TOTP 2FA authentication in Python
To generate TOTPs using [PyOTP](https://pyauth.github.io/pyotp/), you need to instantiate the `TOTP` class of the PyOTP library and call the `now` method.

Here is a sample Python code that demonstrates this functionality:

```python
import pyotp

# generating TOTP codes with provided secret
totp = pyotp.TOTP("base32secret3232")
print(totp.now())
```

You can proceed to validate generated tokens using the `verify` method. 

You can do this with the code below:

```python
import pyotp

# verifying TOTP codes with PyOTP
totp = pyotp.TOTP("base32secret3232")
print(totp.verify("492039"))
```

You can generate and validate Counter-based OTPs using the code below:

```python
import pyotp

# generating HOTP codes with PyOTP
hotp = pyotp.HOTP("base32secret3232")
print(hotp.at(0))
print(hotp.at(1))
print(hotp.at(1401))

# verifying HOTP codes with PyOTP
print(hotp.verify("316439", 1401))
print(hotp.verify("316439", 1402))
```

PyOTP also provides a helper library to generate secret keys to initiate the `TOTP` and `HOTP` classes. 

You can do this with the following code:

```python
import pyotp

# generating random PyOTP secret keys
print(pyotp.random_base32())
```

You might want the secret key formatted as a hex-encoded string:

```python
import pyotp

# generating random PyOTP in hex format
print(pyotp.random_hex()) # returns a 32-character hex-encoded secret
```

#### TOTP 2FA authentication in Flask
You will write the code to provide users with the page to set up TOTP 2FA. Start by updating the `login` route in the `app.py` file to redirect users to the 2FA page after successful authentication.

```python
# redirecting users to 2FA page when creds are valid
if username == creds["username"] and password == creds["password"]:
    return redirect(url_for("login_2fa"))
```

You will also create the `login_2fa` route that will be responsible for handling TOTP 2FA. 

Add the following code to the `app.py` file:

```python
# 2FA page route
@app.route("/login/2fa/")
def login_2fa():
    # generating random secret key for authentication
    secret = pyotp.random_base32()
    return render_template("login_2fa.html", secret=secret)
```

You will also create a file named `login_2fa.html` that will be stored in the `templates` directory and save the following code in it:

```html
{% extends "bootstrap/base.html" %}

{% block content %}
<div class="container">
  <div class="row justify-content-center">
    <div class="col-lg-12">
      <div class="jumbotron text-center p-4">
        <h2>Flask + 2FA Demo</h2>
        <h4>Setup and Authenticate 2FA</h4>
      </div>
    </div>
    <div class="col-lg-5">
      <form>
        <div>
          <h5>Instructions!</h5>
          <ul>
            <li>Download <a href="https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en&gl=US" target="_blank">Google Authenticator</a> on your mobile.</li>
            <li>Create a new account with <strong>setup key</strong> method.</li>
            <li>Provide the required details (name, secret key).</li>
            <li>Select time-based authentication.</li>
            <li>Submit the generated key in the form.</li>
          </ul>
        </div>
        <div class="form-group">
          <label for="secret">Secret Token</label>
          <input type="text" class="form-control" id="secret" value="{{ secret }}" readonly>
        </div>
        <div class="text-center">
          <button type="button" class="btn btn-primary" onclick="copySecret()">Copy Secret</button>
        </div>
      </form>
    </div>
    <div class="col-lg-7">
      {% with messages = get_flashed_messages(with_categories=true) %}
      {% if messages %}
      {% for category, message in messages %}
      <div class="alert alert-{{ category }}" role="alert">
        {{ message }}
      </div>
      {% endfor %}
      {% endif %}
      {% endwith %}
      <form method="POST">
        <div class="form-group">
          <label for="otp">Generated OTP</label>
          <input type="hidden" name="secret" value="{{ secret }}" required>
          <input type="number" class="form-control" id="otp" name="otp" required>
        </div>
        <div class="text-center">
          <button type="submit" class="btn btn-primary">Authenticate User</button>
        </div>
      </form>
    </div>
  </div>
</div>

<script>
  function copySecret() {
    /* Get the text field */
    var copyText = document.getElementById("secret");

    /* Select the text field */
    copyText.select();
    copyText.setSelectionRange(0, 99999); /*For mobile devices*/

    /* Copy the text inside the text field */
    document.execCommand("copy");

    alert("Successfully copied TOTP secret token!");
  }
</script>
{% endblock %}
```

![Generating OTP](/implementing-totp-2fa-using-flask/generating-otp.png)

You will also write a route to handle `POST` requests made to the 2FA page and authenticate them. 

Update the `app.py` file by adding the code below:

```python
# 2FA form route
@app.route("/login/2fa/", methods=["POST"])
def login_2fa_form():
    # getting secret key used by user
    secret = request.form.get("secret")
    # getting OTP provided by user
    otp = int(request.form.get("otp"))

    # verifying submitted OTP with PyOTP
    if pyotp.TOTP(secret).verify(otp):
        # inform users if OTP is valid
        flash("The TOTP 2FA token is valid", "success")
        return redirect(url_for("login_2fa"))
    else:
        # inform users if OTP is invalid
        flash("You have supplied an invalid 2FA token!", "danger")
        return redirect(url_for("login_2fa"))
```

You should get an image similar to the one below when an invalid token is provided:

![invalid OTP](/implementing-totp-2fa-using-flask/invalid-otp.png)

You should get an image similar to the one below when a valid token is provided:

![valid OTP](/implementing-totp-2fa-using-flask/valid-otp.png)

### Conclusion
In this article, we learned the concept of two-factor authentication and discussed different 2FA factors, including possession factor, biometric factor, and other factors. 

We also highlighted the importance of implementing 2FA into applications and integrated two-factor authentication using Google Authenticator and PyOTP into a Flask application.

Looking to develop the two-factor authentication application further, improve the functionality or check out example code? Check out the [GitHub Repo](https://github.com/prince-joel/totp-section).

Happy coding.

### Resources
- [Multi-Factor Authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication)
- [PyOTP](https://pyauth.github.io/pyotp/)
- [Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator)

---
Peer Review Contributions by: [Saiharsha Balasubramaniam](/authors/saiharsha-balasubramaniam/)