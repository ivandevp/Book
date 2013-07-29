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


[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md)
