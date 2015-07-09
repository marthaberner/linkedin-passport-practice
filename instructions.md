## Boilerplate Part 1 - Create and deploy your app

Generate an express app that includes a `.gitignore` file:

```
express --git linkedin-oauth
cd linkedin-oauth
npm install
nodemon
```

Make sure that the app works in a browser.  Then initialize a git repository:

```
git init .
git add -A
git commit -m "Initial commit"
```

Now create a repository on Github, set the remote properly and push.

## Boilerplate Part 2 - Deploy

Create an app on Heroku, deploy to it and verify that your app works on Heroku:

```
heroku apps:create
git push heroku master
heroku open
```

Now that you have a Heroku URL, add a README file, commit and push to Github.

## Boilerplate Part 3 - Install and configure dotenv and cookie-session

Passport requires that your app have a `req.session` object it can write to.  To enable this, install and configure `cookie-session`.  In order to keep your secrets safe, you'll need to also configure and install `dotenv`.

```
npm install dotenv --save
touch .env
echo .env >> .gitignore
npm install cookie-session --save
```

In your `.env` file add two environment variables: `SESSION_KEY1` and `SESSION_KEY2`.

In `app.js`, configure `cookie-session` appropriately:

```js
// in the require section up at the top...
var session = require('cookie-session')

// after app.use(express.static(path.join(__dirname, 'public')));
app.use(session({ keys: [process.env.SESSION_KEY1, process.env.SESSION_KEY2] }))
```

Ensure that you app works locally, then git add, commit, push to Github and deploy to Heroku.

Once you've deployed, be sure to set `SESSION_KEY1` and `SESSION_KEY2` on Heroku:

```
heroku config:set SESSION_KEY1=abc123 SESSION_KEY2=def234
```

Verify that your app still works correctly on Heroku.

## Boilerplate Part 4 - Set the HOST environment variable

For your app to work both locally and on production, it will need to know what URL it's being served from.  To do so, add a HOST environment variable to `.env` locally to localhost:

```
echo HOST=http://localhost:3000
```

Then set the HOST environment variable on Heroku to your Heroku URL:

```
heroku config:set HOST=https://your-heroku-app.herokuapp.com
```

There should be nothing to commit at this point.

## Register your LinkedIn Application

1. Login to https://www.linkedin.com/
1. Visit https://www.linkedin.com/developer/apps and create a new app
1. For Logo URL, add your own OR you can use https://brandfolder.com/galvanize/attachments/suxhof65/galvanize-galvanize-logo-g-only-logo.png?dl=true
1. Under website your Heroku URL
1. Fill in all other required fields and submit

On the "Authentication" screen:

- Under authorized redirect URLs enter http://localhost:3000/auth/linkedin/callback
- Under authorized redirect URLs enter your Heroku url, plus `/auth/linkedin/callback`

You should see a Client ID and Client Secret.  Add these to your `.env` file, and set these environment variables on Heroku.  Your `.env` file should look something like:

```
LINKEDIN_CLIENT_ID=abc123
LINKEDIN_CLIENT_SECRET=abc123
```

Set those variables on Heroku as well:

```
heroku config:set LINKEDIN_CLIENT_ID=abc123 LINKEDIN_CLIENT_SECRET=def234
```

There should be nothing to add to git at this point.

## Install and configure passport w/ the LinkedIn strategy

```
npm install passport --save
npm install passport-linkedin --save
```

First, add the Passport middleware to `app.js`:

```js
var passport = require('passport')
app.use(passport.initialize());
app.use(passport.session());
```

Then tell Passport to use the LinkedIn strategy:

```js
var LinkedInStrategy = require('passport-linkedin').Strategy

passport.use(new LinkedInStrategy({
    consumerKey: process.env.LINKEDIN_CLIENT_ID,
    consumerSecret: process.env.LINKEDIN_CLIENT_SECRET,
    callbackURL: process.env.HOST + "/auth/linkedin/callback"
  },
  function(token, tokenSecret, profile, done) {
    done(null, profile)
  }
));
```

Finally, tell Passport how to store the user's information in the session cookie:

```
passport.serializeUser(function(user, done) {
  done(null, user);
});

passport.deserializeUser(function(user, done) {
  done(null, user)
});
```

Run the app locally to make sure that it's still functioning (and isn't throwing any errors).

## Create the auth and oauth-related routes

Create a new route file for your authentication routes:

```
touch routes/auth.js
```

In `routes/auth.js`, you'll need to add a route for logging in, and one for logging out.  In addition, you'll have to create the route that LinkedIn will call once the user has authenticated properly:

```
var express = require('express');
var router = express.Router();
var passport = require('passport');

router.get('/auth/linkedin', passport.authenticate('linkedin'));

router.get('/logout', function (req, res, next) {
  req.session = null
  res.redirect('/')
});

router.get('/auth/linkedin/callback',
  passport.authenticate('linkedin', { failureRedirect: '/' }),
  function(req, res) {
    res.redirect('/')
  });

module.exports = router;
```

Back in `app.js`, be sure to require and use the new routes file:

```js
// up with the require statements...
var authRoutes = require('./routes/index');

// right after app.use('/', routes);
app.use('/', authRoutes);
```

With this setup, you should be able to login with LinkedIn successfully by visiting the following URL directly:

http://localhost:3000/auth/linkedin

If it's successful, you should be redirected to the homepage.  You won't see anything cool yet...

## Configure the views

You'll want to display the logged-in user's name in the header of the page.  To do so, add the following lines inside the `body` tag in `./views/layout.jade`:

```jade
    if user
      div Hello #{user.displayName}
```

If you refresh your browser now, it will throw an error, because we haven't set the `user` local variable yet.  Let's use some custom middleware to do so:

```js
app.use(function (req, res, next) {
  res.locals.user = req.user
  next()
})
```

http://passportjs.org/docs
https://github.com/jaredhanson/passport-linkedin

There's working code here:

https://github.com/jaredhanson/passport-linkedin/blob/master/examples/login/app.js

https://github.com/expressjs/session

https://www.linkedin.com/

https://developer.linkedin.com/

## Configure Passport

Follow instructions here: https://github.com/jaredhanson/passport-linkedin#configure-strategy

Make the callback URL: http://localhost:3000/auth/linkedin/callback

That README has some code like:

```
User.findOrCreate({ linkedinId: profile.id }, function (err, user) {
  return done(err, user);
});
```

This is using a different Mongo library (Mongoose) as opposed to Monk.  What do you think they are doing there?  How can you rewrite that part using just Monk code?

HINT: look into the `upsert` option in Mongo:

http://docs.mongodb.org/manual/reference/method/db.collection.update/#db.collection.update

Of course, you can just use standard find / insert methods too :)


http://passportjs.org/docs/configure#configure


passport.serializeUser(function(user, done) {
  done(null, user);
});

passport.deserializeUser(function(user, done) {
  done(null, user);
});
