# AngularProjectStructure
Angular Lazy loading modules

### Use Case

Suppose you are building an Angular project for an e-commerce application that has the following sections:

1. **Home** (Homepage with general components)
2. **Products** (A list of products, product details, etc.)
3. **Cart** (Shopping cart functionality)
4. **User Profile** (User account and profile management)
5. **Shared** (Common components like buttons, modals, or pipes)
6. **Core** (Core services, guards, interceptors that are needed throughout the app)

You want to:
- **Lazy load** feature modules like `Home`, `Products`, `Cart`, and `User Profile`.
- Have a **CoreModule** that provides services and guards (singleton) and is imported only once.
- Have a **SharedModule** for reusable components (buttons, pipes, etc.) that can be used across different modules without duplication.

### Recommended Project Structure

```bash
src/
│
├── app/
│   ├── core/               # CoreModule (Singleton services, Guards, Interceptors)
│   ├── shared/             # SharedModule (Shared Components, Pipes, Directives)
│   ├── home/               # HomeModule (Lazy-loaded feature module)
│   ├── products/           # ProductsModule (Lazy-loaded feature module)
│   ├── cart/               # CartModule (Lazy-loaded feature module)
│   ├── user-profile/       # UserProfileModule (Lazy-loaded feature module)
│   └── app.module.ts       # AppModule (Root module)
│   └── app-routing.module.ts  # AppRoutingModule (Handles route configuration)
└── main.ts
```

### Step-by-Step Implementation

1. **AppModule (`app.module.ts`)**  
   The `AppModule` should be light and should import minimal modules like `AppRoutingModule`. You should **not** import the feature modules (like `HomeModule`, `ProductsModule`, etc.) here, as they will be lazy-loaded.

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module'; // App routing
import { AppComponent } from './app.component';
import { CoreModule } from './core/core.module'; // Import CoreModule once in AppModule

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,   // Main routing with lazy loading
    CoreModule,         // Import CoreModule here (singleton services)
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

2. **App Routing with Lazy Loading (`app-routing.module.ts`)**  
   Use `loadChildren` to lazy-load the feature modules when their routes are activated.

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', loadChildren: () => import('./home/home.module').then(m => m.HomeModule) },
  { path: 'products', loadChildren: () => import('./products/products.module').then(m => m.ProductsModule) },
  { path: 'cart', loadChildren: () => import('./cart/cart.module').then(m => m.CartModule) },
  { path: 'profile', loadChildren: () => import('./user-profile/user-profile.module').then(m => m.UserProfileModule) },
  { path: '**', redirectTo: '' }  // Wildcard route to handle any unmatched paths
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

3. **CoreModule (`core.module.ts`)**  
   This module should contain all singleton services, guards, and interceptors that need to be available app-wide.

```typescript
// core.module.ts
import { NgModule, Optional, SkipSelf } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AuthService } from './services/auth.service';   // Example service
import { AuthGuard } from './guards/auth.guard';         // Example guard

@NgModule({
  imports: [CommonModule],
  providers: [AuthService, AuthGuard]  // Services that should be singleton
})
export class CoreModule {
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error('CoreModule is already loaded. Import it only in the AppModule.');
    }
  }
}
```

4. **SharedModule (`shared.module.ts`)**  
   SharedModule should contain components, directives, and pipes that are reusable across different modules but don’t provide singleton services. It is designed for components that need to be used in multiple places.

```typescript
// shared.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { SharedButtonComponent } from './components/shared-button/shared-button.component';
import { MyCustomPipe } from './pipes/my-custom.pipe';

@NgModule({
  declarations: [
    SharedButtonComponent,
    MyCustomPipe
  ],
  imports: [CommonModule],
  exports: [
    SharedButtonComponent,  // Export components to be used in other modules
    MyCustomPipe
  ]
})
export class SharedModule { }
```

5. **Feature Modules (e.g., ProductsModule)**  
   Every feature module (like `ProductsModule`, `HomeModule`, etc.) should import `SharedModule` if it needs shared components and should be lazy-loaded via routing.

```typescript
// products.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductsRoutingModule } from './products-routing.module'; // Routing for this module
import { ProductsComponent } from './components/products/products.component';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';
import { SharedModule } from '../shared/shared.module';  // Import shared components

@NgModule({
  declarations: [
    ProductsComponent,
    ProductDetailComponent
  ],
  imports: [
    CommonModule,
    ProductsRoutingModule,
    SharedModule  // Use shared components/pipes if needed
  ]
})
export class ProductsModule { }
```

6. **Products Routing (`products-routing.module.ts`)**  
   Define child routes specific to the `ProductsModule`.

```typescript
// products-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ProductsComponent } from './components/products/products.component';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';

const routes: Routes = [
  { path: '', component: ProductsComponent },
  { path: ':id', component: ProductDetailComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class ProductsRoutingModule { }
```

### Lazy Loading Explained
With the above setup, the feature modules such as `ProductsModule`, `HomeModule`, `CartModule`, etc., will only be loaded when the user navigates to their respective routes, improving the initial load performance of the application.

- The `AppModule` is kept minimal, containing only the root component and `CoreModule`.
- `CoreModule` is only imported once, preventing multiple instances of services like `AuthService`.
- `SharedModule` is used across feature modules without polluting `AppModule`.

By implementing this structure, you enhance performance with lazy loading and follow best practices in Angular for modular architecture.
