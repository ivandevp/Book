# Chapter Three
## User Models and Authentication

In this chapter, we are going to create the logic to sign users up and log them in and out. There are a million ways to do this properly, however, in this book we will be taking a simple and not very secure approach for simplicity. 

We're going to be using mongoDB to store our users in the database and we will use the Mongoose ODM to create Schemas for our database. Schemas allow us to have pre-determined structure for the objects we store in our Mongo database. They can be very helpful.

As a prerequisite, you should have MongoDB installed. Let's get mongo up and running and create a database. First, let's go to the command line and start the mongo server by typing the command

```
$ mongod
all output going to: /usr/local/var/log/mongodb/mongo.log
```

Then to create the database go to the command line and type in mongo to enter the MongoDB shell. 

```bash
$ mongo
MongoDB shell version: 2.4.3
connecting to: test
> 
```

From here we can create a database, lets call it "nulltonode".

```bash
> use nulltonode
switched to db nulltonode
```

For user authentication we are going to use the [basic-auth-mongoose](https://github.com/thauburger/basic-auth-mongoose) library from github to log users in and out. We're also using mongoDB and the Mongoose ODM so let's add them all to our package.json file.

```javascript
...

"basic-auth-mongoose": "0.1.x",
"mongoose": "3.6.x",
"mongodb" : "1.1.11"
"underscore": "1.5.1"

...
```

Also we're going to use [underscore](http://underscorejs.org/) for some things so we've added that to the dependencies as well.

Now that we've changed our package.json file, we need to run `npm install`. Always remember to do this when package.json changes.

Ok, now that we have a Mongo database set up, we need to get some Mongoose Models created so that we can store data in structured objects. Create a file in the root of your project and name it models.js and let's add a couple imports at the top.

```javascript
// Import Mongoose and connect it to our local database.
var mongoose = require('mongoose');
var db       = mongoose.connect('localhost', 'nulltonode');
// Import the mongoose Schema object so we can create some database schemas
var Schema   = mongoose.Schema;
```

Here we are importing mongoose and initializing a connection to our database. We're also importing `mongoose.schema` which we will use to create models of our data. Just after our import, let's add the following code.

```javascript
...

var UserSchema = new Schema({
  name:      { type: String, required: true },
  username:  { type: String, required: true, lowercase: true, trim: true, index: { unique: true } },
  image:     { type: String },
  // following and followers are one to many relationships between users.
  following: [ UserSchema ],
  followers: [ UserSchema ],
});

UserSchema.plugin(require('basic-auth-mongoose'));
exports.User = db.model('User', UserSchema);
```

Above, we are defining `UserSchema` which is like a blueprint for creating users in the database. Every time we create a user we will use this blueprint and mongoose will know how we want our data structured. We also have enabled the basic auth plugin for our schema and exported it as `User` so we can use it from other files.



Now, since we're going to be uploading files, we need to tell our app where to put the files when they are uploaded. In app.js:

```javascript
...

// Upload directory for our images
app.use(express.bodyParser({uploadDir:'./public/uploads'}));
// Cookie parser for our authentication library.
app.use(express.cookieParser());
app.use(express.cookieSession({ secret: 'secret123' }));

...
```

Also, we added in a few lines to enable the express cookieParser so that we can store user sessions. You can read more about this feature [here](http://expressjs.com/api.html#cookieParser). But you don't need to worry too much about it for now.

## The Authentication routes

First thing we need to do is create a couple routes, one we already have is home, this is where the user will sign up. The other two are login and logout. In app.js let's add the following:

```javascript
app.get('/', routes.home);
app.get('/login', routes.login);
app.get('/logout', routes.logout);
```

And again, let's add the corresponding routes to our routes.js file.

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

Let's start off by getting our sign up page going. Here we want the user to be able to come to the home page and sign up for an account by filling out a form. Inside of home.handlebars let's create a form.

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

If the form submission returns an error, the error will be shown above the form.

```html
{{#if formError }}
  <p class="alert alert-warning">{{{ formError }}}</p>
{{/if}}
```

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

First we're grabbing the name, username, and password from response.body where normal form data is stored. We also grab `request.files.image.path` and store it which is where a path to a temporary file has been saved in our uploads directory. We also set the path on our server where we will store the image that the user uploads.

At this point you need to make sure you have an uploads directory inside your public directory or you will run into problems because node can't create files in directories that do not exist.

In the same block, let's add the logic needed to save the user's image to the uploads directory. First add the import for 


```javascript
...

if (tmp_path) {
  fs.rename(tmp_path, target_path, function(err) {
    if (err) throw err;
    // delete the temporary file, so that the explicitly set temporary upload dir does not get filled with unwanted files
    fs.unlink(tmp_path, function() {
      if (err) throw err;
      console.log('profile image successfully saved.');
    });
  });
}
else {
  response.render('home', { layout: 'base', formError: 'You must upload a profile image.' });
}

...
```

We check to see if the user supplied an image by checking for `tmp_path`. Then we move the temp file to our new target location and delete the old file. If `tmp_path` is not present then we can assume the user did not supply a profile image so we will return an error to show on our home page.

Here we're using the file system package to save the file to the uploads directory using the `tmp_path` and `target_path`. For this to work we need to import `fs` into our routes file. At the top of the file add it.

```javascript
var fs = require('fs');
```
-'
''
Next we want to handle the name, username, and password.\\=

```javascript
...

// Check that the username and password are acceptable.
if (username.length >= 3 && password.length >= 6) {
  // Create a new user
  var newUser = new models.User({
    name: name,
    username: username,
    password: password,
    image: target_path
  });

  // Save the new user.
  newUser.save(function (error, user) {
    if (error) {
      response.render('home', { layout: 'base', formError: 'Sorry, that username is already taken.' });
    }
    else {
      request.session.userID = user._id;
      response.redirect('/feed');
    }
  });
}
else {
  response.render('home', { layout: 'base', formError: 'Your username must contain at least 3 characters.<br/> Your password must contain at least 6 characters.' });
}

...
```






[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md)
