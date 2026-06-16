# Constitution

> Principios no negociables del proyecto Odiseo Backend.
> Esta constitución prevalece sobre cualquier convención local, preferencia personal o decisión técnica no documentada.
> Para modificarla se requiere consenso del equipo de arquitectura.

---

## Art. 1 · Propósito y alcance

### 1.1 Propósito

Esta constitución define los principios, estándares y límites que rigen el desarrollo del backend de Odiseo. Su objetivo es garantizar consistencia, mantenibilidad y calidad del código a lo largo del tiempo.

### 1.2 Alcance

Aplica a todo el código dentro del repositorio, tanto en `app/` (legacy) como en `src/` (nueva arquitectura). El código legacy puede tener excepciones explícitas documentadas, pero toda incorporación nueva debe cumplirla.

### 1.3 Prioridad

En caso de conflicto entre esta constitución y otras guías (AGENTS.md, ADRs, README), esta constitución tiene la mayor jerarquía. Los ADRs detal lan decisiones de arquitectura específicas; esta constitución define los límites infranqueables.

### 1.4 Incumplimiento

El incumplimiento de cualquiera de estos artículos debe ser detectado en code review y corregido antes de mergear. No se permite mergear código que viole la constitución.

---

## Art. 2 · Lenguaje, código y estilo

### 2.1 Lenguaje del código

El código fuente (identificadores, clases, métodos, variables, comentarios) se escribe en **inglés**. Las excepciones son:
- Documentación técnica (`docs/`, ADRs, esta constitución) — **español**.
- Mensajes para el usuario final, según requerimiento del producto.
- Archivos de configuración y migraciones heredados.

### 2.2 Tipado estricto

Todo archivo PHP nuevo debe declarar `declare(strict_types=1)`. No se permite el uso de `mixed` ni `array` sin docblock que especifique la forma de los datos.

### 2.3 Formateo

El código debe pasar `./vendor/bin/pint` sin errores. No se permite desviación de las reglas de Pint. No se desactivan reglas por archivo.

### 2.4 Análisis estático

PHPStan nivel 5 en ambos namespaces (`App\` y `Src\`). No se permiten `ignoreErrors` genéricos. Las excepciones deben justificarse con un comentario inline del tipo `/** @phpstan-ignore-next-line <razón> */`. Los DTOs son la única excepción documentada para propiedades no definidas.

### 2.5 Nombramiento

- Clases: `PascalCase`.
- Métodos y funciones: `camelCase`.
- Propiedades y variables: `camelCase`.
- Constantes de clase: `UPPER_SNAKE_CASE`.
- Archivos: `PascalCase` para clases, `snake_case` para configuraciones y migraciones.
- Directorios: `PascalCase` para módulos, `kebab-case` para configuraciones.

Nombra por **intención**, no por implementación. Mal: `UserManager`, `DataProcessor`. Bien: `RegisterUserUseCase`, `GenerateMaterialService`.

---

## Art. 3 · Quality Standards

### 3.1 Cobertura y estrategia de tests

#### 3.1.1 Cobertura mínima

Todo módulo nuevo debe tener una suite de tests que cubra:

| Capa | Cobertura mínima | Tipo de test |
|------|------------------|-------------|
| Use Cases (Application) | 100% de ramas | Unitario |
| Domain (entidades, value objects) | 100% de ramas | Unitario |
| Controllers/Handlers | Escenarios feliz + errores representativos | Integración |
| Repositories | Contrato completo (CRUD + consultas) | Integración |

No se exige cobertura numérica global como métrica de puerta. La métrica real es: **todo código nuevo debe tener tests que prueben su comportamiento, no su implementación**.

#### 3.1.2 Pirámide de tests

Se sigue la pirámide clásica:

- **Unitarios** (80%+): lógica de dominio y aplicación sin dependencias externas. Sin base de datos, sin framework, sin HTTP.
- **Integración** (15%): interacción con base de datos, repositorios reales, jobs, servicios externos reales o con contract test.
- **E2E / HTTP** (5%): rutas completas, autenticación, serialización. Se usa `apiCallWithCookie()` y helpers de `tests/Pest.php`.

#### 3.1.3 Aislamiento

Los tests deben aislarse entre sí:
- Usar `DatabaseTransactions` en tests de integración.
- No compartir estado estático entre tests.
- No depender de IDs hardcodeados. Crear los datos necesarios en cada test.
- Usar factories en lugar de seeds compartidos.

#### 3.1.4 Estructura de tests

Para el código en `src/`, los tests se organizan dentro del módulo correspondiente:

```
src/App/Modules/V2/<Module>/
  └── Tests/
      ├── Unit/
      ├── Integration/
      └── Feature/
```

Para el código legacy en `app/`, los tests van en `tests/Context/<Contexto>/`.

### 3.2 Trazabilidad

Todo proceso asíncrono (jobs, colas, generación de material, preguntas NQ) debe registrar su estado durante todo el ciclo de vida. Estados obligatorios:

| Estado | Significado |
|--------|-------------|
| `pending` | Creado, esperando ejecución |
| `processing` / `in_progress` | En ejecución activa |
| `completed` | Finalizado exitosamente |
| `failed` | Finalizado con error |

No se permite omitir estados intermedios ni transitar directamente de `pending` a `completed`.

### 3.3 Integridad de resultados

El sistema no debe exponer resultados parciales al usuario. Una operación se considera válida únicamente cuando finaliza correctamente y todos sus efectos laterales se han persistido.

En operaciones que involucran múltiples pasos (ej. generar material → convertir a PDF → subir a GCS), si cualquiera falla, todo el proceso se marca como `failed`. No se permiten resultados huérfanos.

### 3.4 Gestión de errores

#### 3.4.1 Registro obligatorio

Todo error producido durante la ejecución debe quedar registrado con:
- Mensaje descriptivo.
- Excepción original (trace completo).
- Identificador del proceso o solicitud asociada.
- Momento exacto (timestamp).
- Contexto adicional relevante (usuario, curso, módulo, etc.).

#### 3.4.2 Clasificación

Los errores se clasifican según ADR-0007:

- **DomainException** (409): violación de regla de negocio.
- **NotFoundException** (404): recurso inexistente.
- **UnauthorizedException** (401): no autenticado.
- **ForbiddenException** (403): sin permisos.
- **ApplicationException** (400): error genérico de aplicación.
- **Exception** (500): error no controlado.

No se permite capturar excepciones genéricas (`catch (\Exception $e)`) sin relanzar o tratar explícitamente.

### 3.5 Linting y formato como bloqueante

- `./vendor/bin/pint` debe pasar en todo el código modificado.
- `./vendor/bin/phpstan` nivel 5 debe pasar en todo el código modificado.
- No se mergea código que no pase ambas herramientas, sin excepción.

### 3.6 Presupuesto de rendimiento

- Las respuestas HTTP síncronas no deben exceder 5 segundos (p95).
- Los jobs en cola no deben exceder el timeout definido en Horizon (300s para `generate_material_class`).
- Las consultas a base de datos no deben exceder 500ms en el percentil 99.
- Toda consulta N+1 es considerada bug, no deuda técnica.

---

## Art. 4 · Architecture Principles

### 4.1 Arquitectura hexagonal (Puertos y Adaptadores)

#### 4.1.1 Estructura de capas

Todo módulo en `src/App/Modules/V2/<Module>/` debe seguir tres capas:

| Capa | Responsabilidad | Depende de |
|------|----------------|------------|
| **Domain** | Reglas de negocio, entidades, value objects, eventos de dominio, repositorios (interfaces) | Nada externo |
| **Application** | Use Cases, DTOs de entrada/salida, puertos de entrada | Domain |
| **Infrastructure** | Implementaciones concretas (controladores HTTP, repositorios Eloquent, jobs, listeners, servicios externos) | Application, Domain |

#### 4.1.2 Regla de dependencia (Dependency Rule)

Las dependencias apuntan hacia adentro:
- **Infrastructure** → **Application** → **Domain**.
- Domain no sabe nada de Application ni de Infrastructure.
- Application no sabe nada de Infrastructure.

Ninguna capa interna puede importar nada de una capa externa. Esto se verifica con PHPStan y en code review.

#### 4.1.3 Comunicación entre capas

- Application se comunica con Infrastructure mediante **interfaces** definidas en Domain (puertos de salida).
- Infrastructure se comunica con Application inyectando Use Cases en los controladores.
- No se permite que Infrastructure implemente lógica de negocio.

### 4.2 Procesamiento asíncrono

La generación de material, preguntas NQ y cualquier proceso pesado debe ejecutarse en segundo plano mediante colas (Redis + Horizon). No debe bloquear la interacción del usuario con el sistema.

Los jobs deben:
- Recibir un DTO, no modelos Eloquent.
- Registrar su estado en el ciclo de vida.
- Notificar el resultado (completado o fallido).

Excepción: procesos cuya ejecución sea inferior a 200ms y no tengan impacto en la experiencia del usuario pueden ejecutarse sincrónicamente.

### 4.3 Separación de responsabilidades

#### 4.3.1 Responsabilidad única

Cada clase debe tener una única razón para cambiar:
- Los **Controllers** solo manejan HTTP (request → respuesta).
- Los **Use Cases** orquestan lógica de negocio, sin saber de HTTP.
- Los **Models** (Eloquent) solo representan persistencia; no contienen lógica de negocio.
- Los **Repositories** abstraen el almacenamiento; no contienen reglas de negocio.
- Los **Jobs** encolan trabajo; no contienen lógica de dominio.

#### 4.3.2 Use Cases (ADR-0002)

- Clase `final`.
- Un único método público `execute`.
- Recibe un DTO de entrada.
- Retorna un DTO de salida o tipo primitivo solo en casos explícitos y simples.
- No recibe Requests, Models ni arrays sin tipar.

### 4.4 Escalabilidad

La solución debe permitir procesar múltiples solicitudes concurrentes sin degradación significativa. Esto implica:
- Colas separadas por tipo de proceso (`generate_material_class`, `generate_nq_questions`, etc.).
- Timeouts configurados por cola.
- Evitar estados compartidos entre procesos.
- Idempotencia en los jobs: ejecutar un job dos veces con la misma entrada debe producir el mismo resultado.

### 4.5 Anti-patrones prohibidos

| Anti-patrón | Por qué está prohibido |
|-------------|----------------------|
| **God Class** | Clases con más de 300 líneas o que mezclan responsabilidades |
| **Service Locator** | Llamar a `app()->make()` o `resolve()` fuera de bootstrap |
| **Static Call** | Métodos estáticos que dependen de estado global (BD, auth, etc.) |
| **Active Record en Domain** | Pasar modelos Eloquent a capas internas |
| **Array como contrato** | Pasar y retornar arrays sin forma definida |
| **Herencia de Use Cases** | Extender Use Cases en lugar de componerlos |
| **Side effects en constructores** | Llamar a BD, APIs o servicios en un constructor |

### 4.6 Patrones permitidos

| Patrón | Dónde usarlo |
|--------|-------------|
| **DTO** | Entrada/salida de Use Cases, comunicación entre capas |
| **Repository** | Abstracción de persistencia contra Eloquent |
| **Mapper** | Conversión entre entidades de dominio y modelos de persistencia |
| **Strategy** | Algoritmos intercambiables (generación de preguntas por tipo) |
| **Observer / Eventos de dominio** | Reacciones a cambios en el dominio |
| **Factory** | Creación de objetos complejos con validación |
| **Value Object** | Datos inmutables con comportamiento (Email, RUC, Currency) |

### 4.7 Transacciones

Las transacciones de base de datos se manejan en la capa de **Application** (Use Case), no en Infrastructure. Un Use Case puede agrupar múltiples operaciones de repositorio en una sola transacción.

### 4.8 Feature Flags (config/features.php)

Todo comportamiento nuevo que afecte a usuarios existentes debe protegerse detrás de un feature flag. Los flags se definen en `config/features.php` y su valor se controla por entorno.

---

## Art. 5 · Datos y persistencia

### 5.1 Conexiones a base de datos

- **Escritura**: conexión `pgsql_master`.
- **Lectura**: conexión `pgsql_stand_by`.
- Ambas apuntan al mismo schema `odiseo`.

Las consultas de solo lectura deben usar la conexión `stand_by`. Las escrituras deben usar `master`. No se permite mezclar conexiones en una misma transacción.

### 5.2 Migraciones

- Todas las migraciones se definen en `database/migrations/`.
- Se nombran con prefijo timestamp (`YYYY_MM_DD_HHmmss_create_<tabla>_table.php`).
- Toda migración debe ser **reversible** (tener `down()`).
- No se permite modificar migraciones ya aplicadas en producción. Se crea una nueva migración.

### 5.3 Modelos

- Los modelos Eloquent viven en `app/Domain/Models/` (legacy) o en la capa de Infrastructure de cada módulo.
- No deben contener lógica de negocio.
- No deben lanzar excepciones de dominio.
- Deben definir `$table`, `$fillable` o `$guarded`, y `$casts`.

### 5.4 Repositorios

- La interfaz del repositorio se define en Domain.
- La implementación concreta vive en Infrastructure.
- El repositorio retorna entidades de dominio, no modelos Eloquent.
- Un mapper convierte entre el modelo de persistencia y la entidad de dominio.

### 5.5 CQRS en persistencia (ADR-0008)

Se separan operaciones de lectura y escritura a nivel de repositorio:
- **Lectura**: repositorios específicos para consultas (read models, proyecciones).
- **Escritura**: repositorios para comandos (CRUD).

Esto no implica separación de base de datos. La separación es de interfaz.

### 5.6 Validación de dominio con acceso a persistencia (ADR-0009)

Los validadores de dominio pueden tener acceso a persistencia solo a través de interfaces definidas en Domain. No se permite inyectar repositorios de Infrastructure directamente en validadores de dominio.

---

## Art. 6 · Seguridad

### 6.1 Autenticación

- Guard por defecto: `api` (driver `jwt` via `tymon/jwt-auth`).
- Token JWT enviado vía cookie con prefijo `{APP_PREFIX_COOKIE}_access_token`.
- El JWT_SECRET debe ser único por entorno y rotarse periódicamente.
- No se permite almacenar tokens JWT en localStorage del lado del cliente.

### 6.2 Autorización

- Se usa el sistema de policies y gates de Laravel.
- Para módulos en `src/`, los checks de autorización se hacen en Application (Use Cases), no en los handlers HTTP.
- La autenticación (¿quién eres?) se resuelve en el middleware. La autorización (¿puedes hacer esto?) se resuelve en el Use Case.

### 6.3 Internas (ODISEO_KEY)

Las comunicaciones entre servicios internos se autentican mediante `ODISEO_KEY`. Esta clave no debe exponerse en logs, respuestas HTTP ni mensajes de error.

### 6.4 Secrets

- Ninguna clave secreta se hardcodea en el código.
- Las credenciales de Google Cloud se cargan desde `google-cloud-credentials.json` o variable de entorno.
- El archivo de credenciales GCS está en `.gitignore`.
- Sentry DSN se configura por entorno, nunca se expone al cliente.

### 6.5 Validación de entrada

- Toda entrada HTTP debe validarse mediante FormRequests de Laravel.
- No se permite confiar en datos no validados del cliente.
- Las reglas de validación deben ser explícitas y específicas (evitar `sometimes` y `nullable` sin necesidad demostrada).

---

## Art. 7 · Boundaries

### ALWAYS DO

**Arquitectura y diseño**
- Declarar `strict_types=1` en todo archivo PHP nuevo.
- Definir una interfaz para cada repositorio en la capa de Domain.
- Usar DTOs para entrada y salida de Use Cases.
- Validar toda entrada HTTP con FormRequests.
- Registrar el inicio, progreso y fin de cada proceso asíncrono (jobs, generación, colas).

**Testing**
- Escribir tests de integración que verifiquen el ciclo feliz y los casos de error representativos.
- Usar `DatabaseTransactions` en tests de integración.
- Verificar estados intermedios de trazabilidad (`pending`, `processing`, `completed`, `failed`).

**Errores**
- Clasificar excepciones como DomainException, NotFoundException, UnauthorizedException, ForbiddenException o ApplicationException según ADR-0007.
- Registrar errores críticos con trace, contexto y timestamp.

**Operaciones**
- Notificar al usuario el resultado final de una generación asíncrona (completado o fallido).
- Usar la conexión `master` para escrituras y `stand_by` para lecturas.
- Pasar `./vendor/bin/pint` y `./vendor/bin/phpstan` antes de abrir un MR.

### ASK FIRST

**Cambios arquitectónicos**
- Modificar la estructura de capas de un módulo hexagonal.
- Introducir un nuevo patrón de diseño no contemplado en Art. 4.6.
- Cambiar la conexión de base de datos por defecto o agregar una nueva.
- Introducir una nueva dependencia externa (paquete, servicio, SDK).

**Comportamiento existente**
- Cambiar reglas académicas o de negocio utilizadas para generar preguntas o material.
- Modificar criterios de validación de resultados de generación.
- Alterar el mapeo de excepciones a códigos HTTP.
- Cambiar la estrategia de colas (timeouts, workers, prioridad).

**Operaciones**
- Introducir un proceso síncrono que pueda superar los 200ms.
- Eliminar o modificar un feature flag existente.
- Cambiar el nombre de una cola o la configuración de Horizon.
- Modificar el sistema de archivos (de GCS a local o viceversa) en producción.

**Datos**
- Agregar un índice sin analizar el impacto en consultas existentes.
- Modificar una migración ya aplicada en producción (se debe crear una nueva, pero si hay necesidad de rollback o alteración, consultar).
- Cambiar el esquema de la base de datos compartida (`odiseo`) sin coordinación.

### NEVER DO

**Arquitectura y código**
- Pasar modelos Eloquent a la capa de Domain o Application.
- Instanciar repositorios directamente con `new` en un Use Case (usar inyección de dependencias).
- Usar `app()->make()`, `resolve()` o Service Locator fuera del bootstrap.
- Definir métodos estáticos con dependencias de estado global (BD, auth, HTTP).
- Ejecutar procesos pesados de generación dentro de solicitudes síncronas de usuario.
- Usar `mixed` o `array` sin tipado como tipo de retorno o parámetro en código nuevo.
- Extender un Use Case (herencia) en lugar de componerlo.

**Datos**
- Ejecutar consultas N+1 sin resolverlas.
- Exponer resultados parciales o inconsistentes al usuario.
- Publicar preguntas, materiales o contenido generado parcialmente o con errores conocidos.
- Hardcodear IDs de base de datos en el código.
- Compartir el mismo modelo Eloquent entre módulos diferentes como contrato.

**Seguridad**
- Exponer JWT_SECRET, ODISEO_KEY, credenciales GCS o Sentry DSN en logs, responses o errores.
- Confiar en datos del cliente sin validar.
- Pasar datos no sanitizados directamente a FTS5, consultas raw o APIs externas.
- Almacenar tokens JWT en localStorage del cliente.

**Operaciones**
- Omitir el registro de errores críticos durante la ejecución.
- Mergear código que no pase `pint` y `phpstan`.
- Ignorar errores de PHPStan sin justificación documentada.
- Desplegar cambios sin feature flag cuando afectan a usuarios existentes.
- Modificar migraciones ya aplicadas en lugar de crear una nueva.

---

## Vigencia

Esta constitución entra en vigor inmediatamente y aplica a todo el código en el repositorio. Cualquier excepción debe ser documentada y aprobada por el equipo de arquitectura.
