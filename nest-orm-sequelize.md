# Using Sequelize MySQL with NestJS

Sequelize is a popular Object-Relational Mapping (ORM) library for Node.js that provides a simple way to interact with relational databases such as MySQL. In this article, we will explore how to use Sequelize with MySQL in a NestJS application by providing an example code fragment of only the model code.
<br>
## Prerequisites

Before we get started, make sure that you have the following installed on your system:
<br>
* Node.js
* NestJS CLI
* MySQL server
* Sequelize ORM

### **Setting up a NestJS application with Sequelize and MySQL**

To begin, create a new NestJS application by running the following command:

``` sh
$ nest new my-app
```

Next, navigate into the newly created project directory and install the necessary dependencies:

<br>
<br>
```
$ cd my-app
$ npm install --save @nestjs/sequelize sequelize mysql2
```
<br>
<br>
Now, let's create a new Sequelize configuration file in a new folder called "config" at the root of the project directory. Create a file named "database.config.ts" and add the following code:

<br>
<br>

``` ts
import { Sequelize } from 'sequelize-typescript';

export const databaseConfig: Sequelize = new Sequelize({
dialect: 'mysql',
host: 'localhost',
port: 3306,
username: 'root',
password: 'password',
database: 'my_database',
define: {
timestamps: true,
},
});
```

This configuration file sets up a new instance of Sequelize and connects it to a MySQL database running on the local machine.
<br>
### Creating a Sequelize model in NestJS

Now that we have our Sequelize instance set up, let's create a new model in NestJS. Create a new file in the "src" directory called "user.model.ts" and add the following code:

<br>
<br>

``` ts
import { Table, Column, Model } from 'sequelize-typescript';

@Table({ tableName: 'users' })
export class User extends Model {

@Column({ primaryKey: true, autoIncrement: true })
id: number;

@Column({ allowNull: false })
name: string;

@Column({ allowNull: false })
email: string;

@Column({ allowNull: false })
password: string;
}
```


<br>
This model defines a new "User" table in the MySQL database with columns for "id", "name", "email", and "password". The "@Table" decorator sets the name of the table in the database, and the "@Column" decorator sets the properties of each column.
<br>

### Using the Sequelize model in NestJS

Finally, let's use our new Sequelize model in a NestJS controller. Create a new file in the "src" directory called "user.controller.ts" and add the following code:

<br>


``` typescript
import { Controller, Get } from '@nestjs/common';
import { InjectModel } from '@nestjs/sequelize';
import { User } from './user.model';

@Controller('users')
export class UserController {
constructor(
@InjectModel(User)
private readonly userModel: typeof User,
) {}

@Get()
async findAll(): Promise<User[]> {
return this.userModel.findAll();
}
}
```


<br>
<br>
This controller uses the "@InjectModel" decorator to inject the "User" model into the class. The "findAll" method returns all users from the database using the "findAll" method provided by Sequelize.
<br>



## Conclusion

In this article, we explored how to use Sequelize with MySQL in a NestJS application by providing an example code fragment of only the model code. We created a new Sequelize instance, defined a new Sequelize model, and used that model in a NestJS controller. By following this guide, you should now have a basic understanding of how to use Sequelize with MySQL in a NestJS application.