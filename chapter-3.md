# Chapter Three: User Authentication

In this section, we are going to create the logic to sign users up and log them in and out. There are a million ways to do this properly, however, in this book we will be taking a simple and not very secure approach for simplicity.

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

When the form is submitted its going to post data to our home route. So lets go there and set things up to recieve this data.

```javascript
exports.home = function (request, response) {
  if (request.session.userID) {
    return response.redirect('/feed');
  }

  if (request.method == 'POST') {
    var name        = request.body.name;
    var username    = request.body.username;
    var password    = request.body.password;
    var tmp_path    = request.files.image.path;

    // Set the directory where we want to store the images. 
    // Make sure to create this directory or it will throw an error.
    var target_path = './uploads/images/' + request.files.image.name;

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

    // Check that the username and password are acceptable.
    if (username.length >= 3 && password.length >= 6) {
      // Create a new user
      var newUser = new models.User({
        name:     name,
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
  }
  else {
    response.render('home', { layout: 'base' });
  }
}

```



[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md)
