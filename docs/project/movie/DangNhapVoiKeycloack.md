---
comments: true
---
# Hướng dẫn đăng nhập SSO cho Angular với Keycloack

## 1. Cấu hình keycloak

Ví dụ tạo client web-movie

Trong môi trường test cấu hình *, còn môi trường production nên ràng buộc đúng domain của dự án để bảo mật

- Client ID *: web-movie
- Valid redirect URIs: *
- Valid post logout redirect URIs: *
- Web origins: *

- Client authentication: off
- Authorization: off

- Authentication flow:
- Standard flow: check
- Direct access grants: check
- Implicit flow: check


## 2. Cấu hình angluar 

Sử dụng thư viện keycloak-angular [https://github.com/mauriciovigolo/keycloak-angular.git](https://github.com/mauriciovigolo/keycloak-angular.git) để đăng nhập SSO thông qua keycloack


Thêm cấu hình môi trường

```typescript   title="environment.ts"  linenums="1"
export const environment = {
  production: true,
  apiUrl: 'http://localhost:9080',
  keycloak: {
    config: {
      url: 'http://auth-production.reb.com', // URL of the Keycloak server  url: 'http://auth.reb.com:8080', // URL of the Keycloak server
      realm: 'reb', // Realm to be used in Keycloak
      clientId: 'web-movie' // Client ID for the application in Keycloak
    }
  }
};

```

Thêm cấu hình khởi tạo keycloack

```typescript   title="auth.config.ts"  linenums="1"
// Function to initialize Keycloak with the necessary configurations
import {KeycloakBearerInterceptor, KeycloakService} from "keycloak-angular";
import {APP_INITIALIZER, Provider} from "@angular/core";
import {HTTP_INTERCEPTORS} from "@angular/common/http";
import {environment} from "@environments/environment";

function initializeKeycloak(keycloak: KeycloakService) {
  return () => {
    if (!globalThis.window) {
      return true;
    }
    return keycloak.init({
      // Configuration details for Keycloak
      config: {
        url: environment.keycloak.config.url, // URL of the Keycloak server  url: 'http://auth.reb.com:8080', // URL of the Keycloak server
        realm: environment.keycloak.config.realm, // Realm to be used in Keycloak
        clientId:  environment.keycloak.config.clientId // Client ID for the application in Keycloak
      },
      // Options for Keycloak initialization
      initOptions: {
        onLoad: 'check-sso', // Action to take on load
        silentCheckSsoRedirectUri:
          window.location.origin + '/silent-check-sso.html' // URI for silent SSO checks
      },
      // Enables Bearer interceptor
      enableBearerInterceptor: true,
      // Prefix for the Bearer token
      bearerPrefix: 'Bearer',
      // URLs excluded from Bearer token addition (empty by default)
      //bearerExcludedUrls: []
    });
  };
}

// Provider for Keycloak Bearer Interceptor
export const KeycloakBearerInterceptorProvider: Provider = {
  provide: HTTP_INTERCEPTORS,
  useClass: KeycloakBearerInterceptor,
  multi: true
};

// Provider for Keycloak Initialization
export const KeycloakInitializerProvider: Provider = {
  provide: APP_INITIALIZER,
  useFactory: initializeKeycloak,
  multi: true,
  deps: [KeycloakService]
}


```


Thêm provider của keycloack


```typescript   title="app.config.ts"  linenums="1"
import {ApplicationConfig, provideZoneChangeDetection} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';
import {provideClientHydration} from '@angular/platform-browser';
import {provideAnimationsAsync} from '@angular/platform-browser/animations/async';
import {KeycloakService} from "keycloak-angular";
import {provideHttpClient, withFetch, withInterceptorsFromDi} from "@angular/common/http";
import {KeycloakBearerInterceptorProvider, KeycloakInitializerProvider} from "@app/auth.config";


export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptorsFromDi(), withFetch()), // Provides HttpClient with interceptors
    KeycloakInitializerProvider, // Initializes Keycloak
    KeycloakBearerInterceptorProvider, // Provides Keycloak Bearer Interceptor
    KeycloakService, // Service for Keycloak

    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideClientHydration(),
    provideAnimationsAsync(),


  ]
};


```

Tham khảo: 

[https://github.com/vanphuoc9/movie.git](https://github.com/vanphuoc9/movie.git)

[https://github.com/mauriciovigolo/keycloak-angular#readme](https://github.com/mauriciovigolo/keycloak-angular#readme)





