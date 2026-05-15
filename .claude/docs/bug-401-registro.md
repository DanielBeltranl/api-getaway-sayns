# Bug: POST /api/registro retornaba 401

## Síntoma

`POST http://localhost:8081/api/registro` devolvía **401 Unauthorized** aunque el endpoint de registro no debería requerir token.

## Causa raíz

Eran **dos problemas encadenados**:

### 1. Route predicate no matcheaba la ruta exacta

```yaml
# application.yml — INCORRECTO
predicates:
  - Path=/api/registro/**
```

El patrón `/**` en Spring Cloud Gateway **no matchea** la ruta base sin trailing slash. Es decir, `/api/registro/**` captura `/api/registro/algo` pero **no** `/api/registro`.

Al no encontrar la ruta, el gateway lanzaba una excepción → Tomcat hacía un forward interno a `/error`.

### 2. El endpoint `/error` estaba protegido

El `SecurityFilterChain` no incluía `/error` en los paths públicos. El forward a `/error` lo procesaba el chain que requería JWT → **401**.

Por eso parecía un error de autenticación cuando en realidad era un error de routing.

> El `permitAll()` para `/api/registro` **sí funcionaba**. El request nunca llegaba al microservicio.

---

## Solución

### SecurityConfig — dos filter chains

```java
@Bean
@Order(1)
public SecurityFilterChain publicChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/api/registro", "/api/registro/**", "/actuator/**", "/error")
        .csrf(csrf -> csrf.disable())
        .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
    return http.build();
}

@Bean
@Order(2)
public SecurityFilterChain securedChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf.disable())
        .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
    return http.build();
}
```

### GatewayConfig — rutas programáticas

El YAML (`spring.cloud.gateway.mvc.routes`) no cargaba las rutas en Spring Cloud Gateway MVC 5.x. Se definieron programáticamente:

```java
@Bean
public RouterFunction<ServerResponse> apiRegistroRoute() {
    return GatewayRouterFunctions.route("api-registro")
            .route(
                    RequestPredicates.path("/api/registro").or(RequestPredicates.path("/api/registro/**")),
                    HandlerFunctions.http()
            )
            .filter(LoadBalancerFilterFunctions.lb("api-registro"))
            .build();
}
```

`LoadBalancerFilterFunctions.lb("api-registro")` resuelve el servicio desde Eureka. Sin este filtro, `lb://` no es un scheme que entienda el cliente HTTP subyacente.

---

## Cómo debuggear este tipo de problema

1. Medir el tiempo de respuesta: si es **< 5ms**, el gateway nunca hizo llamada al downstream.
2. Habilitar debug de Spring Security y buscar `Securing GET /error` en los logs — indica que el error handler está siendo procesado por el security chain.
3. Probar el downstream **directamente** (ej. `curl localhost:8080/api/registro`) para descartar que el 401 venga del microservicio.
