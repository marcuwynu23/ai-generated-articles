# Full Implementation Workflow of Angular CLI with CRUD Todo List using API Calls and JWT Authentication
## Introduction
Angular is a popular JavaScript framework used for building web applications. In this article, we will explore the full implementation workflow of Angular CLI, including creating a CRUD Todo List application that interacts with an API. We will also add JWT (JSON Web Token) authentication to secure the application and implement features like authentication guards, interceptors, middleware, and more. Let's dive in!

## Prerequisites

Before we begin, ensure that you have the following installed:

- Node.js (with npm)
- Angular CLI

## Step 1: Project Setup

Start by creating a new Angular project using the Angular CLI. Open a terminal and run the following command:

```sh
ng new todo-app
```
This command will create a new Angular project called "todo-app" in a directory with the same name. Change to the project directory:

```sh
cd todo-app
```

## Step 2: Component and Service Creation
In Angular, components represent different parts of your application's user interface, and services handle data fetching and manipulation. Create the necessary components and services for our Todo List application:

```sh
ng generate component todo-list
ng generate component todo-form
ng generate service todo
ng generate service auth
```

This will generate the required files for the Todo List components (todo-list and todo-form) and the services (todo and auth).

## Step 3: HTTP Requests and API Integration

In the todo.service.ts file, implement methods to interact with the API for CRUD operations. Use Angular's HttpClient module to make HTTP requests to the API endpoints. For example:

```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';

@Injectable({
providedIn: 'root'
})
export class TodoService {
private apiUrl = 'https://api.example.com/todos';

constructor(private http: HttpClient) { }

getTodos() {
return this.http.get(`${this.apiUrl}`);
}

createTodo(todo: any) {
return this.http.post(`${this.apiUrl}`, todo);
}

updateTodo(todoId: number, todo: any) {
return this.http.put(`${this.apiUrl}/${todoId}`, todo);
}

deleteTodo(todoId: number) {
return this.http.delete(`${this.apiUrl}/${todoId}`);
}
}
```

## Step 4: JWT Authentication Setup

Implement JWT authentication in the auth.service.ts file. This service will handle user authentication, token storage, and token retrieval. Use Angular's HttpClient module to send requests to the authentication endpoints. For example:

```typescript

import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';

@Injectable({
providedIn: 'root'
})
export class AuthService {
private apiUrl = 'https://api.example.com/auth';

constructor(private http: HttpClient) { }

login(credentials: any) {
return this.http.post(`${this.apiUrl}/login`, credentials);
}

logout() {
// Clear token and any stored user information
}

isAuthenticated() {
// Check if the user is authenticated (token is valid)
}
}
```
## Step 5: Authentication Guard

Implement an authentication guard to protect routes that require authentication. Create a new file, auth.guard.ts, and add the necessary logic. The guard can be used to restrict access to certain routes based on the user's authentication status. For example:

```typescript
import { Injectable } from "@angular/core";
import {
	CanActivate,
	ActivatedRouteSnapshot,
	RouterStateSnapshot,
	UrlTree,
	Router,
} from "@angular/router";
import { Observable } from "rxjs";
import { AuthService } from "./auth.service";

@Injectable({
	providedIn: "root",
})
export class AuthGuard implements CanActivate {
	constructor(private authService: AuthService, private router: Router) {}

	canActivate(
		next: ActivatedRouteSnapshot,
		state: RouterStateSnapshot
	):
		| Observable<boolean | UrlTree>
		| Promise<boolean | UrlTree>
		| boolean
		| UrlTree {
		if (this.authService.isAuthenticated()) {
			return true;
		} else {
			this.router.navigate(["/login"]);
			return false;
		}
	}
}
```

## Step 6: Interceptor for JWT Token

Create an interceptor to include the JWT token in every outgoing request. This interceptor will add the token to the Authorization header automatically. Create a new file, jwt.interceptor.ts, and add the necessary logic. For example:

```typescript
import { Injectable } from "@angular/core";
import {
	HttpInterceptor,
	HttpRequest,
	HttpHandler,
	HttpEvent,
} from "@angular/common/http";
import { Observable } from "rxjs";
import { AuthService } from "./auth.service";

@Injectable()
export class JwtInterceptor implements HttpInterceptor {
	constructor(private authService: AuthService) {}

	intercept(
		request: HttpRequest<any>,
		next: HttpHandler
	): Observable<HttpEvent<any>> {
		const token = this.authService.getToken();

		if (token) {
			request = request.clone({
				setHeaders: {
					Authorization: `Bearer ${token}`,
				},
			});
		}

		return next.handle(request);
	}
}
```

## Step 7: Routing and Middleware

Define routes in the app-routing.module.ts file and apply the authentication guard to the appropriate routes. For example:

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";
import { AuthGuard } from "./auth.guard";
import { TodoListComponent } from "./todo-list/todo-list.component";
import { TodoFormComponent } from "./todo-form/todo-form.component";

const routes: Routes = [
	{ path: "login", component: LoginComponent },
	{ path: "todos", component: TodoListComponent, canActivate: [AuthGuard] },
	{ path: "todos/new", component: TodoFormComponent, canActivate: [AuthGuard] },
	{
		path: "todos/:id/edit",
		component: TodoFormComponent,
		canActivate: [AuthGuard],
	},
	{ path: "", redirectTo: "login", pathMatch: "full" },
];

@NgModule({
	imports: [RouterModule.forRoot(routes)],
	exports: [RouterModule],
})
export class AppRoutingModule {}
```

## Conclusion

Congratulations! You have successfully implemented a full workflow for an Angular CLI project with a CRUD Todo List that interacts with an API. You have also added JWT authentication, implemented an authentication guard, interceptor, and middleware. With this implementation, you can securely manage todos with authentication and perform CRUD operations using Angular's powerful features. Feel free to explore and enhance the application further based on your requirements.

Remember to refer to the official Angular documentation and related resources for more in-depth explanations and customization options. Happy coding!

Note: This article provides a simplified implementation of Angular CLI with JWT authentication. In real-world scenarios, you may need to handle additional features like form validation, error handling, user registration, and password recovery.
