# Tecnológico Nacional de México

## Instituto Tecnológico de Toluca

### Ingeniería en Sistemas Computacionales

**Unidad de Aprendizaje:** Desarrollo Seguro
**Asignación:** SC-LAB-001  
**Profesora:** Marelis Lara Carrillo  

**Estudiantes:**
- Jorge Ulises Villavicencio Nuñez
- Angelica Fuentes Gonzalez
- Carlos Daniel Melo Samano
- Ian Daniel Méndez Valenzuela

**Periodo Escolar:** Agosto - Diciembre 2026  
**Fecha de entrega:** 11/09/2026

---

# SECURECAMPUS

## SC-LAB-001

**Identificación inicial de activos, amenazas, vulnerabilidades, ataques, impactos, riesgos y controles**

| Campo | Información |
|---|---|
| **Proyecto** | SecureCampus |
| **Curso** | Desarrollo Seguro |
| **Versión** | 1.0 · Septiembre 2026 |

## 1. Propósito de la práctica

Analizar SecureCampus con mentalidad de seguridad para identificar qué debe protegerse, qué puede salir mal, qué debilidades podrían permitirlo y qué controles iniciales conviene proponer. El objetivo no es adivinar términos: es justificar el razonamiento.

## 2. Escenario SecureCampus

SecureCampus es un sistema web académico para estudiantes, profesores y administradores. Gestionará perfiles, calificaciones, documentos, solicitudes, usuarios, roles/permisos y registros de auditoría. El sistema debe ser funcional, pero también proteger información y operaciones sensibles.

- **Estudiante:** consulta su perfil, sus calificaciones, sus documentos y sus solicitudes.
- **Profesor:** consulta grupos asignados y captura calificaciones de esos grupos.
- **Administrador:** administra usuarios, roles, permisos y consulta logs; algunas operaciones académicas requieren autorización específica.

## 3. Conceptos de trabajo

| Concepto | Pregunta guía |
|---|---|
| **Activo** | ¿Qué información, servicio o capacidad tiene valor y debemos proteger? Recursos críticos tangibles o intangibles (ej. datos de alumnos, actas de notas, disponibilidad del portal). |
| **Amenaza** | ¿Qué situación o actor podría causar daño? Eventos, circunstancias o agentes maliciosos externos/internos con potencial de vulnerar el sistema. |
| **Vulnerabilidad** | ¿Qué debilidad podría facilitar el daño? Falla de diseño, bug de código, configuración errónea o carencia de control en la aplicación. |
| **Ataque** | ¿Qué acción concreta podría explotar la debilidad? Ejecución activa de una técnica o exploit para materializar una amenaza contra la vulnerabilidad. |
| **Impacto** | ¿Qué consecuencia tendría si ocurre? Daño sufrido en confidencialidad, integridad, disponibilidad, reputación o legalidad. |
| **Riesgo** | ¿Qué combinación de posibilidad e impacto debemos gestionar? Probabilidad de que una amenaza explote una debilidad ponderada con la severidad de su impacto. |
| **Control** | ¿Qué medida previene, detecta o corrige el problema? Salvaguarda técnica, administrativa o arquitectónica implementada para mitigar el riesgo. |

## 4. Actividad guiada: consulta de perfiles

**Caso:** María inicia sesión con el perfil 125. Al observar la URL, cambia manualmente `/perfil/125` por `/perfil/126`. El sistema devuelve información de otro estudiante.

| Elemento | Respuesta del equipo | Justificación |
|---|---|---|
| **Activo** | Información personal y académica de estudiantes | Se trata de un recurso que debe protegerse debido a que contiene datos sensibles como nombre, matrícula, calificaciones u otra información privada. |
| **Amenaza** | Acceso no autorizado a información de otros estudiantes | Un usuario puede intentar consultar información a la que no está autorizado. |
| **Vulnerabilidad** | Falta de validación de autorización sobre el ID del perfil (IDOR) | El sistema proporciona información a través de la URL con el número 126 sin validar si María tiene permiso para consultar dicho perfil. |
| **Ataque** | Manipulación del parámetro de la URL | María cambia `/perfil/125` por `/perfil/126` para acceder al perfil de otro estudiante. |
| **Impacto** | Divulgación no autorizada de información y violación de la privacidad | Un estudiante puede visualizar datos de otros usuarios, lo que ocasiona un incumplimiento de privacidad. |
| **Riesgo** | Alto | Existe una posibilidad de acceso a información ajena y el ataque es fácil de realizar, sin hacer uso de herramientas complejas. |
| **Control** | Implementar control de acceso/autorización en el servidor para cada perfil solicitado | El servidor debe verificar que el usuario autenticado tenga permiso para consultar el id solicitado, no se debe confiar únicamente en el ID de la URL. |

## 5. Reto por equipo

Analicen al menos cuatro escenarios diferentes de SecureCampus. Deben incluir obligatoriamente: **calificaciones, documentos y autenticación**; el cuarto escenario lo elige el equipo (**roles/permisos, solicitudes, logs, perfiles u otro módulo aprobado**).

| Escenario | Activo | Amenaza | Vulnerabilidad | Ataque | Impacto | Control |
|---|---|---|---|---|---|---|
| **1. Calificaciones** | Calificaciones de alumnos | Acceso no autorizado a calificaciones de otros grupos | Falta de validación de permisos para acceder a las calificaciones de un grupo | Un docente modifica el ID del grupo en la URL o en una solicitud para consultar las calificaciones de otro grupo | Exposición o modificación no autorizada de calificaciones y pérdida de confidencialidad e integridad | El sistema debe verificar que el docente tenga autorización para acceder al grupo solicitado |
| **2. Documentos** | Documentos académicos de los estudiantes | Acceso no autorizado a documentos de otros alumnos | Falta de control de acceso sobre los documentos solicitados | Un usuario cambia el identificador del documento en la URL o solicitud para descargar un documento ajeno | Exposición de documentos privados y posible fuga de información personal | Validar en el servidor que el usuario tenga permiso para consultar o descargar cada documento |
| **3. Autenticación** | Cuentas y credenciales de los usuarios | Suplantación de identidad o acceso a cuentas ajenas | El sistema permite registrar contraseñas débiles o fáciles de adivinar | Un atacante intenta adivinar o realizar ataques de fuerza bruta sobre contraseñas débiles | Acceso no autorizado a la cuenta y a la información del usuario | Durante el registro, exigir una contraseña segura con una longitud mínima y combinación de diferentes tipos de caracteres, además de limitar intentos de inicio de sesión |
| **4. Elección del equipo** | Cuentas de usuarios y funciones del portal | Acceso de usuarios inactivos o uso de funciones no autorizadas | El sistema no verifica el estado de la cuenta ni los permisos del usuario | Un usuario con cuenta desactivada intenta iniciar sesión o acceder directamente a una sección restringida | Acceso no autorizado a información o funciones del sistema | Verificar que la cuenta esté activa y validar el rol y permisos del usuario antes de permitir el acceso a cada módulo |

## 6. Preguntas de reflexión

### 1. ¿Una amenaza y una vulnerabilidad son lo mismo? Explica con un ejemplo de SecureCampus.

No, una amenaza es una posible situación que puede afectar a un sistema, mientras que una vulnerabilidad es una falla que puede ser aprovechada por cualquier amenaza. En el caso de SecureCampus, una amenaza podría ser el acceso no autorizado de docentes a grupos con los que no estén relacionados a través de una URL alterada, y como tal la vulnerabilidad es que ese apartado no valide antes si el docente logueado cuenta con los permisos para acceder a dicha información.

### 2. ¿Puede existir una vulnerabilidad aunque todavía nadie la haya explotado?

Si, puede que exista una vulnerabilidad sin ser identificada que posteriormente pueda ser aprovechada por una amenaza.

### 3. ¿Un usuario autenticado está automáticamente autorizado para cualquier recurso?

No, la autenticación valida que el usuario tiene acceso al sistema y la autorización implica determinar que procedimientos puede ejecutar y a qué recursos puede acceder.

### 4. ¿Qué control de los propuestos debería definirse desde requisitos o diseño? ¿Por qué?

Corregir fallas de autorización como IDOR o bypass funcional a posteriori suele requerir refactorizar la arquitectura de rutas, middleware y el diseño de la base de datos. Si se define desde la fase de requisitos y diseño (enfoque *Secure by Design*), cada endpoint y consulta nace desacoplada de IDs secuenciales y con verificaciones de privilegios estandarizadas.

### 5. ¿Qué activo consideran más crítico y por qué?

Mientras que una caída de disponibilidad causa interrupciones temporales y una fuga de perfiles afecta la privacidad, la alteración silenciosa de calificaciones vulnera la validez legal y la confianza de los títulos expedidos por la institución educativa, teniendo un impacto legal y de reputación permanente.
