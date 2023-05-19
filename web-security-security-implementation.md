# Web Security Implementation in node Express.js

### Cross-site scripting (XSS)

To prevent XSS attacks, you can use a middleware called helmet, which provides various security-related HTTP headers. Here's an example of how to use helmet in an Express.js application:

```javascript
const helmet = require("helmet");

app.use(helmet());
```

This will automatically set various security-related headers in your HTTP response, including the X-XSS-Protection header, which helps prevent XSS attacks in some browsers.

### SQL injection (SQLi)

To prevent SQL injection attacks, you should always use parameterized queries when interacting with your database. Here's an example of a parameterized query in Node.js using the mysql2 library:

```javascript
const mysql = require("mysql2/promise");

async function getUserByUsername(username) {
	const connection = await mysql.createConnection({
		host: "localhost",
		user: "root",
		password: "password",
		database: "mydatabase",
	});
	const [rows, fields] = await connection.execute(
		"SELECT * FROM users WHERE username = ?",
		[username]
	);
	return rows;
}
```

This example uses a parameterized query with a placeholder (?) for the username input parameter. This helps prevent SQL injection attacks by automatically escaping any special characters in the input.

### Cross-site request forgery (CSRF)

To prevent CSRF attacks, you can use a middleware called csurf, which generates and validates CSRF tokens for each HTTP request. Here's an example of how to use csurf in an Express.js application:

```javascript
const csurf = require("csurf");

app.use(csurf());

app.get("/", (req, res) => {
	const csrfToken = req.csrfToken();
	res.render("index", { csrfToken });
});
```

This example generates a CSRF token for each HTTP request using the req.csrfToken() function, and then passes it to the view template for use in any HTML forms. The csurf middleware will automatically validate the CSRF token for each form submission.

### Distributed denial-of-service (DDoS)

To prevent DDoS attacks, you can use various techniques such as rate limiting, IP blocking, and traffic filtering. Here's an example of how to implement rate limiting using the express-rate-limit middleware:

```javascript
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
	windowMs: 15 * 60 * 1000, // 15 minutes
	max: 100, // limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

This example uses the express-rate-limit middleware to limit the number of requests per IP address to 100 per 15-minute window.

### Man-in-the-middle (MitM)

To prevent MitM attacks, you should always use HTTPS to encrypt all communication between the client and server. Here's an example of how to use HTTPS in an Express.js application:

```javascript
const https = require("https");
const fs = require("fs");

const options = {
	key: fs.readFileSync("server.key"),
	cert: fs.readFileSync("server.crt"),
};

https.createServer(options, app).listen(443);
```

This example uses the https module to create an HTTPS server using a private key and certificate.

### Clickjacking

To prevent clickjacking attacks, you can use a middleware called helmet, which provides the X-Frame-Options header to control how your website can be embedded in other websites. Here's an example of how to use `helmet` in an Express.js application to prevent clickjacking:

```javascript
const helmet = require("helmet");

app.use(helmet.frameguard({ action: "sameorigin" }));
```

This example uses the frameguard middleware from helmet to set the X-Frame-Options header to SAMEORIGIN, which prevents your website from being embedded in other websites.

### Brute-force attacks

To prevent brute-force attacks, you can use a middleware called express-brute, which tracks failed login attempts and blocks further attempts from the same IP address after a certain number of failures. Here's an example of how to use express-brute in an Express.js application:

```javascript
const ExpressBrute = require("express-brute");
const MongoStore = require("express-brute-mongo");
const mongoose = require("mongoose");

const store = new MongoStore(() => mongoose.connection.db, {
	collectionName: "bruteforce",
	expireAfterSeconds: 60 * 60, // 1 hour
});

const bruteforce = new ExpressBrute(store, {
	freeRetries: 5,
	minWait: 5 * 60 * 1000, // 5 minutes
	maxWait: 60 * 60 * 1000, // 1 hour
	lifetime: 24 * 60 * 60, // 1 day
	failCallback: (req, res, next, nextValidRequestDate) => {
		res
			.status(403)
			.send("Too many failed login attempts. Please try again later.");
	},
});

app.post("/login", bruteforce.prevent, (req, res) => {
	// handle login
});
```

This example uses express-brute with a MongoDB store to track failed login attempts and block further attempts from the same IP address after 5 failures. The failCallback function is called when the maximum number of failures is reached, and returns a 403 Forbidden response.

## Conclusion

These are just a few examples of how to prevent common web application attacks in Node.js and Express.js. It's important to keep in mind that security is a continuous process, and you should always be vigilant and keep up-to-date with the latest security practices.
