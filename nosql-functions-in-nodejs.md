# How to Create and Use NoSQL Functions in Node.js with MongoDB

MongoDB's implementation of JavaScript, known as MongoDB Query Language (MQL), allows you to create and execute functions on your data. This can be a powerful tool for data manipulation and querying in your Node.js application. In this article, we'll go through the steps to create and use functions in MongoDB using Node.js and MQL, and show you how to use the function in an API built with Express.js.

## Creating a Function

To create a function in MongoDB, we'll use the `db.createFunction()` method provided by the Node.js MongoDB driver. The method takes two arguments: the name of the function and a JavaScript function that implements the function's logic.

Let's create a simple function that adds two numbers:

```javascript
const MongoClient = require("mongodb").MongoClient;

const uri =
	"mongodb+srv://<username>:<password>@<cluster>.mongodb.net/test?retryWrites=true&w=majority";
const client = new MongoClient(uri, {
	useNewUrlParser: true,
	useUnifiedTopology: true,
});

async function createFunction() {
	try {
		await client.connect();

		await client.db().createFunction("sum", function (x, y) {
			return x + y;
		});

		console.log("Function created successfully");
	} catch (err) {
		console.error(err);
	} finally {
		await client.close();
	}
}

createFunction();
```

In this example, we're creating a function named "sum" that takes two parameters and returns their sum. We're using the createFunction() method to create the function in our MongoDB database.
<br>

## Using the Function in an API with Express.js

To use our "sum" function in an API built with Express.js, we'll need to create a route that calls the function and returns the result. Let's create a simple Express.js application that has a /sum route that takes two parameters and returns their sum:

```js
const express = require("express");
const app = express();
const port = 3000;

const MongoClient = require("mongodb").MongoClient;

const uri =
	"mongodb+srv://<username>:<password>@<cluster>.mongodb.net/test?retryWrites=true&w=majority";
const client = new MongoClient(uri, {
	useNewUrlParser: true,
	useUnifiedTopology: true,
});

app.get("/sum", async (req, res) => {
	try {
		const x = parseInt(req.query.x);
		const y = parseInt(req.query.y);

		await client.connect();

		const result = await client.db().eval("sum", [x, y]);

		res.send(`The sum of ${x} and ${y} is ${result}`);
	} catch (err) {
		console.error(err);
		res.status(500).send("Error");
	} finally {
		await client.close();
	}
});

app.listen(port, () => {
	console.log(`Example app listening at http://localhost:${port}`);
});
```

In this example, we're using the app.get() method to create a route that takes two query parameters, x and y, and returns their sum using the "sum" function we created earlier. We're using the eval() method to execute the function on the MongoDB server and return the result. Finally, we're sending the result back to the client using the res.send() method.

That's it! With just a few lines of code, we've created a function in MongoDB, and used it in an API built with Express.js.
