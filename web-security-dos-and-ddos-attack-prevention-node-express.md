# 5 Techniques to Prevent DoS and DDoS Attacks in Express.js and Node.js Applications

### Introduction

Denial of Service (DoS) and Distributed Denial of Service (DDoS) attacks are common threats that web applications face today. These attacks can cause service disruptions, site downtime, and in some cases, can even lead to a data breach. In this article, we'll discuss some techniques to prevent DoS and DDoS attacks in an Express.js and Node.js application.

### Understanding DoS and DDoS Attacks

A DoS attack is an attempt by an attacker to overwhelm a web application with a large number of requests, thereby making the application unavailable to legitimate users. A DDoS attack is similar to a DoS attack, but it is carried out using multiple devices, often controlled by a botnet.

There are several types of DoS and DDoS attacks, including:

SYN Floods: This type of attack floods the server with TCP SYN requests, exhausting the server's resources and making it unavailable.

HTTP Floods: This type of attack floods the server with HTTP requests, consuming the server's resources and causing it to crash.

UDP Floods: This type of attack floods the server with UDP packets, causing the server to consume its resources and become unavailable.

Preventing DoS and DDoS Attacks in Express.js and Node.js
Here are some techniques to prevent DoS and DDoS attacks in an Express.js and Node.js application:

#### 1. Implement Rate Limiting

Rate limiting is a technique used to limit the number of requests that can be made to a web application within a certain time period. This technique can be used to prevent DoS and DDoS attacks by limiting the number of requests that can be made from a single IP address within a certain time period.

Express.js provides several rate limiting middleware packages, such as express-rate-limit and rate-limiter-flexible, that you can use to implement rate limiting in your application. Here's an example of how you can use express-rate-limit:

```js
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
	windowMs: 60 * 1000, // 1 minute
	max: 100, // limit each IP to 100 requests per windowMs
	message: "Too many requests from this IP, please try again later",
});

app.use(limiter);
```

This will limit each IP address to 100 requests per minute. You can adjust the values to suit your application's needs.

#### 2. Implement CAPTCHA Verification

CAPTCHA verification is a technique used to ensure that the request is made by a human user and not an automated script. This technique can be used to prevent DDoS attacks by ensuring that the requests are made by legitimate users and not bots.

Express.js provides several CAPTCHA middleware packages, such as express-recaptcha and hcaptcha-express, that you can use to implement CAPTCHA verification in your application. Here's an example of how you can use express-recaptcha:

```js
const Recaptcha = require("express-recaptcha").RecaptchaV2;

const recaptcha = new Recaptcha(RECAPTCHA_SITE_KEY, RECAPTCHA_SECRET_KEY);

app.get("/", recaptcha.middleware.render);
app.post("/", recaptcha.middleware.verify, (req, res) => {
	// handle the request
});
```

This will render a CAPTCHA widget on the page and verify the user's response when they submit the form. If the CAPTCHA verification fails, the request will be rejected.

#### 3. Implement Request Body Size Limit

Request body size limit is a technique used to limit the size of the request body that can be sent to the server. This technique can be used to prevent DoS attacks by limiting the amount of data that can be sent to the server within a single request.

You can use the body-parser middleware to limit the size of the request body in your Express.js application. Here's an example of how you can limit the request body size to 1MB:

```js
const bodyParser = require("body-parser");

app.use(bodyParser.json({ limit: "1mb" }));
```

This will limit the size of the JSON request body to 1MB. You can adjust the value to suit your application's needs.

#### 4. Implement Connection Timeout

Connection timeout is a technique used to limit the amount of time that a connection can be open before it is terminated. This technique can be used to prevent DoS attacks by closing idle connections and freeing up server resources.

You can use the timeout middleware to implement connection timeout in your Express.js application. Here's an example of how you can set the connection timeout to 30 seconds:

```js
const timeout = require("connect-timeout");

app.use(timeout("30s"));
```

This will terminate any connection that is idle for more than 30 seconds.

#### 5. Implement IP Blocking

IP blocking is a technique used to block requests from specific IP addresses or IP address ranges. This technique can be used to prevent DoS and DDoS attacks by blocking requests from known malicious IP addresses.

You can use the middleware we discussed earlier to block requests from specific IP addresses or IP address ranges. Here's an example of how you can use the blocklist array we created earlier to block requests from the IP addresses in the list:

```js
app.use(function (req, res, next) {
	var ip = req.headers["x-forwarded-for"] || req.connection.remoteAddress;
	if (blocklist.includes(ip)) {
		res.status(403).send("Forbidden");
		return;
	}
	next();
});
```

This will block any request that comes from an IP address in the blocklist array.

### Conclusion

DoS and DDoS attacks are serious threats that can cause service disruptions and downtime for web applications. By implementing the techniques we discussed in this article, you can protect your Express.js and Node.js application from these attacks and ensure that it remains available to legitimate users. Remember to keep your application and middleware packages up to date to ensure that you are protected from the latest threats.
