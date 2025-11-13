# Angular HMS Dashboard (NgModule + NgRx + Tailwind) — Full Setup (from scratch)

> Non‑standalone Angular app using NgModules, with routing, Tailwind UI, and NgRx (Store, Effects, Router Store, DevTools). Code includes beginner‑friendly inline comments in **TypeScript files**.

---

## 1) Prerequisites
- Node LTS (≥ 18)
- Angular CLI (latest): `npm i -g @angular/cli`

---

## 2) Create project (non‑standalone) + routing + Tailwind
```bash
# 2.1 Create a new Angular app (NgModule-based, not standalone)
ng new hms-dashboard \
  --routing \
  --style=css \
  --no-standalone

cd hms-dashboard
 
```

---

##  Install Tailwind CSS & PostCSS

Run the following command in your project root:

```bash
npm install tailwindcss @tailwindcss/postcss postcss --force
```

---

##  Configure PostCSS Plugins

Create a file named **`.postcssrc.json`** in the **root** of your project and add this configuration:

```json
{
  "plugins": {
    "@tailwindcss/postcss": {}
  }
}
```

---

##  Import Tailwind CSS

Open or create your **`src/styles.css`** file and add the Tailwind import:

```css
@import "tailwindcss";
```

---

 

---

## 3) Install NgRx packages
```bash
# Core NgRx libs
ng add @ngrx/store@latest
ng add @ngrx/effects@latest
ng add @ngrx/store-devtools@latest
ng add @ngrx/router-store@latest

# Optional for entity helpers (not required for this demo)
npm i @ngrx/entity
```

---

## 4) Recommended folder structure (high level)
```
src/
  app/
    core/                    # Singleton services, interceptors, guards
    shared/                  # Reusable components/pipes/directives
    layout/                  # Shell layout + navbar/sidebar
    features/
      opd/                   # Out Patient Dept feature module
      labs/                  # Labs feature module
      billing/               # Billing feature module
      inventory/             # Inventory feature module
      hr/                    # HR feature module
    store/                   # Global NgRx state
      auth/                  # current user + role
      patient/               # current patient context
      notifications/         # system notifications
      app.state.ts
      app.reducers.ts
    app-routing-module.ts
    app-module.ts
    environments/
      environment.ts
      environment.development.ts
```

Create folders:
```bash
mkdir -p src/app/{core,shared,layout,features/{opd,labs,billing,inventory,hr},store/{auth,patient,notifications},environments}
```

---

## 5) Environments

**`src/app/environments/environment.development.ts`**
```ts
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:3000' // mock API base (json-server or your backend)
};
```

**`src/app/environments/environment.ts`**
```ts
export const environment = {
  production: true,
  apiBaseUrl: 'https://api.your-hms.com' // replace with real prod API
};
```

Update `tsconfig.app.json` path mapping (optional) if you like `@env/` alias; we’ll use relative imports to keep it simple.

---

## 6) Layout: Shell + Navbar

Generate a layout module and components:
```bash
ng g m layout --flat=false --module app-module
ng g c layout/shell --export
ng g c layout/navbar --export
```

**`src/app/layout/shell/shell.component.html`**
```html
<div class="min-h-screen bg-gray-50 flex">
  <!-- Sidebar (simple) -->
  <aside class="w-64 bg-white border-r hidden md:block">
    <div class="p-4 font-semibold text-xl">MediHub HMS</div>
    <nav class="px-2 space-y-1">
      <a routerLink="/opd" routerLinkActive="bg-gray-100" class="block px-3 py-2 rounded">OPD</a>
      <a routerLink="/labs" routerLinkActive="bg-gray-100" class="block px-3 py-2 rounded">Labs</a>
      <a routerLink="/billing" routerLinkActive="bg-gray-100" class="block px-3 py-2 rounded">Billing</a>
      <a routerLink="/inventory" routerLinkActive="bg-gray-100" class="block px-3 py-2 rounded">Inventory</a>
      <a routerLink="/hr" routerLinkActive="bg-gray-100" class="block px-3 py-2 rounded">HR</a>
    </nav>
  </aside>

  <!-- Main content -->
  <div class="flex-1 flex flex-col">
    <app-navbar></app-navbar>
    <main class="p-4">
      <router-outlet></router-outlet>
    </main>
  </div>
</div>
```

**`src/app/layout/navbar/navbar.component.html`**
```html
<header class="bg-white border-b">
  <div class="max-w-7xl mx-auto px-4 h-14 flex items-center justify-between">
    <div class="flex items-center gap-3">
      <button class="md:hidden px-2 py-1 border rounded">Menu</button>
      <span class="font-semibold">Enterprise HMS Dashboard</span>
    </div>
    <div class="flex items-center gap-4">
      <!-- Current patient from store (read-only display) -->
      <div class="text-sm text-gray-600" *ngIf="currentPatient$ | async as p">Patient: <strong>{{ p?.name || 'None' }}</strong></div>

      <!-- User role from store controls visibility of Admin menus -->
      <ng-container *ngIf="userRole$ | async as role">
        <a *ngIf="role === 'ADMIN'" routerLink="/hr" class="text-sm underline">Admin</a>
      </ng-container>
    </div>
  </div>
</header>
```

**`src/app/layout/navbar/navbar.component.ts`**
```ts
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { selectCurrentPatient } from '../../store/patient/patient.selectors';
import { selectUserRole } from '../../store/auth/auth.selectors';
import { Patient } from '../../store/patient/patient.models';

@Component({
  selector: 'app-navbar',
  templateUrl: './navbar.component.html'
})
export class NavbarComponent {
  // Expose slices of state to template via Observables
  currentPatient$: Observable<Patient | null> = this.store.select(selectCurrentPatient);
  userRole$: Observable<'ADMIN' | 'DOCTOR' | 'NURSE' | 'BILLING' | 'GUEST'> = this.store.select(selectUserRole);

  constructor(private store: Store) {}
}
```

---

## 7) Feature modules
Generate example feature modules + routes + example components:
```bash
ng g m features/opd --route opd --module app-routing-module
ng g m features/labs --route labs --module app-routing-module
ng g m features/billing --route billing --module app-routing-module
ng g m features/inventory --route inventory --module app-routing-module
ng g m features/hr --route hr --module app-routing-module

# Sample pages inside features
ng g c features/opd/pages/opd-home --module features/opd
ng g c features/labs/pages/labs-home --module features/labs
ng g c features/billing/pages/billing-home --module features/billing
ng g c features/inventory/pages/inventory-home --module features/inventory
ng g c features/hr/pages/hr-home --module features/hr
```

**Example: `src/app/features/opd/opd-routing.module.ts`**
```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { OpdHomeComponent } from './pages/opd-home/opd-home.component';

const routes: Routes = [
  { path: '', component: OpdHomeComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class OpdRoutingModule {}
```

**Example: `src/app/features/opd/pages/opd-home/opd-home.component.html`**
```html
<div class="space-y-4">
  <h1 class="text-2xl font-semibold">OPD</h1>

  <div class="grid md:grid-cols-2 gap-4">
    <section class="p-4 bg-white border rounded">
      <h2 class="font-medium mb-2">Current Patient</h2>
      <div class="text-sm text-gray-600">
        <!-- Display currently selected patient from global store -->
        <ng-container *ngIf="currentPatient$ | async as p; else np">
          <div><strong>{{ p.name }}</strong> ({{ p.id }})</div>
          <div>Age: {{ p.age }} | Gender: {{ p.gender }}</div>
        </ng-container>
        <ng-template #np>None selected</ng-template>
      </div>
      <button (click)="selectDummyPatient()" class="mt-3 px-3 py-2 border rounded">Select Demo Patient</button>
    </section>

    <section class="p-4 bg-white border rounded">
      <h2 class="font-medium mb-2">Notifications</h2>
      <button (click)="loadNotifications()" class="px-3 py-2 border rounded">Load</button>
      <ul class="list-disc ml-6 mt-2" *ngIf="notifications$ | async as notes">
        <li *ngFor="let n of notes">{{ n.message }}</li>
      </ul>
    </section>
  </div>
</div>
```

**`src/app/features/opd/pages/opd-home/opd-home.component.ts`**
```ts
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { Patient } from '../../../../store/patient/patient.models';
import { selectCurrentPatient } from '../../../../store/patient/patient.selectors';
import { setCurrentPatient } from '../../../../store/patient/patient.actions';
import { loadNotifications } from '../../../../store/notifications/notifications.actions';
import { selectAllNotifications } from '../../../../store/notifications/notifications.selectors';
import { Notification } from '../../../../store/notifications/notifications.models';

@Component({
  selector: 'app-opd-home',
  templateUrl: './opd-home.component.html'
})
export class OpdHomeComponent {
  // Selecting state slices to render in template
  currentPatient$: Observable<Patient | null> = this.store.select(selectCurrentPatient);
  notifications$: Observable<Notification[]> = this.store.select(selectAllNotifications);

  constructor(private store: Store) {}

  // Beginner note: dispatching an action is how we "request" a state change.
  selectDummyPatient() {
    const demo: Patient = { id: 'P-1001', name: 'John Carter', age: 42, gender: 'M' };
    this.store.dispatch(setCurrentPatient({ patient: demo }));
  }

  // Trigger an Effect to load notifications from API
  loadNotifications() {
    this.store.dispatch(loadNotifications());
  }
}
```

---

## 8) App routing to use the shell layout

**`src/app/app-routing-module.ts`**
```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ShellComponent } from './layout/shell/shell.component';

const routes: Routes = [
  {
    path: '',
    component: ShellComponent,
    children: [
      { path: 'opd', loadChildren: () => import('./features/opd/opd.module').then(m => m.OpdModule) },
      { path: 'labs', loadChildren: () => import('./features/labs/labs.module').then(m => m.LabsModule) },
      { path: 'billing', loadChildren: () => import('./features/billing/billing.module').then(m => m.BillingModule) },
      { path: 'inventory', loadChildren: () => import('./features/inventory/inventory.module').then(m => m.InventoryModule) },
      { path: 'hr', loadChildren: () => import('./features/hr/hr.module').then(m => m.HrModule) },
      { path: '', pathMatch: 'full', redirectTo: 'opd' }
    ]
  },
  { path: '**', redirectTo: '' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

---

## 9) Global NgRx state

### 9.1 App root reducers combiner

**`src/app/store/app.state.ts`**
```ts
// Central place to describe our root state tree shape
import { AuthState } from './auth/auth.models';
import { PatientState } from './patient/patient.models';
import { NotificationsState } from './notifications/notifications.models';

export interface AppState {
  auth: AuthState;
  patient: PatientState;
  notifications: NotificationsState;
}
```

**`src/app/store/app.reducers.ts`**
```ts
import { ActionReducerMap } from '@ngrx/store';
import { AppState } from './app.state';
import { authReducer } from './auth/auth.reducer';
import { patientReducer } from './patient/patient.reducer';
import { notificationsReducer } from './notifications/notifications.reducer';

export const appReducers: ActionReducerMap<AppState> = {
  auth: authReducer,
  patient: patientReducer,
  notifications: notificationsReducer
};
```

### 9.2 Auth slice (user + role)

**`src/app/store/auth/auth.models.ts`**
```ts
export interface User {
  id: string;
  name: string;
  role: 'ADMIN' | 'DOCTOR' | 'NURSE' | 'BILLING' | 'GUEST';
}

export interface AuthState {
  user: User | null; // null = not logged in
}
```

**`src/app/store/auth/auth.actions.ts`**
```ts
import { createAction, props } from '@ngrx/store';
import { User } from './auth.models';

// User logs in successfully
export const loginSuccess = createAction('[Auth] Login Success', props<{ user: User }>());

// User logs out
export const logout = createAction('[Auth] Logout');
```

**`src/app/store/auth/auth.reducer.ts`**
```ts
import { createReducer, on } from '@ngrx/store';
import { AuthState } from './auth.models';
import { loginSuccess, logout } from './auth.actions';

const initialState: AuthState = {
  user: { id: 'U-1', name: 'Dr. Smith', role: 'DOCTOR' } // demo default; replace with real auth flow
};

export const authReducer = createReducer(
  initialState,
  on(loginSuccess, (state, { user }) => ({ ...state, user })),
  on(logout, () => ({ user: null }))
);
```

**`src/app/store/auth/auth.selectors.ts`**
```ts
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { AuthState } from './auth.models';

const selectAuth = createFeatureSelector<AuthState>('auth');

export const selectUser = createSelector(selectAuth, s => s.user);
export const selectUserRole = createSelector(selectUser, u => u?.role ?? 'GUEST');
```

### 9.3 Patient slice (current patient)

**`src/app/store/patient/patient.models.ts`**
```ts
export interface Patient {
  id: string;
  name: string;
  age: number;
  gender: 'M' | 'F' | 'O';
}

export interface PatientState {
  current: Patient | null; // current patient in context across modules
}
```

**`src/app/store/patient/patient.actions.ts`**
```ts
import { createAction, props } from '@ngrx/store';
import { Patient } from './patient.models';

export const setCurrentPatient = createAction('[Patient] Set Current', props<{ patient: Patient }>());
export const clearCurrentPatient = createAction('[Patient] Clear Current');
```

**`src/app/store/patient/patient.reducer.ts`**
```ts
import { createReducer, on } from '@ngrx/store';
import { PatientState } from './patient.models';
import { setCurrentPatient, clearCurrentPatient } from './patient.actions';

const initialState: PatientState = {
  current: null
};

export const patientReducer = createReducer(
  initialState,
  on(setCurrentPatient, (state, { patient }) => ({ ...state, current: patient })),
  on(clearCurrentPatient, () => ({ current: null }))
);
```

**`src/app/store/patient/patient.selectors.ts`**
```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { PatientState } from './patient.models';

const selectPatient = createFeatureSelector<PatientState>('patient');
export const selectCurrentPatient = createSelector(selectPatient, s => s.current);
```

### 9.4 Notifications slice (with Effect)

**`src/app/store/notifications/notifications.models.ts`**
```ts
export interface Notification {
  id: string;
  message: string;
}

export interface NotificationsState {
  items: Notification[];
  loading: boolean;
}
```

**`src/app/store/notifications/notifications.actions.ts`**
```ts
import { createAction, props } from '@ngrx/store';
import { Notification } from './notifications.models';

export const loadNotifications = createAction('[Notifications] Load');
export const loadNotificationsSuccess = createAction('[Notifications] Load Success', props<{ items: Notification[] }>());
export const loadNotificationsFailure = createAction('[Notifications] Load Failure', props<{ error: unknown }>());
```

**`src/app/store/notifications/notifications.reducer.ts`**
```ts
import { createReducer, on } from '@ngrx/store';
import { NotificationsState } from './notifications.models';
import { loadNotifications, loadNotificationsSuccess, loadNotificationsFailure } from './notifications.actions';

const initialState: NotificationsState = {
  items: [],
  loading: false
};

export const notificationsReducer = createReducer(
  initialState,
  on(loadNotifications, (s) => ({ ...s, loading: true })),
  on(loadNotificationsSuccess, (s, { items }) => ({ ...s, loading: false, items })),
  on(loadNotificationsFailure, (s) => ({ ...s, loading: false }))
);
```

**`src/app/store/notifications/notifications.selectors.ts`**
```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { NotificationsState } from './notifications.models';

const selectNotifications = createFeatureSelector<NotificationsState>('notifications');
export const selectAllNotifications = createSelector(selectNotifications, s => s.items);
export const selectNotificationsLoading = createSelector(selectNotifications, s => s.loading);
```

**Effect + Service to fetch notifications**

```bash
ng g s core/api --skip-tests
ng g e store/notifications/notifications --flat --skip-tests
```

**`src/app/core/api.service.ts`**
```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Notification } from '../store/notifications/notifications.models';
import { environment } from '../environments/environment.development';

@Injectable({ providedIn: 'root' })
export class ApiService {
  // Beginner tip: keep all HTTP calls centralized here; components never call HttpClient directly
  private base = environment.apiBaseUrl;
  constructor(private http: HttpClient) {}

  getNotifications(): Observable<Notification[]> {
    return this.http.get<Notification[]>(`${this.base}/notifications`);
  }
}
```

**`src/app/store/notifications/notifications.effects.ts`**
```ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { ApiService } from '../../core/api.service';
import { loadNotifications, loadNotificationsFailure, loadNotificationsSuccess } from './notifications.actions';
import { catchError, map, mergeMap, of } from 'rxjs';

@Injectable()
export class NotificationsEffects {
  // Effects listen for actions and perform side-effects (like HTTP) then dispatch new actions
  load$ = createEffect(() => this.actions$.pipe(
    ofType(loadNotifications),
    mergeMap(() => this.api.getNotifications().pipe(
      map(items => loadNotificationsSuccess({ items })),
      catchError(error => of(loadNotificationsFailure({ error })))
    ))
  ));

  constructor(private actions$: Actions, private api: ApiService) {}
}
```

Register effects in root (next section).

---

## 10) Root AppModule with Store, Effects, RouterStore, DevTools

**`src/app/app-module.ts`**
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { AppRoutingModule } from './app-routing-module';
import { AppComponent } from './app.component';

// Layout
import { LayoutModule } from './layout/layout.module';

// NgRx
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { StoreRouterConnectingModule } from '@ngrx/router-store';

import { appReducers } from './store/app.reducers';
import { NotificationsEffects } from './store/notifications/notifications.effects';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    HttpClientModule,
    AppRoutingModule,
    LayoutModule,

    // Register root store and effects
    StoreModule.forRoot(appReducers),
    EffectsModule.forRoot([NotificationsEffects]),

    // Sync router state with store
    StoreRouterConnectingModule.forRoot(),

    // Time-travel debugging in dev only
    StoreDevtoolsModule.instrument({ maxAge: 25 })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

**`src/app/app.component.html`**
```html
<!-- Shell wraps feature routes and navbar -->
<app-shell></app-shell>
```

---

## 11) Guards (role-based example using store)

```bash
ng g g core/role --skip-tests
```

**`src/app/core/role.guard.ts`**
```ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, Router, UrlTree } from '@angular/router';
import { Store } from '@ngrx/store';
import { Observable, map } from 'rxjs';
import { selectUserRole } from '../store/auth/auth.selectors';

@Injectable({ providedIn: 'root' })
export class RoleGuard implements CanActivate {
  constructor(private store: Store, private router: Router) {}

  // Allow only if the current user role is within route.data['roles'] array
  canActivate(route: ActivatedRouteSnapshot): Observable<boolean | UrlTree> {
    const allowed = route.data['roles'] as string[];
    return this.store.select(selectUserRole).pipe(
      map(role => allowed.includes(role) ? true : this.router.createUrlTree(['/opd']))
    );
  }
}
```

Apply it to an admin route (e.g., HR module) in **`app-routing-module.ts`**:
```ts
{ path: 'hr', loadChildren: () => import('./features/hr/hr.module').then(m => m.HrModule), canActivate: [RoleGuard], data: { roles: ['ADMIN'] } },
```

---

## 12) Minimal feature module example (HR)

**`src/app/features/hr/hr.module.ts`**
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HrRoutingModule } from './hr-routing.module';
import { HrHomeComponent } from './pages/hr-home/hr-home.component';

@NgModule({
  declarations: [HrHomeComponent],
  imports: [CommonModule, HrRoutingModule]
})
export class HrModule {}
```

**`src/app/features/hr/hr-routing.module.ts`**
```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HrHomeComponent } from './pages/hr-home/hr-home.component';

const routes: Routes = [
  { path: '', component: HrHomeComponent }
];

@NgModule({ imports: [RouterModule.forChild(routes)], exports: [RouterModule] })
export class HrRoutingModule {}
```

**`src/app/features/hr/pages/hr-home/hr-home.component.ts`**
```ts
import { Component } from '@angular/core';

@Component({ selector: 'app-hr-home', templateUrl: './hr-home.component.html' })
export class HrHomeComponent {}
```

**`src/app/features/hr/pages/hr-home/hr-home.component.html`**
```html
<div class="p-4 bg-white border rounded">
  <h1 class="text-2xl font-semibold">HR (Admins only)</h1>
  <p class="text-gray-600">This route is protected by RoleGuard using NgRx auth state.</p>
</div>
```

---

## 13) Mock API (optional) with json-server
```bash
npm i -D json-server

# Create a db.json in project root
cat > db.json << 'JSON'
{
  "notifications": [
    { "id": "N1", "message": "Appointment #A-902 scheduled" },
    { "id": "N2", "message": "Lab result ready for P-1001" }
  ]
}
JSON

# Run mock API on port 3000
npx json-server --watch db.json --port 3000
```

---

## 14) Run the app
```bash
npm start  # or: ng serve -o
```

You should see the shell, sidebar nav, role-sensitive Admin link, current patient placeholder, and the OPD page where you can:
- Click **Select Demo Patient** to set global current patient
- Click **Load** to fetch notifications via NgRx Effect

---

## 15) Why NgRx here
- **Predictable state** across many HMS modules.
- **Traceable actions** enable debugging complex workflows.
- **Time‑travel DevTools** for investigating issues.
- **Effects** isolate side‑effects like HTTP calls.
- **Selectors** give performant, testable reads from global state.

---

## 16) Next steps for production
- Replace demo auth with real JWT login flow; dispatch `loginSuccess` on successful login.
- Normalize complex entity data with `@ngrx/entity` (patients, bills, lab orders, inventory items).
- Add feature-level stores (e.g., LabsState) and lazy‑load their reducers with `StoreModule.forFeature`.
- Introduce router‑store selectors to react to route params (patientId, visitId).
- Add unit tests for reducers/selectors/effects.
- Add a global error handler and HTTP interceptors (auth token, error mapping).
- Add persistence (e.g., store some slices to localStorage) using metareducers.

---

## 17) Cheatsheet of CLI commands used
```bash
ng new hms-dashboard --routing --style=css --no-standalone
npm i -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
ng add @ngrx/store@latest
ng add @ngrx/effects@latest
ng add @ngrx/store-devtools@latest
ng add @ngrx/router-store@latest
ng g m layout --module app-module
ng g c layout/shell --export
ng g c layout/navbar --export
ng g m features/opd --route opd --module app-routing-module
ng g m features/labs --route labs --module app-routing-module
ng g m features/billing --route billing --module app-routing-module
ng g m features/inventory --route inventory --module app-routing-module
ng g m features/hr --route hr --module app-routing-module
ng g c features/opd/pages/opd-home --module features/opd
ng g c features/labs/pages/labs-home --module features/labs
ng g c features/billing/pages/billing-home --module features/billing
ng g c features/inventory/pages/inventory-home --module features/inventory
ng g c features/hr/pages/hr-home --module features/hr
ng g s core/api --skip-tests
ng g e store/notifications/notifications --flat --skip-tests
ng g g core/role --skip-tests
```

---

This gives you a clean, NgModule‑based Angular HMS scaffold with Tailwind UI, router, shell layout, and NgRx wired for global state (auth, patient, notifications) ready to expand across OPD/IPD/Labs/Billing/Inventory/HR.

