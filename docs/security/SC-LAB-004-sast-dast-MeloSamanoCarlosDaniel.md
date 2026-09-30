# SC-LAB-004 — ANÁLISIS DE SEGURIDAD ESTÁTICO Y DINÁMICO

## Datos generales

| Dato | Información |
|---|---|
| **Carrera** | Ingeniería en Sistemas Computacionales |
| **Asignatura** | Desarrollo Seguro |
| **Docente** | Carrillo Lara Marelis |
| **Alumno** | Carlos Daniel Melo Sámano |
| **Matrícula** | 25280359 |
| **Grupo** | 252102 |
| **Período** | Agosto – Diciembre 2026 |
| **Nombre de la práctica** | Análisis de Seguridad Estático y Dinámico (SAST + DAST) |

---

## 1. Objetivo de la práctica

Analizar una aplicación de SecureCampus utilizando técnicas de análisis
estático (SAST) y análisis dinámico (DAST), con el propósito de identificar
vulnerabilidades en el código fuente y durante la ejecución de la aplicación.

Durante la práctica se utilizaron **Semgrep** para el análisis estático y
**OWASP ZAP** para el análisis dinámico. Posteriormente se aplicaron
correcciones a las vulnerabilidades identificadas y se realizaron nuevas
pruebas para comprobar los cambios.

---

## 2. Preparación del entorno

Para desarrollar la práctica se utilizó el siguiente entorno:

- Windows
- PowerShell
- Visual Studio Code
- Python
- Flask
- Git
- Docker Desktop
- Semgrep Community
- OWASP ZAP

Se trabajó dentro del proyecto:

```text
C:\Users\charl\SecureCampus
```

Se creó y activó el entorno virtual de Python:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

También se comprobó el funcionamiento de Git y Docker:

```powershell
git status
docker info
```

---

# 3. Análisis SAST con Semgrep

## 3.1 Análisis manual del código

Antes de ejecutar una herramienta de análisis se revisó manualmente
`src/app.py`.

Se identificó que el programa solicita al usuario el nombre de un estudiante
y utiliza este valor para construir una consulta SQL.

### Análisis previo

| Pregunta | Respuesta |
|---|---|
| **¿Qué dato controla el usuario?** | El nombre del estudiante ingresado mediante `input()`. |
| **¿A dónde llega ese dato?** | Llega a `buscar_estudiante(nombre)` y posteriormente se incorpora a una consulta SQL. |
| **¿Qué riesgo observas?** | Existe riesgo de SQL Injection porque la entrada proporcionada por el usuario se concatena directamente en la sentencia SQL. |
| **¿Qué control propondrías?** | Utilizar una consulta parametrizada para separar los datos proporcionados por el usuario de la instrucción SQL. |

La construcción identificada como insegura fue:

```python
consulta = (
    "SELECT id, nombre, correo "
    "FROM estudiantes "
    "WHERE nombre = '" + nombre + "'"
)

cursor.execute(consulta)
```

---

## 3.2 Ejecución de Semgrep

Para realizar el análisis SAST se ejecutó:

```powershell
docker run --rm -v "${PWD}:/src" semgrep/semgrep semgrep scan --config auto /src/src
```

Semgrep realizó el análisis correctamente y reportó un hallazgo:

```text
1 Code Finding

python.sqlalchemy.security.sqlalchemy-execute-raw-query.sqlalchemy-execute-raw-query

Blocking

23┆ cursor.execute(consulta)
```

El resumen obtenido fue:

```text
Scan completed successfully.
Findings: 1 (1 blocking)
Rules run: 290
Targets scanned: 2
Parsed lines: ~100.0%
```

### Evidencia SAST

| Campo | Resultado |
|---|---|
| **Regla / hallazgo** | `python.sqlalchemy.security.sqlalchemy-execute-raw-query.sqlalchemy-execute-raw-query` |
| **Tipo de problema** | Posible SQL Injection |
| **Archivo / línea** | `src/app.py`, línea 23 |
| **Operación señalada** | `cursor.execute(consulta)` |
| **¿Qué evidencia aporta?** | Semgrep detectó que una entrada no confiable participa en la construcción de una consulta SQL mediante concatenación y posteriormente se ejecuta con `cursor.execute(consulta)`. |
| **¿Coincide con el análisis humano?** | Sí. Durante la revisión manual ya se había identificado que `nombre`, controlado por el usuario, llegaba directamente a la construcción de la consulta SQL. |

---

## 3.3 Corrección de SQL Injection

Para solucionar el problema se eliminó la concatenación y se utilizó una
consulta parametrizada:

```python
consulta = (
    "SELECT id, nombre, correo "
    "FROM estudiantes "
    "WHERE nombre = ?"
)

cursor.execute(consulta, (nombre,))
```

De esta manera, el valor proporcionado por el usuario se procesa como un
parámetro de la consulta y no como parte de la instrucción SQL.

---

## 3.4 Reanálisis con Semgrep

Después de guardar la corrección se volvió a ejecutar:

```powershell
docker run --rm -v "${PWD}:/src" semgrep/semgrep semgrep scan --config auto /src/src
```

El hallazgo relacionado con la construcción insegura dejó de aparecer.

Un resultado sin hallazgos no garantiza que toda la aplicación sea segura.
Solamente indica que las reglas ejecutadas por Semgrep no detectaron otros
problemas en los archivos analizados en ese momento.

---

# 4. Análisis DAST con OWASP ZAP

## 4.1 Ejecución de la aplicación Flask

Para realizar las pruebas dinámicas se inició la aplicación:

```powershell
python src\webapp.py
```

La aplicación quedó disponible localmente mediante Flask.

Para permitir que los contenedores Docker utilizados durante el laboratorio
pudieran comunicarse con Flask, se configuró temporalmente:

```python
app.run(host="0.0.0.0", port=5000, debug=False)
```

Posteriormente se verificó la comunicación desde Docker:

```powershell
docker run --rm curlimages/curl http://host.docker.internal:5000
```

El contenedor obtuvo correctamente el contenido HTML de SecureCampus,
confirmando la comunicación entre Docker y la aplicación.

---

## 4.2 Baseline Scan

Se realizó primero un análisis baseline con OWASP ZAP:

```powershell
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:5000
```

El resultado fue:

```text
FAIL-NEW: 0
FAIL-INPROG: 0
WARN-NEW: 7
WARN-INPROG: 0
INFO: 0
IGNORE: 0
PASS: 60
```

### Principales advertencias

| Hallazgo | Observación | ¿Requiere análisis? |
|---|---|---|
| **Missing Anti-clickjacking Header [10020]** | Falta una cabecera HTTP para protección contra clickjacking. | Sí |
| **X-Content-Type-Options Header Missing [10021]** | La respuesta no contiene la cabecera `X-Content-Type-Options`. | Sí |
| **Server Leaks Version Information [10036]** | La cabecera `Server` expone información acerca del servidor. | Sí |
| **Content Security Policy Header Not Set [10038]** | No existe una política CSP configurada. | Sí |
| **Storable and Cacheable Content [10049]** | ZAP identificó contenido que puede almacenarse en caché. | Sí |
| **Permissions Policy Header Not Set [10063]** | No se encontró una política de permisos configurada. | Sí |
| **Cross-Origin-Embedder-Policy Header Missing or Invalid [90004]** | La cabecera correspondiente se encuentra ausente o no es válida. | Sí |

Los `WARN` no se consideraron automáticamente vulnerabilidades confirmadas,
ya que deben analizarse tomando en cuenta el contexto y funcionamiento de la
aplicación.

---

# 5. Active Scan

Posteriormente se realizó un análisis activo:

```powershell
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t http://host.docker.internal:5000
```

El análisis terminó con:

```text
FAIL-NEW: 0
WARN-NEW: 9
WARN-INPROG: 0
INFO: 0
IGNORE: 0
PASS: 132
```

Entre los resultados se identificó el hallazgo esperado:

```text
WARN-NEW: Cross Site Scripting (Reflected) [40012] x 1
```

El problema fue localizado en:

```text
/buscar?nombre=...
```

ZAP obtuvo una respuesta:

```text
200 OK
```

Esto demuestra que recibir un código HTTP `200 OK` únicamente indica que la
petición fue procesada correctamente por el servidor. No significa que el
contenido generado por la aplicación sea necesariamente seguro.

También se reportó:

```text
Cross Site Scripting (DOM Based) [40026]
```

---

# 6. Validación manual del XSS reflejado

Para comprobar manualmente el hallazgo reportado por OWASP ZAP se introdujo
en el campo correspondiente al nombre:

```html
<script>alert(1)</script>
```

Al realizar la búsqueda, el navegador ejecutó el código JavaScript y mostró
una ventana emergente con:

```text
1
```

Por lo tanto, se confirmó manualmente el comportamiento vulnerable de
**Cross Site Scripting (XSS) reflejado**.

La causa se encontraba en la inserción directa del parámetro `nombre` dentro
del HTML:

```python
nombre = request.args.get("nombre", "")

return f"""
...
<p>Estudiante buscado: {nombre}</p>
...
"""
```

La entrada controlada por el usuario era interpretada por el navegador como
parte del documento HTML.

---

# 7. Registro de la versión vulnerable

Antes de aplicar la corrección se registró en Git la versión vulnerable:

```powershell
git add src/webapp.py
git diff --staged
git commit -m "lab: agregar aplicacion vulnerable para analisis DAST"
git status
```

Esto permite conservar evidencia de la versión de la aplicación sobre la
cual se identificó el XSS.

---

# 8. Corrección del XSS reflejado

Para evitar insertar directamente la entrada del usuario mediante un
`f-string`, se modificó el endpoint para utilizar una variable de plantilla:

```python
@app.route("/buscar")
def buscar():
    nombre = request.args.get("nombre", "")

    resultado_html = """
    <html>
    <head>
        <title>Resultado - SecureCampus</title>
    </head>
    <body>
        <h1>Resultado de búsqueda</h1>
        <p>Estudiante buscado: {{ nombre }}</p>
        <a href="/">Regresar</a>
    </body>
    </html>
    """

    return render_template_string(resultado_html, nombre=nombre)
```

De esta manera, Jinja puede aplicar escaping al contenido de `nombre` antes
de incorporarlo al documento HTML.

Después de modificar `webapp.py`, se reinició Flask para garantizar que las
pruebas posteriores se realizaran sobre la nueva versión.

---

# 9. Retesting

## 9.1 Prueba funcional

Se realizó una búsqueda normal utilizando:

```text
María
```

**Resultado esperado:** la aplicación debe continuar mostrando correctamente
el nombre buscado.

## 9.2 Nueva prueba de XSS

Se volvió a introducir:

```html
<script>alert(1)</script>
```

**Resultado esperado:** el navegador debe mostrar el contenido como texto y
no debe ejecutar JavaScript ni abrir la ventana `alert(1)`.

## 9.3 Nuevo Active Scan

Después de validar manualmente la corrección se debe ejecutar nuevamente:

```powershell
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t http://host.docker.internal:5000
```

El objetivo del retest es comprobar que:

```text
Cross Site Scripting (Reflected) [40012]
```

deje de presentarse como `WARN` y aparezca como `PASS`.

> **Nota:** completar esta sección con el resultado real del último Active
> Scan después de realizar el retest.

---

# 10. Comparación entre SAST y DAST

| Criterio | SAST | DAST |
|---|---|---|
| **Objeto analizado** | Código fuente | Aplicación en ejecución |
| **¿Necesita ejecutar la aplicación?** | No | Sí |
| **Perspectiva** | Interna / estática | Externa / dinámica |
| **Evidencia obtenida** | Construcción SQL insegura | XSS y configuración HTTP |
| **Fortaleza** | Permite encontrar patrones y flujos inseguros directamente en el código. | Permite observar el comportamiento real de la aplicación expuesta. |
| **Limitación** | No garantiza encontrar problemas de lógica de negocio. | No necesariamente conoce ni analiza todas las rutas internas del código. |

---

# 11. Reflexión

### 1. ¿Por qué 0 findings en SAST no equivale a aplicación segura?

Porque solamente indica que las reglas utilizadas durante ese análisis no
detectaron otros hallazgos. Pueden existir vulnerabilidades que no estén
cubiertas por las reglas, errores de lógica de negocio o problemas que
requieran otras técnicas de análisis.

### 2. ¿Por qué un WARN de ZAP debe validarse antes de declararlo vulnerabilidad?

Porque un `WARN` representa una condición que requiere análisis. Dependiendo
del contexto de la aplicación puede representar una vulnerabilidad real, una
configuración mejorable o un hallazgo que no sea explotable.

### 3. ¿Qué diferencia observaste entre baseline y active scan?

El baseline permitió observar pasivamente las respuestas de la aplicación y
detectó principalmente problemas relacionados con cabeceras y configuración
HTTP. El active scan interactuó deliberadamente con los parámetros de la
aplicación y permitió detectar el XSS reflejado.

### 4. ¿Por qué 200 OK no descarta una vulnerabilidad?

Porque `200 OK` solamente informa que el servidor procesó satisfactoriamente
la solicitud HTTP. La respuesta generada todavía puede contener datos
inseguros o permitir comportamientos vulnerables, como ocurrió con el XSS
reflejado.

### 5. ¿Qué aprendiste del hecho de tener que reiniciar Flask antes del retest?

Que modificar el código fuente no es suficiente para asegurar que la
instancia actualmente en ejecución esté utilizando la nueva versión. Antes
de comprobar una corrección es necesario asegurarse de que el servicio esté
ejecutando el código actualizado.

### 6. ¿Qué problema de autorización podría seguir existiendo aunque SAST y DAST no lo reporten?

Podría existir un problema de control de acceso en el que un usuario
autenticado pudiera modificar un identificador o parámetro para consultar
información perteneciente a otro usuario si el servidor no verifica
correctamente que tenga autorización sobre ese recurso.

---

# 12. Evidencia final en Git y GitHub

Una vez finalizado el retesting, la documentación se almacenará en:

```text
docs/security/SC-LAB-004-sast-dast.md
```

Para registrar la corrección final:

```powershell
git status
git diff
git add src/webapp.py docs/security/SC-LAB-004-sast-dast.md
git diff --staged
git commit -m "fix: aplicar escape de salida para prevenir XSS reflejado"
git push
git status
```

---

# 13. Conclusión

Durante la práctica se comprobó que SAST y DAST proporcionan perspectivas
diferentes y complementarias sobre la seguridad de una aplicación.

Mediante Semgrep fue posible identificar en el código fuente una construcción
SQL insegura y posteriormente eliminar el hallazgo mediante una consulta
parametrizada.

Por otra parte, OWASP ZAP permitió analizar la aplicación durante su
ejecución. El Baseline Scan identificó diferentes aspectos de configuración
HTTP, mientras que el Active Scan detectó un XSS reflejado que posteriormente
fue confirmado mediante una prueba manual.

La práctica demuestra que corregir los hallazgos detectados por una
herramienta no implica que una aplicación sea completamente segura. El
análisis automatizado debe complementarse con revisión del código, pruebas
manuales, controles de autorización y validación de la lógica de negocio.