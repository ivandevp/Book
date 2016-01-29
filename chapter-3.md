# Chapter Three: User Models and Authentication

## Creating a MongoDB Database

In this chapter, we are going to create the logic to sign users up and log them in and out. There are a million ways to do this properly, however, in this book we will be taking a simple and not very secure approach for simplicity. 

We're going to be using mongoDB to store our users in the database and we will use the Mongoose ODM to create Schemas for our database. Schemas allow us to have pre-determined structure for the objects we store in our Mongo database. They can be very helpful.

As a prerequisite, you should have MongoDB installed. Let's get mongo up and running and create a database. First, let's go to the command line and start the mongo server by typing the command.

```
$ mongod
```

This will print out information as MongoDB launches. You know it was successful if it prints:

`waiting for connections on port 27017`

This command must continue to run in order for your application to access MongoDB. So open another command line tab or window so we can run additional commands.

Then to create the database we need to enter the MongoDB shell.  We do that with the following command:

```bash
$ mongo
MongoDB shell version: 2.6.4
connecting to: test
> 
```

From here we can create a database, lets call it "nulltonode". (NOTE: Instead of the $ sign, you will notice this command has the > sign. This represents the MongoDB shell.)

```bash
> use nulltonode
switched to db nulltonode
```

### Checkpoint!

Type the following command into the MongoDB Shell:

```
> show dbs
```

If you have been successful up to this point, then a list of databases should print out. Make sure this list includes `nulltonode` (It might be the only one listed). This is the database where we will save all data for our application.

## Installing Mongoose and Dependencies

For user authentication we are going to use the [basic-auth-mongoose](https://github.com/thauburger/basic-auth-mongoose) library from github to log users in and out. We're also using mongoDB and the Mongoose ODM so let's add them all to our package.json file in the dependencies object. (Don't forget: in the dependencies object, a comma must be at the end of every line except the last one.)

```javascript
...

"basic-auth-mongoose": "0.1.x",
"mongoose": "4.3.x",
"mongodb": "2.1.x",
"body-parser": "1.14.x
<!-- "underscore": "1.8.x" -->
<!-- "cookie-parser": "1.4.x -->

...
```

<!-- Also we're going to use [underscore](http://underscorejs.org/) for some things so we've added that to the dependencies as well. -->

Now that we've changed our package.json file, we need to run an install. Always remember to do this when package.json changes.

```
$ npm install
```

### Checkpoint!

You should now be able to find directories for every package we told node to download (`basic-auth-mongoose` and all the others we just typed into `package.json`) inside of `node_modules`.

## Creating Mongoose Models

Ok, now that we have a Mongo database set up, we need to get some Mongoose Models created so that we can store data in structured objects. Create a file in the root of your project and name it models.js and let's add a couple imports at the top.

```javascript
// Import Mongoose and connect it to our local database.
var mongoose = require('mongoose');
var db       = mongoose.connect('localhost', 'nulltonode');
// Import the mongoose Schema object so we can create some database schemas
var Schema   = mongoose.Schema;
```

Here we are importing mongoose and initializing a connection to our database. We're also importing `mongoose.Schema` which we will use to create models of our data. Just after our import, let's add the following code.

```javascript
...

var UserSchema = new Schema();

UserSchema.add({
  name:      { type: String, required: true },
  username:  { type: String, required: true, lowercase: true, trim: true, index: { unique: true } },
  image:     { type: String },
  // following and followers are one to many relationships between users.
  following: [],
  followers: [],
});

UserSchema.plugin(require('basic-auth-mongoose'));
exports.User = db.model('User', UserSchema);
```

Above, we are defining `UserSchema` which is like a blueprint for creating users in the database. Every time we create a user we will use this blueprint and mongoose will know how we want our data structured. We also have enabled the basic auth plugin for our schema and exported it as `User` so we can use it from other files.

Now, let's build the functionality to handle when a user signs up.

## Saving To The Database

When the form is submitted it's going to post data to our home route. So let's go there and set things up to receive this data. Our basic route is going to be structured like this: (in routes.js)

```javascript
exports.home = function (request, response) {
  // Only attempt to process form data if the request method is POST
  if (request.method == 'POST') {
    // process form data
  }
  else {
    response.render('home', { layout: 'main' });
  }
}
```

When a user visits the page it will send a GET request, but when they submit the form it will send a POST request, so we check for that and process the form if that's the case. Next we need to add some logic to process our form data.

Why did we change: response.render('home');
to: response.render('home', { layout: 'main' });
Didn't we set the default layout to main so it would use it without this addition? (in app.js)
Yes, we did. Still it is good to know we can toss in a new layout this way if desired.

Next, inside our `if` body lets grab everything from the form that we need.

```javascript
...

if (request.method == 'POST') {
  // Store the form data in some variables
  var name        = request.body.name;
  var username    = request.body.username;
  var password    = request.body.password;
<!--   var tmp_path    = request.files.image.path;

  // Set the directory where we want to store the images. 
  // Make sure to create this directory or it will throw an error.
  var target_path = './public/uploads/' + request.files.image.name; -->
}

...
```

We are grabbing the name, username, and password from response.body where normal form data is stored. Notice that we are not grabbing the user's profile photo. We will build that functionality in later because it involves a few extra steps. First let's get the route working.

Now that we have the user's information, let's save it to the database.

```javascript
...

// Check that the username and password are acceptable.
if (username.length >= 3 && password.length >= 6) {
  // Create a new user using our user schema
  var newUser = new models.User({
    // Pass in the data from our form.
    name: name,
    username: username,
    password: password
    <!-- image: target_path -->
  });

  // Attempt to save the new user.
  newUser.save(function (error, user) {
    if (error) {
      // If the save fails, it may be because the username is already taken so return an error.
      response.render('home', { layout: 'main', formError: 'Sorry, that username is already taken.' });
    }
    else {
<!--       // Store the user id in a session cookie so we know the user is logged in now.
      request.session.userID = user._id; -->
      // The user was successfully saved.
      console.log('The user was successfully saved.');
    }
  });
}
else {
  // If the username and password validation failed, return an error.
  response.render('home', { layout: 'main', formError: 'Your username must contain at least 3 characters.<br/> Your password must contain at least 6 characters.' });
}

...
```

The commenting in this code can shed some light on what is happening exactly. Basically we make sure we have everything we need to create a user, then we create that user. Otherwise we return an error to the home page and ask the user to correct the form. Also, since we are using the User model in this code we need to import the file (models.js) where the model is located at the top of our routes.js file

```javascript
var models = require('./models');
```

One last thing. We have to tell express how to read the data that is arriving when our form POSTs. We do this with `body-parser`, which you may have noticed that we included in our last update to package.json. So, in app.js, first `require` body-parser at the top of the file, underneath all of our previous requirements.

```
var bodyParser = require('body-parser');

```

Then, tell our app to use `bodyParser`.

```
...
app.engine('handlebars', hbs({defaultLayout: 'main'}));
app.set('view engine', 'handlebars');
app.use('/public', express.static('public'));

app.use(bodyParser.urlencoded({ extended: true }))
...

```

Basically, data passed from a form can be encoded in different ways. Ours is currently being urlencoded. You can see that we called the urlencoded function on bodyParser. bodyParser now knows how to parse the data and create the request.body object we are pulling the user's data from in our home route. Don't worry too much if this doesn't quite make sense yet.

### Checkpoint!

Ok, we came a long way in that section! We built a mongoose model to control the structure of the data we are saving to the database. Then we wrote code to handle the POST request when the user submits the sign up form, which saves the user to the database by using our mongoose model. And lastly, we told the app how to read the data being passed by the form so it can give the home function access to said data by using body-parser. Let's see if it all worked.

First, make sure node is restarted in command line.

```
$ node app.js

```

Then, in the browser at <a href="http://localhost:3000">http://localhost:3000</a>, Fill out the sign up form, but DO NOT upload an image. Remember that in the home function we set parameters saying the username must be at least 3 characters long, and the password must be at least 6 long.

When you press sign up, nothing should happen. However, if you go back to the command line where node is running your app.js file, it should now say: 

`The user was successfully saved.`

We just saved our first user to the database!

If you want to, you can go to the command line window where you are running the MongoDB Shell (where you typed the mongo command) and type in:


```
> db.users.findOne()
```

This should print out the database object that we just saved. You may notice that instead of `password`, there is `pass_hash` and `salt`. This is a security feature provided by the `basic-auth-mongoose` plugin that we attached to our `UserSchema` in `models.js`. It is very bad to store passwords in plain text in a database. `basic-auth-mongoose` scrambles them so if someone hacked our database they still would not be able to easily read the passwords.

## Saving the Uploaded File

Ok, now that we are saving the username and password, we now need to make sure we save the uploaded profile photo. This is a bit complicated and requires us to change a few things we have already done, but we will get through it!


<!-- 



First we're grabbing the name, username, and password from response.body where normal form data is stored. We also grab `request.files.image.path` and store it which is where a path to a temporary file has been saved in our uploads directory. We also set the path on our server where we will store the image that the user uploads.

At this point you need to make sure you have an uploads directory inside your public directory or you will run into problems because node can't create files in directories that do not exist.


Now, since we're going to be uploading files (allowing the user to save a profile picture), we need to tell our app where to put the files when they are uploaded. We will use body-parser for this. First, we need to  In app.js add this above the app.get('/', routes.home); line:

```javascript
...

// Upload directory for our images
app.use(express.bodyParser({uploadDir:'./public/uploads'}));
// Cookie parser for our authentication library.
app.use(cookieParser());
app.use(express.cookieSession({ secret: 'secret123' }));

...
```

Also, we added in a few lines to enable the express cookieParser so that we can store user sessions. You can read more about this feature [here](http://expressjs.com/api.html#cookieParser). But you don't need to worry too much about it for now. 

The ./ that we used to specify where the uploadDir will be just means the directory you are currently at. In this case it is the root directory.

## The Authentication routes

First thing we need to do is create a couple routes, one we already have is home, this is where the user will sign up. The other two are login and logout. In app.js you can remove the app.get('/', routes.home); line and replace it with the following:

```javascript
// Middleware function to add the user to the request object.
app.use(function(request, response, next) {
  if (request.session.userID) {
    models.User.findOne({'_id': request.session.userID}, function(err, user) {
      if (!err) request.user = user;
      next();
    });
  }
  else {
    next();
  }
});

app.all('/', routes.home);
app.all('/login', routes.login);
app.get('/logout', routes.logout);
```

Since we added a models call here we need to add this to the top of app.js under the express3-handlebars include

```javascript
var models = require('./models');
```

We also created a middleware function that will be called during every request. The middleware function will add the current logged in users ID to the request object that is available in every route so we have access to it. 

Now let's create the corresponding routes. In your routes.js file add the login and logout routes under the home route.

```javascript

exports.login = function (request, response) {
  response.render('login');
}

exports.logout = function (request, response) {
  response.render('logout');
}
```

We don't have the templates that these routes are going to try to render so let's make `login.handlebars` and `logout.handlebars` in our views directory.

## Sign Up Page


----------------------------------


In the same block, let's add the logic needed to save the user's image to the uploads directory. First add the import for 


```javascript
...

// Check to see if the user uploaded a profile image.
if (tmp_path) {
  // If the tmp file exists lets move it to a permanent location.
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
  // It appears no image was supplied so let's return an error and tell the user to provide one.
  response.render('home', { layout: 'main', formError: 'You must upload a profile image.' });
}

...
```

We check to see if the user supplied an image by checking for `tmp_path`. Then we move the temp file to our new target location and delete the old file. If `tmp_path` is not present then we can assume the user did not supply a profile image so we will return an error to show on our home page.

Here we're using the file system package to save the file to the uploads directory using the `tmp_path` and `target_path`. For this to work we need to import `fs` into our routes file. At the top of the file add it.

```javascript
var fs = require('fs');
```


So now putting everything together, our routes file should look like this.

```javascript
var models = require('./models');
var fs = require('fs');

exports.home = function (request, response) {
  // Only attempt to process form data if the request method is POST
  if (request.method == 'POST') {
    // Store the form data in some variables
    var name        = request.body.name;
    var username    = request.body.username;
    var password    = request.body.password;
    var tmp_path    = request.files.image.path;

    // Set the directory where we want to store the images. 
    // Make sure to create this directory or it will throw an error.
    var target_path = './public/uploads/' + request.files.image.name;

    // Check to see if the user uploaded a profile image.
    if (tmp_path) {
      // If the tmp file exists lets move it to a permanent location.
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
      // It appears no image was supplied so let's return an error and tell the user to provide one.
      response.render('home', { layout: 'main', formError: 'You must upload a profile image.' });
    }

    // Check that the username and password are acceptable.
    if (username.length >= 3 && password.length >= 6) {
      // Create a new user using our user schema
      var newUser = new models.User({
        // Pass in the data from our form.
        name: name,
        username: username,
        password: password,
        image: target_path
      });

      // Attempt to save the new user.
      newUser.save(function (error, user) {
        if (error) {
          // If the save fails, it may be because the username is already taken so return an error.
          response.render('home', { layout: 'main', formError: 'Sorry, that username is already taken.' });
        }
        else {
          // Store the user id in a session cookie so we know the user is logged in now.
          request.session.userID = user._id;
          // The user was successfully saved.
          console.log('The user was successfully saved.');
        }
      });
    }
    else {
      // If the username and password validation failed, return an error.
      response.render('home', { layout: 'main', formError: 'Your username must contain at least 3 characters.<br/> Your password must contain at least 6 characters.' });
    }
  }
  else {
    // No form was submitted, render the home page.
    response.render('home', { layout: 'main' });
  }
}

exports.login = function (request, response) {
  response.render('login');
}

exports.logout = function (request, response) {
  response.render('logout');
}
```

Now since the user is logged in we need to give them a way to logout. The code to log a user out is pretty simple.

```javascript
exports.logout = function (request, response) {
  // Delete the session and log the user out then return them to the home page.
  delete request.session.userID;
  console.log('Successfully logged out');
  response.redirect('/');
}
```

So now if you navigate to [localhost:3000/logout](http://localhost:3000/logout) you will be logged out. You can check the terminal where your app is running to make sure it worked.

Also, we want to make it so the user can log back in with their existing credentials. Let's start with the login view by creating login.handlebars within the views directory.

```html
<p class="lead">Log In</p>
{{#if formError }}
<p class="alert alert-warning">{{{ formError }}}</p>
{{/if}}
<form action="" method="POST" accept-charset="utf-8">
  <input type="text" name="username" value="" placeholder="username"><br/>
  <input type="password" name="password" value="" placeholder="password"><br/>
  <input class="btn btn-primary" type="submit" name="" value="Log In">
</form>
```

The login route and view should seem somewhat similar to the home route.

```javascript
exports.login = function (request, response) {
  // If the user submitted the login form, attempt to log them in.
  if (request.method == 'POST') {
    // Grab the username and password from the post data. 
    var username = request.body.username;
    var password = request.body.password;

    // Query the database for the user we are looking for. If they exist try to log them in, otherwise return an error.
    var user = models.User.findOne({'username': username }, function(error, user) {
      if (error) response.render('login', { layout: 'main', formError: 'An error occurred. Please try again.' });
    
      // Check the password that was supplied.
      if ( user && user.authenticate(password) ) {
        request.session.userID = user.id;
        console.log('Successfully logged in as '+ user.name);
        response.redirect('/');
      }
      else {
        // The user supplied an incorrect username/password
        response.render('login', { layout: 'main', formError: 'Your login details were incorrect. Please try again.' });
      }
    });
  }
  else {
    // No form was submitted, just show the login page.
    response.render('login', { layout: 'main' });
  }  
}
```

Now if we click the "Log In" link, we see the login page and form. Type in your test username and password, hit the "Log In" button and you'll be logged in. Again, check terminal to make sure everything worked alright.

In the next chapter we will start building all of the routes and views for logged in users. Right now we can log in and out which is awesome, but we really don't get to do much more than that. The good thing is that this is really the hardest part of setting up our site. From here on out we will be building some cool functionality and really starting to see things come together.

[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md) -->

