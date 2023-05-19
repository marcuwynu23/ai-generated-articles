# Using Mongodb Mongoose in NestJS

MongoDB is a popular NoSQL database that is known for its flexibility and scalability. NestJS, on the other hand, is a powerful framework for building server-side applications using TypeScript. In this article, we will look at how to use MongoDB with NestJS.

Prerequisites
Before we start, ensure that you have the following prerequisites installed on your computer:

Node.js

- npm
- NestJS CLI
- MongoDB

## **Creating a NestJS Application**

To get started, we need to create a new NestJS application. Open your terminal and run the following command:

```sh
nest new my-app
```

This will create a new NestJS application with the name "my-app". Change directory into the new application by running:

```sh
cd my-app
```

Next, we need to install the MongoDB driver for Node.js. Run the following command:

```sh
npm install --save @nestjs/mongoose mongoose
```

This will install the Mongoose module, which is an Object Data Modeling (ODM) library for MongoDB and allows us to define models for our data.

##

## Connecting to MongoDB

To connect to MongoDB, we need to create a module that will handle the connection. Create a new file named database.module.ts in the src folder and add the following code:

```js
import { Module } from "@nestjs/common";
import { MongooseModule } from "@nestjs/mongoose";

@Module({
	imports: [MongooseModule.forRoot("mongodb://localhost/my-app")],
})
export class DatabaseModule {}
```

In the code above, we import the MongooseModule from @nestjs/mongoose and use it to connect to MongoDB using the connection string 'mongodb://localhost/my-app'. Replace 'my-app' with the name of your database.

To use the DatabaseModule, we need to import it in our app.module.ts file. Open app.module.ts and add the following code:

```js
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { DatabaseModule } from "./database.module";

@Module({
	imports: [DatabaseModule],
	controllers: [AppController],
	providers: [AppService],
})
export class AppModule {}
```

## Creating a Model

Now that we have connected to MongoDB, let's create a model for our data. In this example, we will create a Task model. Create a new file named task.model.ts in the src folder and add the following code:

```js
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { Document } from "mongoose";

export type TaskDocument = Task & Document;

@Schema()
export class Task {
	@Prop()
	title: string;

	@Prop()
	description: string;

	@Prop()
	completed: boolean;
}

export const TaskSchema = SchemaFactory.createForClass(Task);
```

In the code above, we define a Task model with three properties: title, description, and completed. We also define a TaskSchema using SchemaFactory.createForClass, which converts the Task class into a Mongoose schema.

## Using the Model

To use the Task model in our application, we need to create a service that will handle the CRUD operations. Create a new file named task.service.ts in the src folder and add the following code:

```js
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { Task, TaskDocument } from './task.model

@Injectable()
export class TaskService {
constructor(
@InjectModel(Task.name) private taskModel: Model
,) {}

async create(task: Task): Promise {const createdTask = new this.taskModel(task);return createdTask.save();}

async findAll(): Promise {return this.taskModel.find().exec();}

async findOne(id: string): Promise {return this.taskModel.findById(id).exec();}

async update(id: string, task: Task): Promise {return this.taskModel.findByIdAndUpdate(id, task, { new: true }).exec();}

async remove(id: string): Promise {return this.taskModel.findByIdAndRemove(id).exec();}}

```

In the code above, we use the `@InjectModel` decorator to inject the `Task` model into the service. We then define methods for creating, finding all, finding one, updating, and deleting tasks. Each method calls a corresponding method on the `taskModel`, which is a Mongoose model.
