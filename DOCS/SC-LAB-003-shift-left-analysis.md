# SECURECAMPUS — SC-LAB-003

## Costo de Corrección y Shift Left

### Pregunta guía

**¿Qué cambia cuando un problema de seguridad se descubre en Requisitos, Desarrollo, Pruebas o Producción?**

- **En Requisitos:** El impacto es mínimo porque el problema solo existe en papel o en la documentación; corregirlo únicamente implica ajustar la redacción de la historia de usuario o las reglas del sistema sin tocar código.
- **En Desarrollo:** El impacto es moderado, ya que el programador se da cuenta a tiempo y solo debe reescribir la lógica de la función o módulo en el que está trabajando antes de integrarlo.
- **En Pruebas:** El costo y el retraso aumentan de forma considerable; obliga a rechazar la entrega, reabrir tareas, regresar el flujo a desarrollo para refactorizar y volver a desplegar para repetir todas las pruebas de regresión.
- **En Producción:** Es el peor escenario con el costo más alto e impredecible; el sistema ya está expuesto a ataques reales, lo que exige desplegar parches de emergencia (*hotfixes*), revisar si hubo datos o cuentas comprometidas, arriesgar caídas del servicio y asumir daños a la reputación de la organización.

## 1. Objetivo

Analizar el retrabajo y el impacto de detectar problemas de seguridad tarde, y proponer actividades **Shift Left** sin confundirlas con “hacer toda la seguridad al inicio”.

## 2. Modelo conceptual

Mientras más tarde se descubre un problema, normalmente más artefactos, decisiones, pruebas, despliegues y personas pueden verse afectados. No se usarán multiplicadores universales de costo.

### Shift Left

Mover determinadas actividades de seguridad hacia etapas más tempranas y mantener seguridad durante todo el ciclo.

---

## 3. Caso guiado · Recuperación de contraseña

**RF-010:** “SecureCampus deberá permitir al usuario recuperar su contraseña”.

El enlace generado dura **7 días** y puede reutilizarse varias veces.

| Pregunta | Respuesta del equipo |
|---|---|
| **¿Dónde se originó principalmente la omisión?** | En Requisitos, ya que se define la recuperación de contraseña sin delimitar criterios de seguridad en relación a la vigencia, el uso único o la invalidación del enlace. |
| **¿Dónde podría descubrirse?** | Durante cualquier etapa del SDLC o incluso cuando el sistema ha sido implementado. Podría ser identificado específicamente en Pruebas, al detectarlo mediante una prueba que utiliza el enlace después de haber sido utilizado. |
| **¿Qué artefactos habría que cambiar si se descubre en pruebas?** | Revisar o incluso modificar los requisitos de aceptación, diseño del flujo de recuperación, implementación de backend, generación y validación de tokens, pruebas autorizadas y documentación. Podrían requerirse ajustes adicionales si el cambio afecta principalmente al despliegue o a datos existentes. |
| **¿Qué requisitos/criterios de seguridad faltaron?** | El enlace debe tener una vigencia definida, ser utilizado una única vez, invalidarse después de su uso, hacer uso de un token impredecible y seguro, y no permitir que un enlace comprometido pueda reutilizarse indefinidamente, definiendo criterios de aceptación para estos comportamientos. |
| **¿Qué moverían a la izquierda?** | Definir los criterios de seguridad de recuperación en Requisitos y Diseño, así como las reglas de expiración e invalidación, el diseño seguro del token y los casos de prueba negativos. Después se mantendrían controles durante Desarrollo, Pruebas y Operación. |

---

## 4. Reto integral · Tres situaciones

### Situaciones

- **A · Administrador:** El requisito permite consultar calificaciones sin definir condiciones. Al final se aclara que consultar y modificar requieren permisos distintos.
- **B · Upload:** Se aceptan archivos de usuarios autenticados sin definir tipo, tamaño, nombre ni almacenamiento seguro.
- **C · Dependencia:** Una biblioteca sin vulnerabilidades conocidas al incorporarse publica una vulnerabilidad crítica 8 meses después. Nadie la detecta por 2 meses.

### Análisis

| Caso | Origen | Descubrimiento | Retrabajo / impacto | Actividad Shift Left | Control posterior |
|---|---|---|---|---|---|
| **A · Administrador** | Requisito ambiguo sin condiciones de acceso definidas. | En fases tardías, como pruebas o auditorías. | Modificar la lógica de autorización y re-ejecutar pruebas. | Modelado de amenazas y revisión temprana de requisitos de seguridad. | Pruebas de penetración (PenTest) o análisis dinámico (DAST). |
| **B · Upload** | Falta de especificación de políticas de validación y almacenamiento seguro en la fase de diseño. | Mediante análisis estático de código (SAST), pruebas dinámicas (DAST) o tras la explotación de la vulnerabilidad en producción. | Alto impacto de seguridad, con riesgo de ejecución de código o denegación de servicio. Requiere rediseñar el módulo para validar tipos, tamaños, nombres y asegurar el almacenamiento. | Análisis estático preventivo y definición temprana de directrices seguras para el manejo de archivos. | Pruebas de penetración enfocadas en ataques de carga de archivos no validados (*Unrestricted File Upload*). |
| **C · Dependencia** | Uso de una biblioteca de terceros que introduce una vulnerabilidad crítica meses después de su integración inicial. | Demorado: tarda 10 meses en total desde la vulnerabilidad hasta su detección. Se descubre mediante escaneos de vulnerabilidades o reportes externos (Zero-Day o divulgación pública). | Alto riesgo de exposición continua en producción durante el tiempo que pasó desapercibida; requiere actualizar la biblioteca, verificar compatibilidad y desplegar un parche urgente. | Implementación temprana de análisis de composición de software (SCA - Software Composition Analysis) en el pipeline de CI/CD. | Monitoreo continuo de dependencias y escaneos periódicos de vulnerabilidades en producción. |

---

## 5. Escalera de costo cualitativa

Para el caso de recuperación de contraseña:

| Momento | ¿Qué habría que corregir/revisar? | Costo / retrabajo |
|---|---|---|
| **Requisitos** | Ajustar la redacción para indicar que el link dure poco tiempo y que solo sirva una vez. | **Bajo**, debido a que todavía no se ha desarrollado dicha funcionalidad y solo se modifican especificaciones y criterios. |
| **Diseño** | Modificar los diagramas de flujo y agregar en la base de datos los campos para la fecha de expiración y el estado del token. | **Bajo**, se están planeando las pantallas y la base de datos, todavía no hay código hecho. |
| **Desarrollo** | Cambiar la lógica para generar tokens de un solo uso y agregar las validaciones de tiempo en el backend. | **Medio**, ya que existe una implementación y deben realizarse cambios y pruebas. |
| **Pruebas** | Corregir las vulnerabilidades detectadas, actualizando la implementación y repitiendo pruebas funcionales y de seguridad. | **Medio-Alto**, porque el cambio puede provocar regresiones y obligar a repetir pruebas. |
| **Producción** | Corregir el sistema desplegado, analizando archivos afectados, realizando un nuevo despliegue y evaluando posibles incidentes. | **Muy alto**, ya que el sistema ya está al aire, corregirlo de emergencia cuesta más tiempo, interrumpe el servicio y pone en riesgo las cuentas de los usuarios. |

---

## 6. Pregunta con truco conceptual

### Dependencias

**¿Puede Shift Left ayudar con una vulnerabilidad que todavía no existía públicamente cuando desarrollamos? Explique qué sí puede prepararse desde antes.**

Sí, puede ayudar a prepararse y mitigar el impacto, incluso si la vulnerabilidad no existía públicamente en el momento del desarrollo.

Aunque es imposible prever fallos futuros en código de terceros, la filosofía **Shift Left** busca integrar la seguridad y la resiliencia desde el inicio para que, cuando la vulnerabilidad se haga pública meses después, el tiempo de reacción sea mínimo.

---

## 7. Reflexión

### ¿Shift Left elimina la necesidad de seguridad en operación?

No, debido a que **Shift Left distribuye la seguridad a lo largo del SDLC**, pero no elimina la necesidad de seguridad durante la operación.

### ¿Por qué una funcionalidad puede cumplir su requisito funcional y seguir siendo insegura?

Porque un requisito funcional se delimita a describir las funciones que debe realizar el sistema, pero no define las condiciones de seguridad que debe seguir.

### ¿Qué decisión de su equipo habría sido más barata de corregir antes?

Definir desde Requisitos la matriz de permisos del administrador, estableciendo que consultar calificaciones requiere permiso de consulta y modificar calificaciones requiere permiso de modificación.

---

## Evidencia en GitHub

Ruta solicitada:

```text
docs/security/SC-LAB-003-shift-left-analysis.md
```

Flujo obligatorio:

```bash
git status
git diff
git add
git diff --staged
git commit
git push
```
