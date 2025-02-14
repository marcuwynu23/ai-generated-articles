# **Comprehensive Guide on Angular Workflow**

## **Introduction**

Angular is a powerful framework for building modern web applications. This guide provides a structured workflow for developing an Angular application, covering everything from setup to deployment.

---

## **1. Installing Angular CLI**

The Angular CLI (Command Line Interface) simplifies Angular development.

### **Install Angular CLI**

```sh
npm install -g @angular/cli
```

### **Verify Installation**

```sh
ng version
```

---

## **2. Creating a New Angular Project**

### **Generate a New Angular App**

```sh
ng new my-angular-app
```

- Choose **Yes** for routing.
- Select **SCSS** or **CSS** for styling.

### **Navigate to Project Directory & Start Development Server**

```sh
cd my-angular-app
ng serve
```

Open **http://localhost:4200/** to view your app.

---

## **3. Creating a Modular Feature-Based Architecture**

### **Why Use a Modular Approach?**

- **Scalability**: Makes the application easy to maintain as it grows.
- **Reusability**: Modules can be reused across different parts of the app.
- **Separation of Concerns**: Each feature has its own dedicated module, reducing dependencies.

### **Generate Feature Modules**

```sh
ng g module features/home --routing
ng g module features/about --routing
ng g module features/contact --routing
ng g module features/dashboard --routing
```

This creates a directory structure like:

```
/src/app/features/
  home/
    home.module.ts
    home-routing.module.ts
  about/
    about.module.ts
    about-routing.module.ts
  contact/
    contact.module.ts
    contact-routing.module.ts
  dashboard/
    dashboard.module.ts
    dashboard-routing.module.ts
```

### **Define Routes for Each Module**

Modify `home-routing.module.ts`:

```ts
import { NgModule } from "@angular/core";
import { RouterModule, Routes } from "@angular/router";
import { HomeComponent } from "./home.component";

const routes: Routes = [{ path: "", component: HomeComponent }];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class HomeRoutingModule {}
```

### **Update Feature Module**

Modify `home.module.ts`:

```ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { HomeComponent } from "./home.component";
import { HomeRoutingModule } from "./home-routing.module";

@NgModule({
  declarations: [HomeComponent],
  imports: [CommonModule, HomeRoutingModule],
})
export class HomeModule {}
```

### **Lazy Load Feature Modules in App Routing**

Modify `app-routing.module.ts`:

```ts
const routes: Routes = [
  { path: "", loadChildren: () => import("./features/home/home.module").then((m) => m.HomeModule) },
  { path: "about", loadChildren: () => import("./features/about/about.module").then((m) => m.AboutModule) },
  { path: "contact", loadChildren: () => import("./features/contact/contact.module").then((m) => m.ContactModule) },
  { path: "dashboard", loadChildren: () => import("./features/dashboard/dashboard.module").then((m) => m.DashboardModule) },
  { path: "**", redirectTo: "", pathMatch: "full" },
];
```

---

## **4. Fetching Data Using Services**

### **Generate a Service**

```sh
ng g service services/api
```

Modify `api.service.ts` to fetch data:

```ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class ApiService {
  constructor(private http: HttpClient) {}

  getUsers(): Observable<any> {
    return this.http.get("https://jsonplaceholder.typicode.com/users");
  }
}
```

---

## **5. Implementing Authentication (JWT)**

### **Generate Auth Service**

```sh
ng g service services/auth
```

Modify `auth.service.ts`:

```ts
import { Injectable } from "@angular/core";

@Injectable({
  providedIn: "root",
})
export class AuthService {
  private tokenKey = "authToken";

  isLoggedIn(): boolean {
    return !!localStorage.getItem(this.tokenKey);
  }

  login(username: string, password: string): boolean {
    if (username === "admin" && password === "password") {
      localStorage.setItem(this.tokenKey, "dummy-jwt-token");
      return true;
    }
    return false;
  }

  logout(): void {
    localStorage.removeItem(this.tokenKey);
  }
}
```

---

## **6. Building and Deploying Angular App**

### **Build for Production**

```sh
ng build --configuration production
```

### **Deploy to Firebase**

```sh
ng add @angular/fire
firebase deploy
```

### **Deploy to GitHub Pages**

```sh
ng add angular-cli-ghpages
ng deploy
```

### **Deploy to Vercel**

```sh
vercel
```

---

## **Conclusion**

This guide outlines a **structured workflow** for developing an Angular application, from setup to deployment, including a **modular feature-based approach** for scalability. You now have a solid foundation to build and expand your Angular projects!

**Would you like a guide on state management using NgRx or SSR with Angular Universal?** 🚀
