# Problema 01 — Registro de usuarios sin estados inválidos

## Contexto

Estás construyendo el registro de usuarios de una aplicación.

Cada usuario tiene:

```text
id
email
name
created_at
```

## Problema

El sistema está aceptando:

- emails vacíos;
- nombres vacíos;
- emails repetidos;
- emails con espacios;
- emails con mayúsculas y minúsculas tratadas de forma inconsistente.

## Misión

Diseña:

```http
POST /users
```

de forma que los usuarios creados siempre sean válidos.

## Invariantes

```text
email != null
email != ""
name != ""
email debe ser único
```

## Preguntas de razonamiento

- ¿Qué valida la API?
- ¿Qué debe garantizar PostgreSQL?
- ¿Qué significa realmente que un email sea único?
- ¿Debes normalizar el email?
- ¿Qué error HTTP devolverías por duplicados?
- ¿Qué pasa si dos requests intentan registrar el mismo email casi al mismo tiempo?

## Conceptos para investigar

```text
input validation
normalization
UNIQUE
NOT NULL
CHECK constraints
HTTP 400
HTTP 409
```

## Pruebas mínimas

- usuario válido;
- email vacío;
- nombre vacío;
- email duplicado;
- email con espacios;
- email con diferente capitalización.

## Senior Questions

- ¿La unicidad debería ser case-sensitive?
- ¿Qué parte de la regla pertenece al dominio y cuál al almacenamiento?
- ¿Qué cambiaría si existieran múltiples proveedores de autenticación?

---
