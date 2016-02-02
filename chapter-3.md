# Chapter Three: User Models and Authentication

## Creating a MongoDB Database

In this chapter, we are going to create the logic to sign users up and log them in and out. There are a million ways to do this properly, however, in this book we will be taking a simple and not very secure approach for simplicity. 

We're going to be using mongoDB to store our users in the database and we will use the Mongoose ODM to create Schemas for our database. Schemas allow us to have pre-determined structure for the objects we store in our Mongo database. They can be very helpful.

As a prerequisite, you should have MongoDB installed. Let's get mongo up and running and create a database. First, let's go to the command line and start the mongo server by typing the command.

```bash
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

```bash
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
"body-parser": "1.14.x"

...
```

Now that we've changed our package.json file, we need to run an install. Always remember to do this when package.json changes.

```bash
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
  var name     = request.body.name;
  var username = request.body.username;
  var password = request.body.password;
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
  });

  // Attempt to save the new user.
  newUser.save(function (error, user) {
    if (error) {
      // If the save fails, it may be because the username is already taken so return an error.
      response.render('home', { layout: 'main', formError: 'Sorry, that username is already taken.' });
    }
    else {
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

```javascript
var bodyParser = require('body-parser');

```

Then, tell our app to use `bodyParser` on our home route('/'). Change:

```javascript
app.all('/', routes.home);

```
to be:

```javascript
app.all('/', bodyParser.urlencoded({ extended: true }), routes.home);

```

Basically, data passed from a form can be encoded in different ways. Ours is currently being urlencoded. You can see that we called the urlencoded function on bodyParser. So now, when we POST to our home route, bodyParser will run its urlencoded function and create the request.body object. After this happens, our exports.home function in routes.js will run, where we access the request.body object built by bodyParser so we can save the user's sign up data. Don't worry too much if this doesn't quite make sense yet.

### Checkpoint!

Ok, we came a long way in that section! We built a mongoose model to control the structure of the data we are saving to the database. Then we wrote code to handle the POST request when the user submits the sign up form, which saves the user to the database by using our mongoose model. And lastly, we told the app how to read the data being passed by the form so it can give the home function access to said data by using body-parser. Let's see if it all worked.

First, make sure node is restarted in command line.

```bash
$ node app.js

```

Then, in the browser at <a href="http://localhost:3000">http://localhost:3000</a>, Fill out the sign up form, but DO NOT upload an image. Remember that in the home function we set parameters saying the username must be at least 3 characters long, and the password must be at least 6 long.

When you press sign up, nothing should happen. However, if you go back to the command line where node is running your app.js file, it should now say: 

`The user was successfully saved.`

We just saved our first user to the database!

If you want to, you can go to the command line window where you are running the MongoDB Shell (where you typed the mongo command) and type in:

```bash
> db.users.findOne()
```

This should print out the database object that we just saved. You may notice that instead of `password`, there is `pass_hash` and `salt`. This is a security feature provided by the `basic-auth-mongoose` plugin that we attached to our `UserSchema` in `models.js`. It is very bad to store passwords in plain text in a database. `basic-auth-mongoose` scrambles them so if someone hacked our database they still would not be able to easily read the passwords.

## Saving the Uploaded File

Ok, we are saving the username and password, but we also need to make sure we save the uploaded profile photo. This is a bit complicated and requires us to change a few things we have already done, but we will get through it!

First, the current way our form is encoding data (urlencoding) will not support sending an image file. So we need to go into home.handlebars and change the type of encoding by altering the form tag like this:

```html
<form action="" method="POST" enctype="multipart/form-data">
```

Next, we need to tell our app how to handle this new encoding type. We will do this by using <a href="https://github.com/expressjs/multer">multer</a>, which is a middleware specifically designed to handle multipart/form-data encoding.

You should know the process by now. First, add it to the list of our dependencies in package.json:

```javascript
"body-parser": "1.14.x",
"multer": "1.1.x"
```

Then run `npm install`.

Next, we must require it in app.js

```javascript
var bodyParser = require('body-parser');
var multer = require('multer');
var upload = multer({ dest: './public/uploads' });
```

Notice we also created an `upload` variable. In this, we call the multer function and pass it an object telling it the destination where we want to save the image files.

The ./ that we used to specify where the destination will be just means the directory you are currently at. In this case it is the root directory, because that is where app.js is saved.

At this point you need to make sure you have an uploads directory inside your public directory or you will run into problems because node can't create files in directories that do not exist.

Now, where we previously called bodyParser, we need to instead run multer. So change this:

```javascript
app.all('/', bodyParser.urlencoded({ extended: true }), routes.home);
```

to this:

```javascript
app.all('/', upload.single('image'), routes.home);
```

What we are doing is telling multer to upload our single file, named 'image' (we named the file 'image' in our form. See if you can find where in home.handlebars.). This then saves the file to the './public/uploads' directory, because that is where we told multer to put it when we declared the `upload` variable. While multer is doing this, it also creates the request.body object so we still have access to the username and password in our home function in routes.js.

So, we are saving the file. Now we need to save the file's path with the user in the database. In the home function in the routes.js, add the following underneath the `password` variable.

```javascript
...

var password = request.body.password;

// Make sure the user uploaded an image
if(request.file) {
  var imgPath  = request.file.path;
}
else {
  response.render('home', { formError: 'You must upload a profile image.' });
  return;
}

...
```

What this says is, if the user uploaded a file, save it as variable `imgPath`. If they did not upload a file, render the 'home' view with an error telling the user they must upload a profile image. The `return;` call kills the function so we do not save the user to the database until they upload an image.

Next, where we create the `newUser` variable, we need to add an `image` key to our object.

```javascript
...

var newUser = new models.User({
  // Pass in the data from our form.
  name: name,
  username: username,
  password: password,
  image: imgPath
});

...
```

We already have an `image` key in our mongoose UserSchema, so there is no need to worry about that.

So, putting it all together routes.js should look like this:

```javascript
var models = require('./models');

exports.home = function (request, response) {
  // Only attempt to process form data if the request method is POST
  if (request.method == 'POST') {
    // Store the form data in some variables
    var name     = request.body.name;
    var username = request.body.username;
    var password = request.body.password;

    // Make sure the user uploaded an image
    if(request.file) {
      var imgPath  = request.file.path;
    }
    else {
      response.render('home', { formError: 'You must upload a profile image.' });
      return;
    }

    // Check that the username and password are acceptable.
    if (username.length >= 3 && password.length >= 6) {
      // Create a new user using our mongoose user schema
      var newUser = new models.User({
        // Pass in the data from our form.
        name: name,
        username: username,
        password: password,
        image: imgPath
      });

      // // Attempt to save the new user.
      newUser.save(function (error, user) {
        if (error) {
          console.log(error);
          // If the save fails, it may be because the username is already taken so return an error.
          response.render('home', { layout: 'main', formError: 'Sorry, that username is already taken.' });
        }
        else {
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
    response.render('home', { layout: 'main' });
  }
};
```

Ready to test it?

### Checkpoint!

Restart node, make sure mongod is still running in a command line window, and then refresh the browser page showing our app. Then, sign up another user (different username), but this time upload a profile photo.

After you click the sign up button, check node. It should say 'The user was successfully saved.' like it did before. Now, however, if you look in the public/uploads directory, you should see the image you uploaded. Yay!

Run the following query in the MongoDB Shell:

```bash
> db.users.find().pretty()
```

It should print out two database entries. One should be the first user you saved without an image. The other should be the new one you just signed up, and the value of the image key should be the path to the saved image.

Let's get rid of the entry without an image. Run:

```bash
> db.users.remove({"image":null})
```

Now, if you rerun the previous find command (remember that you can move through the history of your previous commands with the up arrow. In this case, push it twice, then hit enter), you should now see only the entry that has an image saved.

## Storing a User Session

Now, before we can build out login and logout functionality, we need a way to keep track of whether a user is logged in or not. We will do this by using the browser's ability to store little bits of information, or 'cookies'. If the user is logged in, we will store the user's ID so the next time they come to our site, we know who they are and they do not have to log in again. This is not a very secure approach, but it works for the purpose of this tutorial.

To handle this, we will use the express middleware called <a href="https://github.com/expressjs/cookie-session">cookie-session</a>.

Let's add it to package.json. Don't forget to run an install afterwards.

```javascript
"multer": "1.1.x",
"cookie-session": "1.2.x"
```

After installing, let's import it into app.js, where we have imported all of our other dependencies.

```javascript
var cookieSession = require('cookie-session');

```

Now, directly above our route, let's call the cookieSession function to set things up:

```javascript
...

app.use(cookieSession({
  name: 'session',
  keys: ['userID']
}));

app.all('/', upload.single('image'), routes.home);

...
```

You can see we are calling cookieSession, naming it 'session', and telling it we will be saving the userID. Now, if a user has a cookie from our application saved in their browser, cookie-session will store that cookie in the request object that passes through our routes at request.session.userID. If this userID exists, we want to find the user's database entry so we can use their information (such as who they are following) to build their feed (remember, we are building a twitter clone).

Let's write the code to do this underneath where we called cookieSession.

```javascript
...

app.use(cookieSession({
  name: 'session',
  keys: ['userID']
}));

// Middleware function to add the user to the request object.
app.use(function(request, response, next) {
  console.log(request.session);
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

...
```

These `app.use` functions are run every request, before the routing function is run. So, when a user lands on our home route ('/'), first cookie-session will look in the browser's cookies and see if there is one stored from the last time the user used our application, and stores what it finds in the request object.

Then, the middleware function we just wrote will run. If there is a userID stored in the session cookie, then we retrieve that user's database entry, store it as `request.user`, then proceed to the next step. If there is no userID, we just proceed straight to the next step. In this case the next step is the `routes.home` function.

Since we added a models call in the code above, we need to add this to the top of app.js under our other imports.

```javascript
var models = require('./models');
```

Ok, let's do the final setup to get to a stage where we can test what we have done so far. We need to add the ability to store the cookie, so the functionality we just wrote for finding a cookie will actually find something. A good place to do this is when the user signs up. So in the home route in routes.js, find where we are console logging 'The user was successfully saved.' Directly above this, add the following:

```javascript
// Store the user id in a session cookie so we know the user is logged in now.
request.session.userID = user._id;
```

This is storing the userID in our code, but it doesn't store it to the browser until we send a response. Currently, we are not responding when we save a user. You may have noticed that when you sign up a user, nothing ever happens in the browser. That is because the browser is just sitting there waiting for a response. For now, let's just respond by telling it to render the `home` view again. The `else` statement where we are storing the userID should now look like this:

```javascript
else {
  // Store the user id in a session cookie so we know the user is logged in now.
  request.session.userID = user._id;
  console.log('The user was successfully saved.');
  response.render('home');
}
```
See the render call at the bottom?

### Checkpoint!

Let's see if this worked. Restart node and refresh the browser. After doing these steps, in order, when you go back to the command line running node, you should see an empty object printed out, like this:

```bash
App running on port 3000
{}
```

That is coming from the middleware function we just wrote in app.js. It is consoling the request.session object. Find that line of code so you know what is going on.

Now, Back in our browser, sign up another fake user. Don't forget to upload an image.

When you clicked the sign up button, something subtly different should have happened this time. The page should have refreshed. This is because we are now responding to the POST request by telling it to re-render `home`.

If you go to where node is running your app, below the last consoled empty object, there should be a new object with the key userID and a value of random numbers and letters.

```bash
App running on port 3000
{}
{ userID: '56abbf36f7bf7de21c488daf' }
```

This is the data from your cookie! Every time you refresh the browser, another identical object should console in the command line. This is because every time the page is refreshed, the browser is sending a new GET request to our server. Our cookieSession function is catching that request and attaching the session object to `request` with the data from our cookie inside. Then, our custom middleware function is consoling out the value of `request.session`, which is what we are seeing. Behind the scenes the same middleware function is also querying the database and retrieving the entry that matches the userID.

## Logout Functionality

Ok, we can sign up new users, and we know if a user is alreday signed in based on their cookie. Now let's build the functionality to log them out. This will basically entail deleting the cookie from their browser.

To save some time, let's also add the setup for the ability to login as well. 

First, let's tell our app that we want two new routes. In app.js, under our home route, add the login and logout routes.

```javascript
app.all('/', upload.single('image'), routes.home);
app.all('/login', bodyParser.urlencoded({extended:true}), routes.login);
app.get('/logout', routes.logout);
```

Notice that bodyParser is back in the login route. This is because the login form will not have any files submitted, so it will be urlencoded. We do not need a parser on logout because no data will be passed from the browser at that route.

We are telling the app to use functions called `routes.login` and `routes.logout`, so we need to add those to routes.js, under the home route. For now we will just add placeholders, and will build out the functionality in a minute.

```javascript
...

exports.login = function (request, response) {
  response.render('login');
};

exports.logout = function (request, response) {
  response.render('logout');
};
```

We don't have the templates that these routes are going to try to render so let's make `login.handlebars` and `logout.handlebars` in our views directory. They can be empty files for now.

Great. Now, since the user is logged in (we know this because our cookie is being consoled to the command line.) we need to give them a way to logout. The code to log a user out is pretty simple.

```javascript
exports.logout = function (request, response) {
  // Delete the session and log the user out then return them to the home page.
  delete request.session.userID;
  console.log('Successfully logged out');
  response.redirect('/');
}
```

All we are doing is deleting the cookie and redirecting them home.

### Checkpoint!

As always, restart node. When you refresh the browser, your cookie should still be consoling out to the command line.

Now if you navigate to [localhost:3000/logout](http://localhost:3000/logout) you will be logged out. You can check the terminal where your app is running to make sure it worked. You should see:

```bash
Successfully logged out
{}
```

The empty object shows that our cookie is now deleted. Success!

Now that we know our code is working, go ahead and delete from app.js the line:

```javascript
console.log(request.session);
```

We are doing this to keep our console clean. We don't need to see the session object consoled out during every request.

## Login Functionality

Now we want to make it so the user can log back in with their existing credentials. Let's start with the login view by adding the following to login.handlebars within the views directory.

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

The login route and view should seem somewhat similar to the home route. You should know where the following goes by now:

```javascript
exports.login = function (request, response) {
  // If the user submitted the login form, attempt to log them in.
  if (request.method == 'POST') {
    // Grab the username and password from the post data. 
    var username = request.body.username;
    var password = request.body.password;

    // Query the database for the user we are looking for. If they exist try to log them in, otherwise return an error.
    var user = models.User.findOne({'username': username }, function(error, user) {
      if (error) response.render('login', {formError: 'An error occurred. Please try again.' });
    
      // Check the password that was supplied.
      if ( user && user.authenticate(password) ) {
        request.session.userID = user._id;
        console.log('Successfully logged in as '+ user.name);
        response.redirect('/');
      }
      else {
        // The user supplied an incorrect username/password
        response.render('login', { formError: 'Your login details were incorrect. Please try again.' });
      }
    });
  }
  else {
    // No form was submitted, just show the login page.
    response.render('login');
  }  
}
```

The commenting in the above code should give a breakdown of what the code is doing.

### Checkpoint!

(restart node)

Now if we click the "Log In" link, in the upper right corner of our home page, we see the login page and form. Type in your test username and password, hit the "Log In" button and you'll be logged in. Again, check terminal to make sure everything worked alright. It should tell you that you successfully logged in.

There was a lot covered in this chapter! Now we can log in and out which is awesome. The good thing is that this is really the hardest part of setting up our site. From here on out we will be building some cool functionality and really starting to see things come together.

In the next chapter we will start building all of the routes and views for logged in users. This includes building our feed and giving the user the ability to post tweets!

[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md) -->

