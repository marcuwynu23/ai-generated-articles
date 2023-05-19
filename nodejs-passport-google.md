# Implementing Google OAuth Using Passport in Node.js

Passport is a popular authentication middleware for Node.js that supports many authentication methods, including Google OAuth. In this article, we will walk through the steps to implement Google OAuth using Passport in a Node.js application.

## Step 1: Set Up a New Node.js Project
The first step is to set up a new Node.js project. Open your terminal and run the following commands:

```
mkdir passport-google
cd passport-google
npm init -y
```


This will create a new directory called `passport-google` and initialize a new Node.js project in it.

## Step 2: Install Dependencies
Next, we need to install the necessary dependencies for Passport and Google OAuth. Run the following command:



```
npm install passport passport-google-oauth20
```


This will install Passport and the Passport Google OAuth 2.0 authentication strategy.

## Step 3: Configure Google OAuth Credentials
Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project. Then, go to the "APIs & Services" > "Credentials" page and click on "Create Credentials" > "OAuth client ID". Choose "Web application" as the application type and enter the following details:

- Name: Any name you want to give to the OAuth client ID.
- Authorized JavaScript origins: `http://localhost:3000`
- Authorized redirect URIs: `http://localhost:3000/auth/google/callback`

Click on "Create" and note down the client ID and client secret.

## Step 4: Configure Passport
Create a new file called `index.js` in your project directory and add the following code:

```javascript
const express = require('express');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

const app = express();

passport.use(new GoogleStrategy({
    clientID: 'YOUR_CLIENT_ID',
    clientSecret: 'YOUR_CLIENT_SECRET',
    callbackURL: 'http://localhost:3000/auth/google/callback'
  },
  (accessToken, refreshToken, profile, done) => {
    // Callback function
    console.log(profile);
  }
));

app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));

app.get('/auth/google/callback', passport.authenticate('google'));

app.listen(3000, () => {
  console.log('Server started on port 3000');
});
```
Replace YOUR_CLIENT_ID and YOUR_CLIENT_SECRET with the client ID and client secret you obtained in Step 3.

This code configures Passport to use the Google OAuth 2.0 authentication strategy. When a user visits the /auth/google route, Passport will redirect them to the Google OAuth consent screen. If the user grants permission to the application, they will be redirected back to the /auth/google/callback route with an access token. Passport will then use this access token to fetch the user's profile information and call the callback function specified in the GoogleStrategy constructor.

## Step 5: Test the Application
Start the server by running node index.js in your terminal. Open your web browser and go to http://localhost:3000/auth/google. You should be redirected to the Google OAuth consent screen. Grant permission to the application and you should be redirected back to http://localhost:3000/auth/google/callback. Check your terminal output for the user's profile information.

In conclusion, using Passport to implement Google OAuth in Node.js is a simple and effective way to authenticate users in your web application. By following these steps, you can easily implement Google OAuth using Passport in your Node.js project.
