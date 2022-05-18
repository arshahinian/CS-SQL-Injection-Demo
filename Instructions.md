Activity (iam-2-sql-injection-demo)

# Activity: SQL Injection Demo
In order to take a closer look at SQL injection attacks, we're going to mount one ourselves. This activity will allow us to mimic a SQL injection attack using a small app of our own creation. The app will consist of a simple log in and a mock user database on the back end that has purposefully been left vulnerable to a SQL injection attack.

## Project Setup
Let's get started by creating our directory and installing dependencies.

From the terminal, create a new folder called sql-injection-activity.
Within the folder, create the following files:
* app.js
* index.html
* style.css

We should create a .gitignore file as well, and ignore node_modules

We can now run git init and npm init -y from the root directory of our activity. This should initialize a Git repository and create a package.json and package-lock.json. Make sure your directory contains both before moving on.

Next, let's install the dependencies we'll need by running npm install to bring each one into our project. Our project is tiny, so there are only three dependencies!

* express
* sqlite3
* body-parser

## Starter Code: HTML
Now that we've got the foundation of our project ready, let's populate our project with the basic code we will need. Let's start with some HTML.

First, let's create an HTML document skeleton within our index.html.
Next, go ahead and link our style.css within the head tag.
Now, within the body of the document:
Create a form with the action set to "/login" and the method set to "post"

The form should contain:
A required text input labeled username
A required text input labeled password. Be sure to set the name attribute of this input to "password".

A Submit button

Within the form tag, wrap the newly created fields and button in a div with the class of container.

Within the form, underneath the container, let's add a script that will alert us to invalid log ins:
~~~
<script>
   if(document.location.hash === '#unauthorized') {
        document.write('<div class="unauthorized">Invalid Username or Password</div>');
   } else if(document.location.hash === '#error') {
        document.write('<div class="error">An error has occured</div>');
   }
 </script>
~~~
That's all the HTML we'll need. Let's next fill out the necessary JS in app.js.

## Starter Code: JavaScript
Once we've navigated to our app.js file, let's add the initial code we will need to create our server. For the sake of time, this code has been provided:
~~~
const http = require('http'),
path = require('path'),
express = require('express'),
bodyParser = require('body-parser');
const sqlite3 = require('sqlite3').verbose();
const app = express();
app.use(express.static('.'))
app.use(bodyParser.urlencoded({ extended: true }))
app.use(bodyParser.json())
~~~
Next, let's create the mock database that will hold our log-in information.

1. First, let's create a new sqlite database and populate it. Feel free to change the log-in information to something different.
~~~
const db = new sqlite3.Database(':memory:');
db.serialize(function () {
 db.run("CREATE TABLE user (username TEXT, password TEXT, title TEXT)");
 db.run("INSERT INTO user VALUES ('privilegedUser', 'privilegedUser1', 'Administrator')");
});
~~~

2. Next, we need to create a GET method route to '/' that will send our HTML file to the browser (hint: res.sendFile('index.html')).

3. We also need an Express POST route to '/login' that will handle any forms that are submitted via our HTML log-in form.
* We know that within the req.body of our POST request, we can find the username and password that were entered in the form.
* * Best practice would be to set those values to scoped variables within our POST callback function.
* Once we have our username and password, let's use them to create a SQL query to check their validity.
* * Declare a variable query and set it equal to a SQL query that utilizes the username and password we collected from the request object.
* * * It should look something like this:
SELECT title FROM user WHERE username = '" + username + "' AND password = '" + password + "';

### Remember! This SQL syntax is purposefully written badly for the purpose of our project. SQL queries should never be written this way in practice.

* Next, to see our SQL injection at work, go ahead and separately console.log() the username, password, and the SQL query string.
* For the final step of our POST request, let's run a sqlite method db.get() in order to verify our log in and handle errors in the event of invalid log in:
~~~
    db.get(query, function (err, row) {
	if (err) {
		console.log(â€˜ERRORâ€™, err);
		res.redirect(â€œ/index.html#errorâ€);
	} else if (!row) {
		res.redirect(â€œ/index.html#unauthorizedâ€);
	} else {
		res.send('Hello <b>' + row.title + '!</b><br /> This file contains all your secret data: <br /><br /> SECRETS <br /><br /> MORE SECRETS <br /><br /> 
		<a href=â€œ/index.htmlâ€>Go back to login</a>');
	}
});
~~~

4. Our last step is to call app.listen(), passing in a port of our choosing.

## Prepare the attack
Now that the code for our little log-in app is in place, go ahead and open it in the browser to make sure everything is working. Run node app.js in a terminal in the root of our project directory.

It should allow log in upon successful credential entry.
It should display an HTML error message upon invalid credential entry.
It should be receiving output in the terminal with the log-in information that was used.
Go ahead and play around with the functionality of the app for a moment. Perhaps even add a second user. Get familiar with what to expect in the terminal upon both a successful and unsuccessful log in.

Given what you already know about SQL, can you spot our vulnerability?
Hint: Check the query string!

## Mount the attack
In order to exploit the vulnerability in our SQL code, we're going to use a special password rather than the password we designated for our user.

Go ahead and log in again using the correct username but enter "unknown' OR '1'='1" as the password.

What happened?
We should have been logged in!

Taking your best guess, what do you think happened?
Hint: Take a look at the output in the terminal after using our new password. What do you notice?

Because of the way we crafted our SQL query and also because we are not sanitizing our input, we left open the possibility that a smart hacker could fit an additional clause into the SQL query that validates log ins.

## Acceptance Criteria
* When the proper command is run, the app runs successfully in the browser.
* When using chosen credentials, log in should be successful.
* When incorrect credentials are used, we should receive an error message in the console.
* When logging in using the SQL injection query string, we should be logged in successfully despite incorrect credentials.
Before submitting, make sure you do a self review of your code. Check the formatting and spelling, include comments in your code, and ensure you have a healthy commit history.

Make sure to submit your GitHub repository link on the submission page.