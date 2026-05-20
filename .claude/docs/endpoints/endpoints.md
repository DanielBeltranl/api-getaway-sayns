
## Api registro

**POST** — no requiere token

Ruta: `http://localhost:8081/api/registro`

Body:
```json
{
    "email": "string",
    "password": "string",
    "phone": number,
    "userType": "string",
    "refugioName": "string",
    "adress": "string"
}
```

Respuesta:
```json
{
    "error": "string",
    "message": "string",
    "status": number,
    "token": "string"
}
```

---

## Api login

**POST** — no requiere token

Ruta: `http://localhost:8081/api/login`

Body:
```json
{
    "email": "string",
    "password": "string"
}
```

Respuesta:
```json
{
    "error": "string",
    "message": "string",
    "status": number,
    "token": "string"
}
```

---

## Reportes

**GET** — requiere token JWT (Bearer)

Ruta: `http://localhost:8081/reportes`

Respuesta: definida por `bff-sanys`

---

## Reportes — Crear

**POST** — requiere token JWT (Bearer)

Ruta: `http://localhost:8081/reportes/crear`

Body: definido por `bff-sanys`

Respuesta: definida por `bff-sanys`






