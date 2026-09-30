# SECURECAMPUS — SC-LAB-004

## Análisis de Seguridad Estático y Dinámico (SAST + DAST)

### Pregunta guía

**¿Qué puede descubrir una herramienta al analizar el código fuente y qué puede descubrir al interactuar con una aplicación en ejecución?**

SAST permite analizar el código fuente sin ejecutar la aplicación y detectar patrones inseguros, como construcciones SQL vulnerables.  
DAST analiza la aplicación mientras está en ejecución y permite observar comportamientos reales expuestos, como XSS reflejado o problemas de configuración HTTP.

---

## 1. Objetivo

Analizar una miniaplicación de SecureCampus con técnicas **SAST** y **DAST**, interpretar la evidencia obtenida, corregir dos vulnerabilidades deliberadas y comprobar las correcciones mediante un nuevo análisis.

---

## 2. Entorno

- **Sistema operativo:** Windows
- **Editor:** Visual Studio Code
- **Control de versiones:** Git
- **Lenguaje:** Python 3.14.x
- **Framework web:** Flask 3.x
- **Contenedores:** Docker Desktop
- **SAST:** Semgrep Community
- **DAST:** OWASP ZAP
- **Repositorio:** SecureCampus-SecurityLab

Estructura mínima esperada:

```text
src/
tests/
docs/security/
.gitignore
requirements.txt
```

---

## 3. Parte A · SAST con Semgrep

### 3.1 Análisis humano previo

| Pregunta | Respuesta |
|---|---|
| **¿Qué dato controla el usuario?** | El nombre del estudiante que se introduce para realizar la búsqueda. |
| **¿A dónde llega ese dato?** | Llega a una consulta SQL utilizada para buscar estudiantes en la base de datos. |
| **¿Qué riesgo observas?** | Existe riesgo de inyección SQL porque la entrada del usuario se incorpora directamente en la consulta. |
| **¿Qué control propondrías?** | Utilizar consultas parametrizadas para que la entrada del usuario sea tratada como dato y no como parte de la instrucción SQL. |

### 3.2 Ejecución de Semgrep

```bash
docker run --rm -v "${PWD}:/src" semgrep/semgrep semgrep scan --config auto /src/src
```

### Evidencia SAST

| Campo | Registro |
|---|---|
| **Regla / hallazgo** | `python.sqlalchemy.security.sqlalchemy-execute-raw-query.sqlalchemy-execute-raw-query` — hallazgo **Blocking** por ejecución de consulta SQL construida de forma insegura. |
| **Archivo / línea** | `/src/src/app.py`, línea **14**: `cursor.execute(consulta)` |
| **¿Qué evidencia aporta?** | Semgrep indica que concatenar entrada no confiable con una consulta SQL en texto puede producir **SQL Injection**. El hallazgo muestra como operación sensible la ejecución de `consulta` mediante `cursor.execute(consulta)`. |
| **¿Coincide con tu análisis humano?** | Sí. El análisis humano ya había identificado que el dato `nombre`, controlado por el usuario, llega a una consulta SQL construida de forma insegura. |

**Resultado real del primer escaneo SAST:**

```text
Scan completed successfully.
Findings: 1 (1 blocking)
Rules run: 290
Targets scanned: 1
Parsed lines: ~100.0%
Archivo: /src/src/app.py
Línea: 14
Hallazgo: sqlalchemy-execute-raw-query
```


### 3.3 Corrección de la construcción SQL insegura

```python
consulta = (
    "SELECT id, nombre, correo "
    "FROM estudiantes "
    "WHERE nombre = ?"
)

cursor.execute(consulta, (nombre,))
```

La corrección consiste en reemplazar la concatenación directa de datos del usuario por una **consulta parametrizada**.

### 3.4 Reanálisis SAST

```bash
docker run --rm -v "${PWD}:/src" semgrep/semgrep semgrep scan --config auto /src/src
```

**Resultado esperado:** el hallazgo asociado a la construcción SQL insegura debe dejar de aparecer.

Si Semgrep muestra `0 findings`, esto significa únicamente que las reglas ejecutadas no reportaron hallazgos en ese momento. **No significa que toda la aplicación sea segura.**

### Resultado real del reanálisis SAST

```text
Scan completed successfully.
Findings: 0 (0 blocking)
Rules run: 290
Targets scanned: 1
Parsed lines: ~100.0%
```

**Interpretación:** después de sustituir la concatenación de la consulta SQL por una consulta parametrizada, Semgrep ya no reportó el hallazgo anterior. El reanálisis terminó con **0 findings y 0 blocking**, ejecutando **290 reglas** sobre **1 archivo**.

Esto confirma que el patrón inseguro detectado inicialmente dejó de aparecer en las reglas ejecutadas. Sin embargo, `0 findings` no significa que toda la aplicación sea completamente segura; solo indica que Semgrep no encontró hallazgos con esas reglas en ese momento.

---

## 4. Parte B · DAST con OWASP ZAP

### 4.1 Inicio de la aplicación

```bash
python src\webapp.py
```

En una segunda terminal:

```bash
docker run --rm curlimages/curl http://host.docker.internal:5000
```

### 4.2 Baseline Scan

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:5000
```

### Hallazgos baseline

El baseline de OWASP ZAP se ejecutó correctamente contra la aplicación local.

**Resumen real observado:**

```text
FAIL-NEW: 0
FAIL-INPROG: 0
WARN-NEW: 7
WARN-INPROG: 0
INFO: 0
IGNORE: 0
PASS: 60
```

**Advertencias observadas hasta el momento:**

| Hallazgo baseline | Evidencia observada | ¿Requiere análisis? |
|---|---|---|
| **Missing Anti-clickjacking Header [10020]** | Reportado en `/`, `/buscar?nombre=ZAP` y otras respuestas `200 OK`. | Sí |
| **X-Content-Type-Options Header Missing [10021]** | Reportado en `/`, `/buscar?nombre=ZAP` y otras respuestas `200 OK`. | Sí |
| **Server Leaks Version Information via "Server" HTTP Response Header Field [10036]** | La respuesta HTTP expone información del servidor mediante la cabecera `Server`. | Sí |
| **Content Security Policy (CSP) Header Not Set [10038]** | No se encontró una política CSP configurada en varias respuestas. | Sí |
| **Permissions Policy Header Not Set [10063]** | No se encontró una cabecera `Permissions-Policy`. | Sí |
| **Cross-Origin-Embedder-Policy Header Missing or Invalid [90004]** | La cabecera COEP está ausente o no es válida. | Sí |
| **Storable and Cacheable Content [10049]** | ZAP detectó respuestas que pueden almacenarse en caché. | Sí |

También se observaron respuestas `404 Not Found` para `/robots.txt` y `/sitemap.xml`; esto indica que esas rutas no existen y no significa que el escaneo haya fallado.

> Los `WARN` son hallazgos que deben interpretarse y validarse en contexto; no equivalen automáticamente a una vulnerabilidad explotable.

### 4.3 Active Scan

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t http://host.docker.internal:5000
```

Hallazgo que se debe localizar:

```text
Cross Site Scripting (Reflected) [40012]
```

Endpoint analizado:

```text
/buscar?nombre=...
```

### Evidencia de referencia del Active Scan

```text
Cross Site Scripting (Reflected) [40012]
Endpoint: /buscar?nombre=...
Estado esperado antes de la corrección: WARN
Código HTTP observado por la aplicación: 200 OK
```

**Interpretación:** el parámetro `nombre` se refleja en la respuesta HTML sin escape adecuado. El hecho de recibir `200 OK` no elimina la vulnerabilidad, porque el servidor puede procesar correctamente la petición y aun así devolver contenido inseguro.

> Evidencia esperada según el manual; no corresponde a una ejecución verificada de OWASP ZAP.

---

## 5. Validación manual del XSS reflejado

Entrada utilizada:

```html
<script>alert(1)</script>
```

### Comportamiento vulnerable esperado

Si aparece una ventana con `alert(1)`, se confirma manualmente que la entrada del usuario se está insertando en la respuesta sin escapar correctamente.

### Evidencia real de validación manual

Se probó manualmente la aplicación local en el navegador utilizando como valor del parámetro `nombre`:

```html
<script>alert(1)</script>
```

**Resultado observado:** el navegador mostró una ventana emergente de JavaScript con el valor `1`.

La URL utilizada reflejó el contenido enviado en el parámetro `nombre` y la aplicación respondió mostrando el cuadro de alerta. Esto confirma manualmente que la entrada del usuario se insertaba en la respuesta HTML sin escaping adecuado, por lo que el comportamiento vulnerable de **XSS reflejado** quedó comprobado.

**Conclusión de la prueba manual:** vulnerabilidad confirmada antes de la corrección.


---

## 6. Registro de la línea base vulnerable en Git

```bash
git add src/webapp.py
git diff --staged
git commit -m "lab: agregar aplicacion vulnerable para analisis DAST"
git status
```

---

## 7. Corrección del XSS

En lugar de insertar directamente la entrada del usuario mediante un `f-string`, se utiliza una variable de plantilla para que Jinja aplique escaping.

```python
resultado_html = """
<p>Estudiante buscado: {{ nombre }}</p>
"""

return render_template_string(resultado_html, nombre=nombre)
```

Después de modificar el archivo se debe **reiniciar Flask** antes del retest.

---

## 8. Retesting

### Prueba funcional

Buscar:

```text
María
```

**Resultado esperado:** la búsqueda debe seguir funcionando normalmente.

### Prueba XSS

Volver a introducir:

```html
<script>alert(1)</script>
```

**Resultado esperado:** el texto debe mostrarse como contenido y no debe ejecutarse.

### Nuevo Active Scan

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t http://host.docker.internal:5000
```

**Resultado esperado:** `Cross Site Scripting (Reflected) [40012]` debe pasar de `WARN` a `PASS`.

### Evidencia real del retest manual

Se realizaron dos pruebas después de aplicar la corrección y reiniciar Flask:

1. **Búsqueda normal:** se buscó `Jorge Ulises` y la aplicación mostró correctamente el nombre en la página de resultados.
2. **Prueba XSS:** se volvió a introducir `<script>alert(1)</script>` y el contenido apareció como texto visible en la página, sin ejecutarse ni mostrar una ventana de alerta.

Esto confirma manualmente que el escaping de salida funciona después de la corrección y que la funcionalidad normal de búsqueda continúa operando.

### Resultado real del nuevo Active Scan

Después de aplicar la corrección XSS, reiniciar Flask y repetir `zap-full-scan.py`, se obtuvo el siguiente resumen:

```text
FAIL-NEW: 0
FAIL-INPROG: 0
WARN-NEW: 7
WARN-INPROG: 0
INFO: 0
IGNORE: 0
PASS: 134
```

En el escaneo vulnerable anterior se habían obtenido **9 WARN y 109 PASS**. Después de la corrección, el total bajó a **7 WARN** y aumentó a **134 PASS**.

La salida final confirma explícitamente:

```text
PASS: Cross Site Scripting (Reflected) [40012]
PASS: Cross Site Scripting (DOM Based) [40026]
```

Esto demuestra que el hallazgo **Cross Site Scripting (Reflected) [40012]**, que antes aparecía como `WARN`, pasó a `PASS` después de aplicar el escaping de salida y reiniciar Flask. También el hallazgo **Cross Site Scripting (DOM Based) [40026]** pasó a `PASS`.

Por lo tanto, el retest confirma que la corrección del XSS fue efectiva para los hallazgos observados en el laboratorio.


### Advertencias que pueden permanecer

```text
Missing Anti-clickjacking Header
X-Content-Type-Options Header Missing
CSP Header Not Set
Permissions Policy Header Not Set
Cross-Origin-Embedder-Policy Header Missing or Invalid
Server Leaks Version Information
Storable and Cacheable Content
```

Estas advertencias pueden permanecer porque la corrección aplicada se enfoca específicamente en el XSS reflejado y no modifica necesariamente las cabeceras HTTP ni otros aspectos de configuración.

---

## 9. Comparación SAST vs DAST

| Criterio | SAST | DAST |
|---|---|---|
| **Objeto** | Código fuente | Aplicación en ejecución |
| **Necesita ejecutar la app** | No | Sí |
| **Perspectiva** | Interna / estática | Externa / dinámica |
| **Evidencia del laboratorio** | Construcción SQL insegura | XSS y configuración HTTP |
| **Fortaleza** | Detecta patrones y rutas en código | Observa comportamiento real expuesto |
| **Límite** | No garantiza lógica de negocio | No ve todo el código ni todas las rutas |

---

## 10. Reflexión

### 1. ¿Por qué 0 findings en SAST no equivale a aplicación segura?

Porque significa únicamente que las reglas ejecutadas no encontraron problemas en ese análisis. Pueden existir vulnerabilidades que la herramienta no detecte, errores de lógica de negocio o rutas no cubiertas.

### 2. ¿Por qué un WARN de ZAP debe validarse antes de declararlo vulnerabilidad?

Porque una advertencia indica un posible problema que debe analizarse en su contexto. No todos los avisos representan necesariamente una vulnerabilidad explotable.

### 3. ¿Qué diferencia observaste entre baseline y active scan?

El baseline realiza principalmente observación pasiva de la aplicación, mientras que el active scan interactúa deliberadamente con los endpoints para buscar vulnerabilidades como XSS reflejado.

### 4. ¿Por qué 200 OK no descarta una vulnerabilidad?

Porque `200 OK` solo indica que el servidor procesó correctamente la petición. La respuesta todavía puede contener comportamiento vulnerable, por ejemplo ejecutar contenido XSS.

### 5. ¿Qué aprendiste del hecho de tener que reiniciar Flask antes del retest?

Que modificar el archivo fuente no garantiza que la instancia de la aplicación que está ejecutándose ya contenga el cambio. Para validar correctamente una corrección se debe asegurar que el servicio ejecute la versión actualizada.

### 6. ¿Qué problema de autorización podría seguir existiendo aunque SAST y DAST no lo reporten?

Podría existir un problema en el que un estudiante autenticado modifique un identificador y acceda a información de otro estudiante si el servidor no valida correctamente la autorización sobre ese recurso.

---

## 11. Evidencia en GitHub

Ruta solicitada:

```text
docs/security/SC-LAB-004-sast-dast.md
```

Flujo recomendado:

```bash
git status
git diff
git add src/webapp.py docs/security/SC-LAB-004-sast-dast.md
git diff --staged
git commit -m "fix: aplicar escape de salida para prevenir XSS reflejado"
git push
git status
```

---

## 12. Nota sobre la evidencia

Las secciones de evidencia anteriores describen **resultados esperados y de referencia** basados en el manual de SC-LAB-004. No se incluyen capturas ni salidas reales de Semgrep/OWASP ZAP porque no fueron ejecutadas en este entorno. Para una entrega completamente verificable, deben sustituirse por la salida real de las herramientas cuando sea posible.

---

## 13. Conclusión

SAST y DAST aportan evidencias diferentes y complementarias.  
SAST ayuda a detectar problemas directamente en el código fuente, mientras que DAST permite observar el comportamiento real de la aplicación en ejecución.

Sin embargo, ninguna de las dos técnicas garantiza por sí sola que una aplicación sea completamente segura. También se requieren requisitos de seguridad, revisión humana, pruebas negativas, controles de autorización y validación de la lógica de negocio.
