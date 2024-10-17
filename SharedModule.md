The SharedModule contains various types of reusable code such as components, directives, pipes, and other shared logic.
This module can be imported into different feature modules, providing them with access to the shared functionality without the need for duplication.

### Structure of Shared Module

```bash
src/
├── app/
│   ├── shared/                         
│   │   ├── components/                # Shared reusable components
│   │   │   ├── shared-button/         # A shared button component
│   │   │   │   ├── shared-button.component.ts
│   │   │   │   ├── shared-button.component.html
│   │   │   ├── confirm-dialog/        # A shared confirmation dialog component
│   │   │   │   ├── confirm-dialog.component.ts
│   │   │   │   ├── confirm-dialog.component.html
│   │   │
│   │   ├── directives/                # Shared reusable directives
│   │   │   ├── highlight.directive.ts # A directive that highlights text
│   │   │
│   │   ├── pipes/                     # Shared reusable pipes
│   │   │   ├── capitalize.pipe.ts     # A pipe to capitalize text
│   │   │   ├── date-format.pipe.ts    # A pipe to format dates
│   │   │
│   │   ├── services/                  # Shared services (if any)
│   │   │   ├── notification.service.ts  # A service to manage notifications
│   │   │
│   │   ├── shared.module.ts           # SharedModule definition
│   │   └── index.ts                   # Barrel file for SharedModule exports
```

---

### 1. **Shared Components**

#### **Shared Button Component (`shared-button.component.ts`)**

This button component is a reusable button that you can use across multiple modules.

```typescript
// shared-button.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-shared-button',
  template: `<button [ngStyle]="{'background-color': color}" (click)="onClick()">
               {{ label }}
             </button>`,
  styles: ['button { padding: 10px 20px; border: none; cursor: pointer; }']
})
export class SharedButtonComponent {
  @Input() label: string = 'Click Me';
  @Input() color: string = '#007bff';  // Default color is blue
  
  onClick() {
    console.log('Button clicked!');
  }
}
```

#### **Shared Button HTML (`shared-button.component.html`)**

```html
<!-- shared-button.component.html -->
<button [ngStyle]="{'background-color': color}" (click)="onClick()">
  {{ label }}
</button>
```

#### **Confirmation Dialog Component (`confirm-dialog.component.ts`)**

A confirmation dialog component that can be used to confirm user actions.

```typescript
// confirm-dialog.component.ts
import { Component, Inject } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';

@Component({
  selector: 'app-confirm-dialog',
  templateUrl: './confirm-dialog.component.html'
})
export class ConfirmDialogComponent {
  constructor(
    public dialogRef: MatDialogRef<ConfirmDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public data: any
  ) {}

  onNoClick(): void {
    this.dialogRef.close(false);
  }

  onYesClick(): void {
    this.dialogRef.close(true);
  }
}
```

#### **Confirmation Dialog HTML (`confirm-dialog.component.html`)**

```html
<!-- confirm-dialog.component.html -->
<h1 mat-dialog-title>{{ data.title }}</h1>
<div mat-dialog-content>{{ data.message }}</div>
<div mat-dialog-actions>
  <button mat-button (click)="onNoClick()">No</button>
  <button mat-button color="primary" (click)="onYesClick()">Yes</button>
</div>
```

### 2. **Shared Directives**

#### **Highlight Directive (`highlight.directive.ts`)**

This directive highlights text with a specified color when the mouse hovers over it.

```typescript
// highlight.directive.ts
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() highlightColor: string = 'yellow';

  constructor(private el: ElementRef) {}

  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.highlightColor);
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(null);
  }

  private highlight(color: string | null) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

### 3. **Shared Pipes**

#### **Capitalize Pipe (`capitalize.pipe.ts`)**

A pipe that capitalizes the first letter of each word.

```typescript
// capitalize.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'capitalize'
})
export class CapitalizePipe implements PipeTransform {
  transform(value: string): string {
    if (!value) return value;
    return value.replace(/\b\w/g, (char) => char.toUpperCase());
  }
}
```

#### **Date Format Pipe (`date-format.pipe.ts`)**

A pipe that formats dates using Angular's built-in `DatePipe` but with a default format.

```typescript
// date-format.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';
import { DatePipe } from '@angular/common';

@Pipe({
  name: 'dateFormat'
})
export class DateFormatPipe implements PipeTransform {
  constructor(private datePipe: DatePipe) {}

  transform(value: Date | string | number, format: string = 'MMM d, y'): string | null {
    return this.datePipe.transform(value, format);
  }
}
```

### 4. **Shared Services**

#### **Notification Service (`notification.service.ts`)**

A service to handle notifications across the application.

```typescript
// notification.service.ts
import { Injectable } from '@angular/core';
import { MatSnackBar } from '@angular/material/snack-bar';

@Injectable({
  providedIn: 'root'
})
export class NotificationService {
  constructor(private snackBar: MatSnackBar) {}

  showSuccess(message: string) {
    this.snackBar.open(message, 'Close', {
      duration: 3000,
      panelClass: ['success-snackbar']
    });
  }

  showError(message: string) {
    this.snackBar.open(message, 'Close', {
      duration: 3000,
      panelClass: ['error-snackbar']
    });
  }
}
```

### 5. **SharedModule (`shared.module.ts`)**

Finally, we tie everything together in the `SharedModule`, which imports, declares, and exports the shared components, directives, pipes, and services. This module can be imported into any feature module that requires shared functionality.

```typescript
// shared.module.ts
import { NgModule } from '@angular/core';
import { CommonModule, DatePipe } from '@angular/common';
import { FormsModule } from '@angular/forms';

// Shared Components
import { SharedButtonComponent } from './components/shared-button/shared-button.component';
import { ConfirmDialogComponent } from './components/confirm-dialog/confirm-dialog.component';

// Shared Directives
import { HighlightDirective } from './directives/highlight.directive';

// Shared Pipes
import { CapitalizePipe } from './pipes/capitalize.pipe';
import { DateFormatPipe } from './pipes/date-format.pipe';

// Material modules (for dialog)
import { MatDialogModule } from '@angular/material/dialog';
import { MatButtonModule } from '@angular/material/button';

// Service example
import { NotificationService } from './services/notification.service';

@NgModule({
  declarations: [
    // Declare shared components
    SharedButtonComponent,
    ConfirmDialogComponent,
    
    // Declare shared directives
    HighlightDirective,
    
    // Declare shared pipes
    CapitalizePipe,
    DateFormatPipe
  ],
  imports: [
    CommonModule,
    FormsModule,
    
    // Import Angular Material modules (for dialogs, buttons)
    MatDialogModule,
    MatButtonModule
  ],
  exports: [
    // Export shared components
    SharedButtonComponent,
    ConfirmDialogComponent,
    
    // Export shared directives
    HighlightDirective,
    
    // Export shared pipes
    CapitalizePipe,
    DateFormatPipe,
    
    // Export Material modules for other modules to use
    MatDialogModule,
    MatButtonModule
  ],
  providers: [
    NotificationService,
    DatePipe  // Provide DatePipe if used in DateFormatPipe
  ]
})
export class SharedModule { }
```

### 6. **Barrel File (`index.ts`)**

A barrel file is useful to re-export the contents of the `SharedModule`. It provides a cleaner and more organized way to import items from the `SharedModule` into other modules.

```typescript
// index.ts
export * from './shared.module';
export * from './components/shared-button/shared-button.component';
export * from './components/confirm-dialog/confirm-dialog.component';
export * from './directives/highlight.directive';
export * from './pipes/capitalize.pipe';
export * from './pipes/date-format.pipe';
export * from './services/notification.service';
```

### Example of Usage in a Feature Module

Now, if you want to use the shared components, pipes, directives, or services in a feature module (e.g., `ProductsModule`), simply import the `SharedModule`.

```typescript
// products.module.ts
import { NgModule } from '@angular/core';
import

 { CommonModule } from '@angular/common';
import { ProductsRoutingModule } from './products-routing.module';
import { ProductListComponent } from './components/product-list/product-list.component';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';
import { SharedModule } from '../shared/shared.module';  // Import SharedModule

@NgModule({
  declarations: [
    ProductListComponent,
    ProductDetailComponent
  ],
  imports: [
    CommonModule,
    ProductsRoutingModule,
    SharedModule  // Now you can use SharedButtonComponent, pipes, etc.
  ]
})
export class ProductsModule { }
```

### Conclusion

This structure covers all common use cases of a shared module, including:

- **Reusable components** (e.g., buttons, dialogs)
- **Reusable directives** (e.g., highlight)
- **Reusable pipes** (e.g., capitalize, date formatting)
- **Shared services** (e.g., notifications)
  
By using the `SharedModule`, you can avoid code duplication across your app and ensure that common functionality is easy to reuse. The module can be imported into any feature module, and the components, directives, and pipes can be used anywhere in the application.
