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

## **3. Creating Pages & Components**

### **Generate a Component**
```sh
ng generate component pages/home
```

Or shorthand:
```sh
ng g c pages/home
```

### **Create More Pages**
```sh
ng g c pages/about
ng g c pages/contact
ng g c pages/dashboard
```

---

## **4. Setting Up Routing**
Modify `app-routing.module.ts` to define routes:

```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './pages/home/home.component';
import { AboutComponent } from './pages/about/about.component';
import { ContactComponent } from './pages/contact/contact.component';
import { DashboardComponent } from './pages/dashboard/dashboard.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent },
  { path: 'dashboard', component: DashboardComponent },
  { path: '**', redirectTo: '', pathMatch: 'full' },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

---

## **5. Fetching Data Using Services**

### **Generate a Service**
```sh
ng g service services/api
```

Modify `api.service.ts` to fetch data:

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ApiService {
  constructor(private http: HttpClient) {}

  getUsers(): Observable<any> {
    return this.http.get('https://jsonplaceholder.typicode.com/users');
  }
}
```

Use the service in a component:

```ts
import { Component, OnInit } from '@angular/core';
import { ApiService } from '../../services/api.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.scss'],
})
export class HomeComponent implements OnInit {
  users: any[] = [];

  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService.getUsers().subscribe((data) => {
      this.users = data;
    });
  }
}
```

---

## **6. Implementing Authentication (JWT)**

### **Generate Auth Service**
```sh
ng g service services/auth
```

Modify `auth.service.ts`:

```ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class AuthService {
  private tokenKey = 'authToken';

  isLoggedIn(): boolean {
    return !!localStorage.getItem(this.tokenKey);
  }

  login(username: string, password: string): boolean {
    if (username === 'admin' && password === 'password') {
      localStorage.setItem(this.tokenKey, 'dummy-jwt-token');
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

## **7. Setting Up Protected Routes**

### **Generate Auth Guard**
```sh
ng g guard guards/auth --implements CanActivate
```

Modify `auth.guard.ts`:

```ts
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root',
})
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(): boolean {
    if (this.authService.isLoggedIn()) {
      return true;
    } else {
      this.router.navigate(['/login']);
      return false;
    }
  }
}
```

Modify `app-routing.module.ts`:

```ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'login', component: LoginComponent },
  { path: 'dashboard', component: DashboardComponent, canActivate: [AuthGuard] },
  { path: '**', redirectTo: '' },
];
```

---

## **8. Building and Deploying Angular App**

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
This guide outlines a **structured workflow** for developing an Angular application, from setup to deployment. You now have a solid foundation to build and expand your Angular projects!

**Would you like a guide on state management using NgRx or SSR with Angular Universal?** 🚀

