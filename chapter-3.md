# Chapter Three: User Authentication

In this chapter, we are going to create the logic to sign users up and log them in and out. There are a million ways to do this properly, however, in this book we will be taking a simple and not very secure approach for simplicity. 

We're going to be using mongoDB to store our users in the database and we will use the Mongoose ODM to create Schemas for our database. Schemas allow us to have pre-determined structure for the objects we store in our Mongo database. They can be very helpful.

As a prerequisite, you should have MongoDB installed. Let's get mongo up and running and create a database. First, let's go to the command line and start the mongo server by typing the command

```
$ mongod
all output going to: /usr/local/var/log/mongodb/mongo.log
```

Then to create the database go to the command line and type in mongo to enter the MongoDB shell. From here we can create a database, lets call it "nulltonode".

```bash
> 
```

We are going to use the basic-auth-mongoose library from github to log users in and out so let's add it to our package.json file.

"basic-auth-mongoose": "0.1.x",



Since we are going to be updloading files, we need to tell our app where to put the files when they are uploaded. In app.js let's do that now.

```javascript

```

## The Authentication routes

First thing we need to do is create a couple routes, one we already have is home, this is where the user will sign up. The other two are login and logout.

```javascript
app.get('/', routes.home);
app.get('/login', routes.login);
app.get('/logout', routes.logout);
```

And again, lets add the corresponding routes to our routes.js file.

```javascript
exports.home = function (request, response) {
  response.render('home');
}

exports.login = function (request, response) {
  response.render('login');
}

exports.logout = function (request, response) {
  response.render('logout');
}
```

We don't have the templates that these routes are going to try to render so let's make `login.handlebars` and `logout.handlebars` in our views directory.

## Sign Up Page

Let's start off by getting our sign up page going. Here we want the user to be able to come to the home page and sign up for an account by filling out a form. Inside of home.handlebars lets create a form.

```html
<h1>Welcome</h1>
<p>To get started, fill out the sign up form below.</p>
<hr>
<p class="lead">Sign Up</p>
{{#if formError }}
  <p class="alert alert-warning">{{{ formError }}}</p>
{{/if}}
<form action="" method="POST" enctype="multipart/form-data">
  <input type="text" name="name" value="" placeholder="Name"><br/>
  <input type="text" name="username" value="" placeholder="username"><br/>
  <input type="password" name="password" value="" placeholder="password"><br/>
  <p>Profile Image</p><input type="file" name="image"><hr>
  <input class="btn btn-primary" type="submit" name="" value="Sign Up">
</form>
```

In this form, we have an input for name, username, and password along with an file input for the user's profile image. Notice that the form method is set to `POST`, this will tell our form to post data to the action url when the form is submitted. In this case our action is blank wich will post to the current location. Another thing to note is `enctype="multipart/form-data"` which will ensure that our image file will be posted to the server.

When the form is submitted its going to post data to our home route. So lets go there and set things up to recieve this data. Our basic route is going to be structured like this.

```javascript
exports.home = function (request, response) {
  // Only attempt to process form data if the request method is POST
  if (request.method == 'POST') {
    // process form data
  }
  else {
    response.render('home', { layout: 'base' });
  }
}
```

When a user visits the page it will send a GET request, but when they submit the form it will send a POST request, so we check for that and process the form if thats the case. Next we need to add some logic to process our form data.

Next, inside our if body lets grab everything from the form that we need.

```javascript
...

if (request.method == 'POST') {
  // Store the form data in some variables
  var name        = request.body.name;
  var username    = request.body.username;
  var password    = request.body.password;
  var tmp_path    = request.files.image.path;

  // Set the directory where we want to store the images. 
  // Make sure to create this directory or it will throw an error.
  var target_path = './public/uploads/' + request.files.image.name;
}

...
```

First we are grabbing the name, username, and password from response.body where normal form data is stored. We also grab `request.files.image.path` and store it which is where a path to a temporary file has been saved in our uploads directory.






[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md)
