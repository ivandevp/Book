# Chapter Four: Interacting with the Database

We've now successfully created a user and thus have committed a successful write to the database. However, this isn't going to do much if the object is later able to be retrieved and displayed. This chapter will address pulling objects out of the database and making them useable in your templates.

Let's begin by defining what a post is in the schema. What information do we need to store each post?
- The poster. The user who wrote it.
- The body. What the message says
- The time. The moment of when it was posted so we can sort by the latest posts.

First thing we'll need to import the ObjectId because we'll identify the poster by his user's ObjectId. Your imports on models.js should look like this:
```javascript
var mongoose = require('mongoose');
var mongoUri = process.env.MONGOLAB_URI || 'localhost';
var db       = mongoose.connect('localhost', 'nulltonode');
var Schema   = mongoose.Schema;
var ObjectId = Schema.ObjectId;
```

Next we need to in a new model: a post model. This model will represent each individual post, or in Twitter's terms, each tweet. We'll also need to expose this model to the rest of application by exporting it.

```javascript
var PostSchema = new Schema({
  user: { type: ObjectId, ref: 'User' },
  body: { type: String, required: true },
  date: { type: Date, required: true }
})

UserSchema.plugin(require('basic-auth-mongoose'));
exports.User = db.model('User', UserSchema);
exports.Post = db.model('Post', PostSchema);
```

Now we have a post model so we're free to use them. So now that we have a model of posts, we want to be able to create them and see them as well. Let's first create a new feed.handlebars in the views (not views/layouts) directory.

```html
<h1>Your Feed</h1>
<hr>
<div class="row">
  <div class="span5">
    <ul class="post-feed">
      {{#each posts}}
      <li class="post">
        <img class="img-circle" src="{{ user.image }}">
        <div class="post-content">
          <p><b>{{ user.name }}</b></p>
          <p>{{ body }}</p>
        </div>
      </li>
      {{/each}}
    </ul>
  </div>
  <div class="span3">
    <div class="user-profile well">
      <p class="lead">{{ user.name }}</p>
      <p><b>@{{ user.username }}</b></p>
      <img class="user-image img-circle" src="{{ user.image }}" alt="">
    </div>
    {{#if formError }}
    <p class="alert alert-error">{{{ formError }}}</p>
    {{/if}}
    <form action="" method="post" accept-charset="utf-8">
      <p>Write a new post:</p>
      <textarea name="postBody"></textarea>
      <input class="btn btn-primary" type="submit" name="" value="Submit">
    </form>
  </div>
</div>
```

What's happening here? Here we're using Handlebars to display each post. the `{{#each posts}}` is going to loop over an array will provide called `posts` and display each one. The `{{/#each}}` lets Handlebars the scope of what will be repeated for each item in the `posts` array. After that, we're going to show the user his/her own information and then provide a `<textarea>` where the user can create and post new tweets. This will be a mostly unstyled mess, but it should display.

Okay, now we'll need to be able to get this new page, so let's put a few lines in app.js to allow us to see it.

```javascript
app.all('/feed', auth, routes.feed);
```

Notice we've put this auth method in. It's a pretty simple method where we're going to check if they're logged in. Pretty simplistic and not a whole solution for authentication.

```javascript
var auth = [routes.checkAuth];
```

This goes at the top of app.js. We're providing Node.js with an array of functions to run before running the function given. In this case, it's just one function that we're going to call from the routes.js file.

```javascript
exports.checkAuth = function (request, response, next) {
  if(!request.session.userID) {
    response.redirect('/');
  }

  next();
}
```
Pretty simple. We're just checking for the session userID. If it's there, we continue by calling the callback. If not, we redirect to the homepage.

Okay, let's go create this feed method now in the routes.js

```javascript
exports.feed = function (request, response) {
  if (request.method == 'POST') {
    var postBody = request.body.postBody;

    // Create a new post
    var newPost = new models.Post({
      user: request.session.userID,
      body: postBody,
      date: new Date()
    });

    // Save the new post.
    newPost.save(function (error, user) {
      if (error) {
        console.log(error);
        response.render('feed', { layout: 'base', formError: 'There was an error creating your post.' });
      }
      else {
        response.redirect('/feed');
      }
    });
  }
  else {
    var userIDs = _.union(request.user.following, [request.user.id])

    models.Post.find().where('user').in(userIDs).populate('user').sort('-date')
    .exec(function (err, posts) {
      if (err) console.log(err);
      response.render('feed', { 'posts': posts, 'user': request.user });
    });
  }
}
```

If the user is posting to the page then they're creating a new "tweet." If they're not, they're trying to get the information then they're trying to display the page and thus should be given their feed.

So this looks awful, but let's worry about the following for now. Right now, you only really follow yourself. We want to be able follow other people. We're going to this by AJAX, meaning we're not going to refresh the page to do the follow. Let's first create a place a server that we can call in app.js.

```javascript
app.post('/follow', auth, routes.follow);
```

Pretty straightforward. Now let's create the route in route.js.

```javascript
exports.follow = function (request, response) {
  if (request.body.id && request.user.id ) {
    models.User.findOne({ '_id': request.body.id }, function (err, userToFollow) {
      if (err) response.end(err);
      userToFollow.followers.push(request.user.id);
      request.user.following.push(userToFollow.id);

      userToFollow.followers = _.uniq(userToFollow.followers);
      request.user.following = _.uniq(request.user.following);

      userToFollow.save();
      request.user.save();

      response.end( userToFollow.id );
    });
  }
}
```

Pretty straightforward. We're updating both the session (the current visit according to the server) and the database (so that next time they visit they'll still be following them.) We're storing this in MongoDB.

Now we need to be able to do this from the user's browser so we're going run some client-JavaScript to accomplish this. In the public directory create one called javascripts. In there, put a follow.js file with this in there.

```javascript
function followUser (userID) {
  console.log('ajax Follow user: ' + userID);

  $.ajax({
    type: "POST",
    url: '/follow',
    data: { id: userID },
    success: userFollowCallback
  });
}

function userFollowCallback (response) {
  var userElement = $('#user-'+ response);
  userElement.find('a').removeClass('btn-primary').addClass('btn-success').html('<i class="icon-white icon-check"></i>&nbsp; Following');
}
```

Now we need to include this in the template.


