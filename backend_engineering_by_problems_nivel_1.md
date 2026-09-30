# Backend Engineering by Problems
## Nivel 1 — Fundamentos de razonamiento para Backend Engineers

> **Objetivo:** aprender Backend Engineering resolviendo problemas reales, no memorizando tecnologías.

Este nivel está diseñado para construir una base sólida de razonamiento técnico. Cada reto parte de una situación realista y obliga a pensar en requisitos, invariantes, modelado, validación, persistencia, errores, contratos HTTP y calidad del diseño.

La tecnología es secundaria. El problema manda.

---

# 1. Metodología

Cada reto sigue el mismo ciclo:

```text
Situación real
    ↓
Entender el problema
    ↓
Identificar invariantes
    ↓
Diseñar una solución
    ↓
Implementar una versión simple
    ↓
Intentar romperla
    ↓
Investigar conceptos
    ↓
Refactorizar
    ↓
Explicar decisiones y trade-offs
```

La finalidad no es solamente que el código funcione.

Debes llegar a poder responder:

- ¿Qué problema estoy resolviendo realmente?
- ¿Qué condiciones nunca deben romperse?
- ¿Qué responsabilidad pertenece a la API?
- ¿Qué responsabilidad pertenece a la base de datos?
- ¿Qué errores son esperables?
- ¿Qué estados son inválidos?
- ¿Qué puede ocurrir si el cliente envía datos incorrectos?
- ¿Qué diseño sería difícil de mantener?
- ¿Cómo demostraría que mi solución es correcta?
- ¿Qué cambiaría si el sistema creciera?

---

# 2. Reglas de trabajo

## Regla 1 — No buscar la solución antes de pensar

Antes de investigar:

1. Lee el problema.
2. Escribe tu hipótesis.
3. Diseña una solución.
4. Explica por qué crees que funcionará.
5. Solo entonces investiga.

---

## Regla 2 — Primero correctness, después elegancia

Tu primera pregunta no debe ser:

> ¿Cómo hago esto bonito?

Debe ser:

> ¿Cómo hago imposible que el sistema quede en un estado incorrecto?

---

## Regla 3 — La base de datos también forma parte del diseño

No dependas únicamente de:

```python
if email_already_exists:
    raise Error()
```

Pregúntate también:

```text
¿Debería existir una UNIQUE constraint?
¿Debería ser NOT NULL?
¿Necesito una FOREIGN KEY?
¿Existe un CHECK constraint útil?
```

---

## Regla 4 — Cada proyecto debe poder romperse

Una implementación que funciona únicamente en el happy path no está terminada.

Debes intentar:

- datos inválidos;
- requests repetidos;
- recursos inexistentes;
- estados imposibles;
- consultas grandes;
- fallos de dependencias;
- combinaciones inesperadas.

---

## Regla 5 — Explica antes de refactorizar

Antes de cambiar una solución escribe:

```text
Qué está mal.
Por qué está mal.
Qué propiedad quiero mejorar.
Qué coste tendrá el cambio.
```

---

# 3. Plantilla para cada reto

Puedes crear una carpeta por reto:

```text
challenge-01/
├── README.md
├── DESIGN.md
├── src/
├── tests/
└── POSTMORTEM.md
```

## README.md

```md
# Problema

## Contexto

## Requisitos

## Invariantes

## Restricciones

## Criterios de aceptación
```

## DESIGN.md

```md
# Diseño

## Modelo de datos

## Endpoints

## Flujo principal

## Errores

## Decisiones

## Alternativas consideradas

## Trade-offs
```

## POSTMORTEM.md

```md
# Postmortem

## Qué esperaba que ocurriera

## Qué ocurrió realmente

## Qué rompió primero

## Qué entendí

## Qué cambiaría

## Qué preguntas me quedaron
```

---

# 4. Stack recomendado para Nivel 1

Puedes usar:

```text
Python
FastAPI
PostgreSQL
SQLAlchemy
Alembic
pytest
Docker
Git
```

Pero no es obligatorio.

El objetivo del nivel no es aprender FastAPI.

El objetivo es aprender a pensar como Backend Engineer.

---

# 5. Nivel 1 — 20 problemas

---

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

# Problema 02 — Actualización parcial de perfiles

## Contexto

Ahora los usuarios pueden modificar su perfil.

## Problema

Quieres implementar:

```http
PATCH /users/{id}
```

pero necesitas distinguir:

```json
{}
```

de:

```json
{"name": null}
```

y de:

```json
{"name": ""}
```

## Misión

Diseña correctamente una actualización parcial.

## Invariantes

El sistema nunca debe dejar un usuario con datos inválidos.

## Preguntas

- ¿Qué significa PATCH?
- ¿Cómo distingues campo ausente de campo nulo?
- ¿Permites borrar un valor?
- ¿Qué ocurre con propiedades desconocidas?
- ¿PUT y PATCH deberían comportarse igual?

## Conceptos

```text
PATCH
PUT
partial updates
nullable fields
validation
HTTP semantics
```

## Pruebas

- update de un solo campo;
- body vacío;
- campo desconocido;
- null explícito;
- recurso inexistente.

---

# Problema 03 — Estados válidos de una orden

## Contexto

Una tienda tiene órdenes.

Estados:

```text
pending
paid
shipped
cancelled
```

## Problema

Actualmente puedes hacer:

```text
cancelled → shipped
shipped → pending
```

## Misión

Evitar transiciones imposibles.

## Invariantes

Solo deben existir transiciones permitidas.

Ejemplo:

```text
pending → paid
pending → cancelled
paid → shipped
paid → cancelled
```

## Preguntas

- ¿El estado es solamente un string?
- ¿Dónde viven las reglas de transición?
- ¿Cómo evitas lógica duplicada?
- ¿Cómo registrarías el historial?

## Conceptos

```text
state machines
domain invariants
enums
business rules
```

## Senior Questions

- ¿Qué ocurre si aparecen más estados?
- ¿Las transiciones necesitan auditoría?
- ¿Debería una transición ser un endpoint específico?

---

# Problema 04 — Crear recursos relacionados

## Contexto

Una orden pertenece a un usuario.

## Problema

El cliente intenta crear una orden con:

```json
{"user_id": 999999}
```

pero ese usuario no existe.

## Misión

Garantizar integridad referencial.

## Invariantes

Toda orden debe pertenecer a un usuario válido.

## Preguntas

- ¿Compruebas el usuario desde Python?
- ¿Necesitas una FOREIGN KEY?
- ¿Qué ocurre si el usuario desaparece?
- ¿CASCADE o RESTRICT?

## Conceptos

```text
foreign keys
referential integrity
ON DELETE
constraints
```

---

# Problema 05 — Diseñar errores consistentes

## Contexto

Tu API responde errores de muchas formas:

```json
{"error":"bad"}
```

```json
{"detail":"Not found"}
```

```json
{"message":"Invalid request"}
```

## Misión

Diseñar un contrato consistente de errores.

## Requisitos

El cliente debe poder distinguir:

- validación;
- autenticación;
- recurso inexistente;
- conflicto;
- error inesperado.

## Preguntas

- ¿Qué debe entender una máquina?
- ¿Qué debe entender un humano?
- ¿Debes exponer excepciones internas?
- ¿Cómo manejas correlation IDs?

## Conceptos

```text
error contracts
HTTP status codes
error codes
exception handling
correlation IDs
```

---

# Problema 06 — Transferencia entre cuentas

## Contexto

Tienes cuentas con saldo.

```text
Account A = 100
Account B = 50
```

Quieres transferir 30.

## Problema

No puede ocurrir:

```text
A = 70
B = 50
```

si el segundo update falla.

## Misión

La transferencia debe ser una única operación lógica.

## Invariantes

```text
dinero total antes = dinero total después
saldo >= 0
```

## Preguntas

- ¿Qué operaciones deben ocurrir juntas?
- ¿Qué pasa si falla la segunda query?
- ¿Qué debe hacer rollback?

## Conceptos

```text
transactions
atomicity
rollback
ACID
```

## Senior Questions

- ¿Qué cambia si las cuentas están en diferentes bases de datos?
- ¿El dinero debería representarse como float?

---

# Problema 07 — Dinero sin errores de precisión

## Contexto

Tu sistema almacena:

```python
price = 0.1
```

## Problema

Las operaciones financieras empiezan a producir resultados inesperados.

## Misión

Modelar dinero correctamente.

## Preguntas

- ¿Por qué float es peligroso?
- ¿Decimal?
- ¿Centavos como integer?
- ¿Dónde se define la moneda?

## Conceptos

```text
floating point
Decimal
NUMERIC
money representation
currency
```

---

# Problema 08 — Evitar cantidades imposibles

## Contexto

Un cliente crea una orden con:

```json
{"quantity": -5}
```

## Misión

Hacer imposible almacenar cantidades inválidas.

## Invariantes

```text
quantity > 0
price >= 0
```

## Preguntas

- ¿Validación de API o CHECK constraint?
- ¿Por qué quizás necesitas ambas?
- ¿Qué pasa si otro proceso escribe directamente a la DB?

## Conceptos

```text
CHECK constraints
defense in depth
domain validation
```

---

# Problema 09 — Listados que no explotan

## Contexto

Tienes:

```http
GET /users
```

y la base tiene 300,000 usuarios.

## Problema

El endpoint devuelve todo.

## Misión

Diseñar paginación.

## Preguntas

- ¿limit/offset?
- ¿Cuál debería ser el límite máximo?
- ¿Cómo comunicas el total?
- ¿Qué ocurre si se insertan filas mientras navegas?

## Conceptos

```text
pagination
limit
offset
cursor pagination
API design
```

## Senior Questions

- ¿Qué problemas tiene OFFSET con grandes tablas?
- ¿Qué condiciones necesita cursor pagination?

---

# Problema 10 — Filtros y ordenamiento seguros

## Contexto

Quieres:

```http
GET /orders?status=paid&sort=created_at
```

## Problema

El cliente puede enviar cualquier campo como `sort`.

## Misión

Diseñar filtros y ordenamiento sin crear una API frágil o insegura.

## Conceptos

```text
query parameters
allowlists
dynamic queries
SQL injection
filter design
```

## Pruebas

- filtro válido;
- filtro inválido;
- sort válido;
- sort inexistente;
- combinaciones múltiples.

---

# Problema 11 — Buscar correctamente

## Contexto

El usuario quiere buscar por nombre:

```http
GET /users?q=ana
```

## Problema

No sabes si usar igualdad, LIKE o búsqueda más compleja.

## Misión

Definir semántica de búsqueda clara.

## Preguntas

- ¿case-sensitive?
- ¿substring?
- ¿prefix?
- ¿qué ocurre con espacios?
- ¿cuándo una búsqueda empieza a necesitar otro enfoque?

## Conceptos

```text
LIKE
ILIKE
text normalization
indexes
search semantics
```

---

# Problema 12 — El problema N+1

## Contexto

Listado:

```http
GET /orders
```

devuelve cada orden con su usuario.

Tu código ejecuta:

```text
1 query para orders
100 queries para users
```

## Misión

Detectar y corregir N+1.

## Preguntas

- ¿Cómo sabes cuántas queries ejecuta un endpoint?
- ¿JOIN?
- ¿eager loading?
- ¿cuándo un JOIN también puede ser problemático?

## Conceptos

```text
N+1 queries
joins
eager loading
lazy loading
query profiling
```

---

# Problema 13 — Crear recursos de forma predecible

## Contexto

Implementas:

```http
POST /orders
```

## Problema

El endpoint devuelve `200 OK`, aunque creó un recurso.

## Misión

Diseñar correctamente la respuesta.

## Preguntas

- ¿200 o 201?
- ¿debe devolver el objeto?
- ¿Location header?
- ¿qué ocurre si la creación es asíncrona?

## Conceptos

```text
HTTP semantics
201 Created
202 Accepted
Location header
```

---

# Problema 14 — Eliminar sin romper relaciones

## Contexto

Quieres borrar un usuario.

Pero tiene:

```text
orders
payments
audit records
```

## Problema

Un simple:

```sql
DELETE FROM users
```

puede destruir información importante.

## Misión

Definir política de eliminación.

## Preguntas

- ¿hard delete?
- ¿soft delete?
- ¿RESTRICT?
- ¿CASCADE?
- ¿qué datos deben conservarse?

## Conceptos

```text
soft delete
hard delete
referential integrity
data retention
```

## Senior Questions

- ¿Soft delete afecta consultas únicas?
- ¿Qué pasa con privacidad y derecho a eliminación?

---

# Problema 15 — Auditoría de cambios

## Contexto

Un administrador cambia:

```text
price: 100 → 10
```

y nadie sabe quién lo hizo.

## Misión

Crear una estrategia básica de auditoría.

## Registrar

```text
actor
action
resource
before
after
timestamp
```

## Preguntas

- ¿La auditoría forma parte de la misma transacción?
- ¿Qué información no deberías guardar?
- ¿Es mutable un audit log?

## Conceptos

```text
audit logging
immutability
actor identity
traceability
```

---

# Problema 16 — Importación CSV con datos sucios

## Contexto

El negocio entrega:

```text
users.csv
```

con 100,000 filas.

Algunas tienen:

- emails inválidos;
- fechas incorrectas;
- duplicados;
- columnas faltantes.

## Misión

Importar sin corromper la base de datos.

## Preguntas

- ¿fallar todo por una fila?
- ¿procesar parcialmente?
- ¿cómo reportar errores?
- ¿cómo manejar duplicados?

## Conceptos

```text
batch processing
validation
partial failures
error reports
bulk inserts
```

## Puente hacia Data Engineering

Este es uno de los primeros problemas que conecta Backend con procesamiento de datos.

---

# Problema 17 — Exportaciones grandes

## Contexto

Un usuario solicita:

```http
GET /reports/orders.csv
```

con 5 millones de registros.

## Problema

Construir todo el archivo en memoria puede matar el proceso.

## Misión

Diseñar una exportación razonable.

## Preguntas

- ¿streaming?
- ¿background job?
- ¿paginación interna?
- ¿cuándo devolver 202?

## Conceptos

```text
streaming responses
memory usage
background jobs
batching
```

---

# Problema 18 — Requests demasiado lentos

## Contexto

Un endpoint necesita:

```text
1. guardar order
2. generar PDF
3. enviar email
4. actualizar analytics
```

Tarda 12 segundos.

## Misión

Decidir qué pertenece al request path.

## Preguntas

- ¿Qué necesita el usuario antes de recibir respuesta?
- ¿Qué puede ocurrir después?
- ¿Qué tareas deberían ejecutarse fuera del request?

## Conceptos

```text
request lifecycle
synchronous vs asynchronous work
background jobs
latency budgets
```

---

# Problema 19 — Logs útiles

## Contexto

Producción devuelve:

```text
500 Internal Server Error
```

y tus logs dicen:

```text
ERROR: something failed
```

## Misión

Diseñar logging útil.

## Preguntas

- ¿Qué información necesitas para reconstruir un fallo?
- ¿Qué datos no deberías registrar?
- ¿request ID?
- ¿structured logging?

## Conceptos

```text
structured logging
log levels
request IDs
observability
PII
```

---

# Problema 20 — Mini sistema integrado

## Contexto

Construye una pequeña plataforma de órdenes que reúna lo aprendido.

Debe permitir:

```http
POST /users
POST /products
POST /orders
PATCH /orders/{id}
GET /orders
GET /orders/{id}
DELETE /users/{id}
```

## Requisitos

Usuarios:

```text
email único
nombre obligatorio
```

Productos:

```text
price >= 0
name obligatorio
```

Órdenes:

```text
deben pertenecer a un usuario
deben contener al menos un producto
quantity > 0
total calculado correctamente
estado válido
```

## Debe incluir

```text
validación
constraints
foreign keys
transactions
errores consistentes
paginación
filtros
logging
tests
audit básico
```

## Restricciones

```text
PostgreSQL obligatorio
sin Redis
sin Kafka
sin microservicios
```

La finalidad es comprobar cuánto puedes resolver correctamente con una arquitectura sencilla.

---

# 6. Evaluación del Nivel 1

No evalúes los retos solamente con:

```text
funciona / no funciona
```

Usa cuatro dimensiones.

## 1. Correctness

Preguntas:

- ¿Mantiene invariantes?
- ¿Acepta estados inválidos?
- ¿La base de datos protege reglas críticas?

---

## 2. Reasoning

Debes poder explicar:

```text
por qué elegiste esta solución
qué alternativas consideraste
qué responsabilidad tiene cada capa
```

---

## 3. Failure Awareness

Debes saber:

```text
qué puede fallar
cómo se manifiesta
cómo detectarlo
```

---

## 4. Trade-offs

Debes poder responder:

```text
qué ganas
qué pierdes
qué complejidad agregaste
```

---

# 7. Sistema de puntuación personal

No es para competir.

Es para detectar debilidades.

Por cada reto:

```text
0 — no entiendo el problema
1 — puedo implementarlo siguiendo ejemplos
2 — puedo implementarlo solo
3 — puedo explicar por qué funciona
4 — puedo identificar fallos y alternativas
5 — puedo defender el diseño y sus trade-offs
```

El objetivo no es obtener 5 inmediatamente.

El objetivo es mover problemas desde:

```text
1 → 2 → 3 → 4
```

---

# 8. Cómo usar los hints

Cada reto debería tener tres niveles de ayuda.

## Hint 1 — conceptual

Te ayuda a pensar.

Ejemplo:

```text
La aplicación no es la única capa capaz de proteger una invariante.
```

## Hint 2 — dirección

Te señala una familia de soluciones.

Ejemplo:

```text
Revisa constraints de PostgreSQL.
```

## Hint 3 — técnico

Te acerca a una implementación.

Ejemplo:

```text
Investiga UNIQUE, CHECK y FOREIGN KEY.
```

Regla:

> No leas Hint 2 sin haber intentado Hint 1.  
> No leas Hint 3 sin haber intentado Hint 2.

---

# 9. Rutina recomendada por reto

## Fase A — 15 minutos

Sin código.

Escribe:

```text
problema
invariantes
modelo
endpoint
errores
```

---

## Fase B — 45–90 minutos

Implementa una primera versión.

---

## Fase C — 20–40 minutos

Intenta romperla.

---

## Fase D — 20–40 minutos

Investiga solamente los conceptos que aparecieron.

---

## Fase E — 30 minutos

Refactoriza.

---

## Fase F — 10 minutos

Completa `POSTMORTEM.md`.

---

# 10. Criterio para avanzar al Nivel 2

No necesitas que los 20 proyectos sean perfectos.

Pero antes de avanzar deberías poder razonar con comodidad sobre:

```text
HTTP
validación
invariantes
constraints
foreign keys
transactions básicas
errores
modelado relacional
paginación
queries
testing
logging
```

Y deberías empezar a pensar automáticamente:

```text
¿Qué puede quedar inconsistente?

¿Qué regla puede garantizar la base de datos?

¿Qué estados son inválidos?

¿Qué debería ocurrir si el request falla a mitad?

¿Cómo sé que esto realmente funciona?
```

Cuando esas preguntas empiezan a aparecer naturalmente, el Nivel 1 está cumpliendo su función.

---

# 11. Próximo nivel

El Nivel 2 debe introducir problemas donde varias operaciones ocurren al mismo tiempo.

Temas:

```text
race conditions
concurrency
isolation levels
locking
optimistic concurrency
pessimistic concurrency
idempotency
duplicate requests
lost updates
retries
```

Ejemplos:

```text
dos personas compran el último producto
dos requests gastan el mismo saldo
dos usuarios reservan el mismo asiento
un cliente repite un pago
dos workers procesan el mismo mensaje
```

Aquí empieza el razonamiento serio sobre consistencia y concurrencia.

---

# Principio final

No estudies una herramienta porque aparece en una roadmap.

Estudia una herramienta cuando puedas responder:

```text
¿Qué problema concreto me resuelve?
¿Por qué mi solución actual no es suficiente?
¿Qué nueva complejidad introduce?
¿Qué propiedad del sistema mejora?
```

La meta no es construir muchos proyectos.

La meta es acumular muchos **modelos mentales transferibles**.

Ese es el fundamento del razonamiento de Backend Engineering.
