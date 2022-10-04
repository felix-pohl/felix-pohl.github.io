guard.spec.ts
```typescript
import { Location } from '@angular/common';
import { Component, NgModule } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { Router, Routes } from '@angular/router';
import { RouterTestingModule } from '@angular/router/testing';
import { of } from 'rxjs';
import { HappyGuard } from './happy.guard';
import { PermissionService } from './permission.service';


@Component({})
export class BlankComponent { }

@NgModule({})
export class BlankModule { }

describe('HappyGuard', () => {
  let router: Router;
  let location: Location;
  let mockPermissionService: PermissionService;

  const routes: Routes = [
    { path: "happy", component: BlankComponent, canActivate: [HappyGuard] },
    { path: "loadhappy", loadChildren: () => import('./happy.guard.spec').then(m => m.BlankModule), canLoad: [HappyGuard] },
    { path: "**", component: BlankComponent }
  ]

  beforeEach(async () => {
    mockPermissionService = {
      permissions$: of([])
    }
    await TestBed.configureTestingModule({
      imports: [
        RouterTestingModule.withRoutes(routes)
      ],
      providers: [
        { provide: PermissionService, useValue: mockPermissionService }
      ]
    });
    router = TestBed.inject(Router);
    location = TestBed.inject(Location);
  });

  it('should be created', () => {
    const guard = TestBed.inject(HappyGuard);
    expect(guard).toBeTruthy();
  });

  it('should be happy', async () => {
    mockPermissionService.permissions$ = of(['happieness']);
    await router.navigateByUrl("/happy")
    expect(location.path()).toBe('/happy')
  })

  it('should be unhappy', async () => {
    mockPermissionService.permissions$ = of(['unhappy']);
    await router.navigateByUrl("/happy")
    expect(location.path()).toBe('/error')
  })

  it('should be unhappy', async () => {
    mockPermissionService.permissions$ = of(['happieness']);
    await router.navigateByUrl("/loadhappy")
    expect(location.path()).toBe('/loadhappy')
  })

  it('should be unhappy', async () => {
    mockPermissionService.permissions$ = of(['unhappy']);
    await router.navigateByUrl("/loadhappy")
    expect(location.path()).toBe('/error')
  })
});
```

guard.ts
```typescript
import { Injectable } from '@angular/core';
import { ActivatedRouteSnapshot, CanActivate, CanLoad, Route, Router, RouterStateSnapshot, UrlSegment, UrlTree } from '@angular/router';
import { map, Observable } from 'rxjs';
import { PermissionService } from './permission.service';

@Injectable({
  providedIn: 'root'
})
export class HappyGuard implements CanActivate, CanLoad {

  constructor(private readonly router: Router, private readonly permissionService: PermissionService) { }

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return this.permissionService.permissions$.pipe(map(
      value => {
        if (value.indexOf('happieness') !== -1) {
          return true;
        }
        return this.router.parseUrl('/error');
      }
    ))
  }

  canLoad(
    route: Route,
    segments: UrlSegment[]): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return this.permissionService.permissions$.pipe(map(
      value => {
        if (value.indexOf('happieness') !== -1) {
          return true;
        }
        return this.router.parseUrl('/error');
      }
    ))
  }
}
```
