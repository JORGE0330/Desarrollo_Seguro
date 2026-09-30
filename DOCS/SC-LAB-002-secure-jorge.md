# SECURECAMPUS
## SC-LAB-002 · Mapa de Seguridad a lo largo del SDLC

**Curso:** Desarrollo Seguro  
**Práctica:** SC-LAB-002  
**Tema:** Mapa de Seguridad a lo largo del SDLC

---

## 5. Actividad guiada · Calificaciones

**Caso:** Un estudiante autenticado puede cambiar `/calificaciones/125` por `/calificaciones/126` y consultar calificaciones ajenas.

| Fase | ¿Qué debería hacerse? | Control / evidencia |
|---|---|---|
| **Requisitos** | Definir claramente que un estudiante únicamente puede consultar sus propias calificaciones. También se deben especificar los permisos de otros usuarios, por ejemplo, profesores y administradores. | Requisito de seguridad documentado: **“Un estudiante solo podrá consultar sus propias calificaciones y no podrá acceder a información de otros estudiantes.”** |
| **Diseño** | Diseñar el sistema para que, además de verificar que el usuario inició sesión, también compruebe que tiene autorización y posteriormente consulte la calificación solicitada. La validación debe realizarse en el servidor y no depender únicamente del número que aparece en la URL. | Matriz de roles y permisos, diagrama de control de acceso. Seguir la regla de autorización: validar usuario autenticado, verificar rol y permitir o denegar el acceso. |
| **Desarrollo** | Programar una validación que compare al estudiante autenticado con el propietario de las calificaciones. Aunque alguien cambie `/125` por `/126`, el servidor debe comprobar si esas calificaciones le pertenecen antes de mostrarlas. | Validación de autorización en el endpoint o controlador. Si el usuario no es propietario ni tiene un rol autorizado, se devuelve **403 Forbidden**. |
| **Pruebas** | Probar intencionalmente el cambio de identificadores. Por ejemplo, iniciar sesión como el alumno 125 e intentar consultar `/calificaciones/126`. El sistema debe impedir el acceso. También se deben probar diferentes usuarios y roles. | Realizar pruebas de autorización, en donde el alumno 125 trata de ingresar a `/calificaciones/126` y el sistema le indica **Acceso denegado**. Documentar el resultado de la prueba. |
| **Despliegue** | Antes de liberar el sistema, comprobar que las configuraciones de seguridad y los controles de autorización funcionen también en el ambiente de producción. Además, habilitar registros de los accesos realizados. | Checklist de despliegue y configuración segura. Verificar que las rutas protegidas requieren tanto autenticación como autorización. |
| **Operación / Mantenimiento** | Revisar periódicamente los registros para detectar intentos de acceso indebido. | Generar logs de accesos denegados, alertas y revisión constante de eventos de autorización. |

---

## 6. Reto por equipo

Analizar los cuatro escenarios. Para cada uno, proponer al menos un control temprano y un control posterior.

### Escenarios

| Escenario | Situación |
|---|---|
| **A · Documentos** | Un estudiante intenta descargar el documento de otro usuario modificando un identificador. |
| **B · Token** | Un desarrollador intenta incluir un token dentro de un commit. |
| **C · Profesor** | Un profesor intenta modificar calificaciones de un grupo no asignado. |
| **D · Login** | Una cuenta registra 100 intentos fallidos de autenticación en 10 minutos. |

### Mapa de controles por fase

| Esc. | Requisitos | Diseño | Desarrollo | Pruebas | Despliegue | Operación |
|---|---|---|---|---|---|---|
| **A** | Definir que cada estudiante solo pueda acceder a sus propios documentos. | Diseñar permisos que relacionen al usuario con sus documentos. | Validar que el documento pertenezca al usuario autenticado. | Cambiar el identificador y comprobar que el acceso sea rechazado. | Verificar que los controles de autorización funcionen en producción. | Registrar y alertar intentos de acceso a documentos ajenos. |
| **B** | Establecer que los tokens no deben guardarse en el código. | Diseñar el uso de variables de entorno o un gestor de secretos. | Implementar detección de tokens antes del commit. | Probar que un commit con un token sea detectado o bloqueado. | Activar el escaneo de secretos en el repositorio. | Revocar el token expuesto, generar uno nuevo y revisar su uso. |
| **C** | Definir que un profesor solo pueda modificar sus grupos asignados. | Diseñar permisos que relacionen profesor y grupo. | Validar que el profesor tenga asignado el grupo antes de modificar. | Intentar modificar un grupo no asignado y comprobar que sea rechazado. | Verificar que los roles y permisos funcionen correctamente. | Registrar los cambios de calificaciones en una bitácora. |
| **D** | Definir un límite de intentos fallidos de inicio de sesión. | Diseñar un bloqueo temporal después de varios intentos fallidos. | Implementar contador de intentos y bloqueo temporal. | Simular varios intentos incorrectos y comprobar que se active la protección. | Activar monitoreo y alertas de autenticación. | Detectar los 100 intentos fallidos, generar una alerta y proteger la cuenta. |

---

## 7. Clasificación conceptual

Elegir dos decisiones del mapa y explicar qué concepto representa mejor cada una.

| Decisión | Secure SDLC / By Design / By Default / Shift Left | Justificación |
|---|---|---|
| **1. Diseñar el sistema para verificar que cada estudiante solo pueda acceder a sus propios documentos.** | **Security By Design** | Porque la seguridad se considera desde el diseño del sistema, definiendo desde el inicio cómo se controlará el acceso a los documentos y no agregando la protección después de que aparezca el problema. |
| **2. Detectar tokens o secretos antes de realizar un commit.** | **Shift Left** | Porque la revisión de seguridad se realiza en una etapa temprana del desarrollo, antes de que el código llegue al repositorio o al despliegue, permitiendo detectar el problema lo antes posible. |

---

## 8. Reflexión

### ¿Qué riesgo de SC-LAB-001 necesitó controles en más fases?

El riesgo de **acceso no autorizado a calificaciones de otros grupos** necesitó controles en más fases, porque la autorización debe definirse desde requisitos y diseño, implementarse durante el desarrollo, comprobarse en pruebas y mantenerse mediante controles y monitoreo durante el despliegue y la operación.

### ¿Qué habría ocurrido si el equipo hubiera esperado hasta pruebas?

Se habría detectado demasiado tarde, provocando más retrabajo y cambios en el diseño y código ya desarrollados.

### ¿Qué control depende de una regla de negocio y cuál puede automatizarse?

El control de autorización de un profesor sobre sus grupos depende de una **regla de negocio**, porque se debe conocer qué grupos tiene asignados. La detección de tokens o secretos antes de un commit puede **automatizarse** mediante herramientas de análisis del código.

---

## INTEGRANTES:

- ANGELICA FUENTES GONZALEZ
- JORGE ULISES VILLAVICENCIO NUÑEZ 
- CARLOS MELO SAMANO
- IAN DANIEL MENDEZ VALENZUELA 





