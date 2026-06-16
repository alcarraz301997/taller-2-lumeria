# Constitution

## Art. 3 · Quality Standards

### 3.1 Trazabilidad

Toda solicitud de generación de preguntas NQ debe registrar su estado durante el ciclo de vida del proceso (pendiente, en ejecución, completado o fallido).

### 3.2 Integridad de resultados

El sistema no debe exponer resultados parciales al usuario. Una generación será considerada válida únicamente cuando finalice correctamente.

### 3.3 Gestión de errores

Los errores producidos durante la generación deben quedar registrados con información suficiente para permitir su análisis y corrección.

---

## Art. 4 · Architecture Principles

### 4.1 Procesamiento asíncrono

La generación de preguntas NQ debe ejecutarse en segundo plano y no bloquear la interacción del usuario con el sistema.

### 4.2 Separación de responsabilidades

La lógica de negocio relacionada con la generación de preguntas debe mantenerse desacoplada de las interfaces de usuario y de los mecanismos de ejecución.

### 4.3 Escalabilidad

La solución debe permitir procesar múltiples solicitudes de generación sin afectar significativamente la disponibilidad del sistema.

---

## Art. 7 · Boundaries

### ALWAYS DO

* Registrar el inicio y fin de cada ejecución.
* Validar los datos requeridos antes de iniciar el procesamiento.
* Notificar el resultado final de la generación.

### ASK FIRST

* Cambios en las reglas académicas utilizadas para generar preguntas.
* Modificaciones en los criterios de validación de resultados.
* Cambios que impacten procesos existentes de generación manual.

### NEVER DO

* Ejecutar procesos pesados de generación dentro de solicitudes síncronas de usuario.
* Publicar preguntas generadas parcialmente o con errores conocidos.
* Omitir el registro de errores críticos durante la ejecución.
