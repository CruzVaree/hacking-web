
# Guía de Cross-Site Scripting (XSS)

## Índice

1. [¿Qué es XSS?](#1-qué-es-xss)
2. [Tipos de XSS](#2-tipos-de-xss)
3. [Cómo detectar XSS](#3-cómo-detectar-xss)
4. [Payloads básicos](#4-payloads-básicos)
5. [Payloads HTML y atributos](#5-payloads-html-y-atributos)
6. [Payloads SVG y eventos](#6-payloads-svg-y-eventos)
7. [Payloads para XSS basado en DOM](#7-payloads-para-xss-basado-en-dom)
8. [Codificación de payloads](#8-codificación-de-payloads)
9. [Cómo interpretar los resultados](#9-cómo-interpretar-los-resultados)
10. [Herramientas para detectar XSS](#10-herramientas-para-detectar-xss)
11. [Cómo prevenir XSS](#11-cómo-prevenir-xss)
12. [Lista de verificación](#12-lista-de-verificación)
13. [Referencias](#13-referencias)

---

## 1. ¿Qué es XSS?

El **Cross-Site Scripting (XSS)** es una vulnerabilidad de seguridad web que permite que datos controlados por un usuario sean interpretados como código ejecutable en el navegador de otra persona.

Dependiendo de la vulnerabilidad, un atacante podría modificar el contenido de una página, ejecutar acciones con los permisos de la víctima o acceder a información disponible para el código de la aplicación.

XSS suele aparecer cuando una aplicación introduce datos no confiables en HTML, atributos, URLs o código JavaScript sin aplicar el tratamiento adecuado.

### ¿Qué puede provocar?

- Modificación del contenido de una página.
- Suplantación visual de formularios o pantallas.
- Ejecución de acciones con los permisos de la víctima.
- Acceso a información disponible para JavaScript.
- Redirecciones no deseadas.
- Ataques contra otros usuarios que visitan una página vulnerable.

El impacto real depende de los permisos de la aplicación, las protecciones del navegador y el contexto donde se ejecuta el código.

---

## 2. Tipos de XSS

### 2.1. XSS reflejado (Reflected XSS)

Sucede cuando una aplicación recibe datos mediante una solicitud HTTP y los incorpora inmediatamente en una respuesta sin tratarlos de forma segura.

Ejemplo de URL:

```text
https://ejemplo.test/search?term=libros
```

La aplicación podría mostrar:

```html
<p>Resultados para: libros</p>
```

Si el valor de `term` se inserta de manera insegura, una entrada especialmente construida podría interpretarse como HTML o JavaScript.

**Cómo detectarlo:**

1. Identifica parámetros de URL y campos de búsqueda.
2. Introduce un marcador como `xss-prueba-123`.
3. Comprueba si aparece en la respuesta.
4. Inspecciona el contexto donde se refleja.
5. Realiza una prueba inocua en un entorno autorizado.

### 2.2. XSS almacenado (Stored XSS)

Ocurre cuando una aplicación almacena datos controlados por un usuario y posteriormente los muestra sin procesarlos de forma segura.

Ejemplos de lugares donde podría aparecer:

- Comentarios.
- Perfiles de usuario.
- Mensajes.
- Publicaciones.
- Nombres visibles.
- Paneles administrativos.

Ejemplo conceptual:

```html
<p>Comentario: [contenido proporcionado por el usuario]</p>
```

Si la aplicación interpreta el contenido del comentario como código, podría ejecutarse cada vez que otro usuario visite la página.

**Cómo detectarlo:**

1. Introduce un marcador único en un campo que se almacene.
2. Guarda el registro.
3. Abre la página donde se presenta.
4. Revisa otras vistas que reutilicen el dato.
5. Comprueba si la entrada se interpreta como HTML o código.
6. Elimina los datos de prueba cuando termines.

### 2.3. XSS basado en DOM (DOM-based XSS)

Se produce cuando el JavaScript del navegador toma datos controlables por el usuario y los utiliza de forma insegura.

Fuentes frecuentes:

- `location.search`
- `location.hash`
- `window.location`
- `document.referrer`
- Datos de formularios.
- Mensajes recibidos mediante `postMessage`.

Destinos que requieren especial atención:

- `innerHTML`
- `outerHTML`
- `document.write()`
- `eval()`
- `new Function()`

Estas APIs no son vulnerables automáticamente. El riesgo aparece cuando datos no confiables llegan a ellas de una forma que permite interpretar contenido como código.

**Cómo detectarlo:**

1. Abre las herramientas de desarrollador.
2. Busca referencias a `location.search` y `location.hash`.
3. Sigue el recorrido del dato por el código.
4. Identifica si llega a un destino peligroso.
5. Inspecciona el DOM actualizado.
6. Utiliza puntos de interrupción para examinar los valores.

### 2.4. XSS ciego (Blind XSS)

Es una variante en la que la entrada podría ejecutarse en una interfaz que quien realiza la prueba no observa directamente.

Por ejemplo, un campo enviado a un sistema de soporte podría mostrarse después en un panel administrativo.

Debe evaluarse únicamente con autorización y mecanismos de prueba controlados.

### 2.5. XSS mutado (mXSS)

Sucede cuando transformaciones del HTML realizadas por el navegador o por un proceso de sanitización terminan creando una estructura peligrosa.

Es especialmente relevante en aplicaciones que permiten HTML enriquecido.

La prevención requiere sanitización robusta, actualizada y pruebas sobre el HTML resultante.

---

## 3. Cómo detectar XSS

### Paso 1. Identificar los puntos de entrada

Busca datos controlables por el usuario en:

- Parámetros GET.
- Formularios POST.
- Fragmentos de URL.
- Campos de comentarios.
- Perfiles.
- Mensajes.
- Cabeceras que posteriormente se muestran en una página.
- Datos importados desde otros servicios.

### Paso 2. Introducir un marcador

Empieza con una cadena inocua:

```text
xss-prueba-7f31
```

Busca esa cadena en:

- La respuesta HTTP.
- El código HTML recibido.
- La pestaña Elements de DevTools.
- El contenido generado por JavaScript.

El marcador ayuda a identificar el recorrido del dato sin ejecutar código.

### Paso 3. Identificar el contexto

Determina dónde aparece el valor:

**Texto HTML:**

```html
<p>VALOR</p>
```

**Atributo HTML:**

```html
<input value="VALOR">
```

**Bloque JavaScript:**

```html
<script>
const dato = "VALOR";
</script>
```

**URL:**

```html
<a href="VALOR">Abrir</a>
```

Cada contexto necesita una estrategia de protección distinta.

### Paso 4. Probar el comportamiento

En un laboratorio autorizado, utiliza los payloads de las siguientes secciones.

Comprueba si:

- El navegador muestra el contenido literalmente.
- Las etiquetas se interpretan como HTML.
- Los atributos se conservan.
- Los eventos se ejecutan.
- El dato llega a un destino JavaScript peligroso.

### Paso 5. Documentar el resultado

Registra:

- URL o ruta afectada.
- Parámetro o campo vulnerable.
- Tipo de XSS.
- Payload utilizado.
- Resultado observado.
- Pasos para reproducirlo.
- Corrección recomendada.

---

## 4. Payloads básicos

Un **payload** es una cadena diseñada para comprobar cómo procesa una aplicación una entrada.

Los siguientes ejemplos sirven para verificar comportamientos concretos en un laboratorio propio.

| Payload | Qué comprueba |
|---|---|
| `xss-prueba-123` | Si la entrada se refleja o almacena. |
| `<b>Prueba</b>` | Si el contenido se interpreta como HTML. |
| `<i>Prueba</i>` | Si se permite marcado HTML. |
| `<u>Prueba</u>` | Si se interpreta una etiqueta HTML. |
| `<h1>Prueba</h1>` | Si la entrada puede modificar la estructura visual. |
| `<p>Prueba</p>` | Si se aceptan etiquetas de párrafo. |
| `<script>alert(1)</script>` | Si un bloque de script inyectado llega a ejecutarse. |
| `<img src=x onerror=alert(1)>` | Si se permite un manejador de error HTML. |
| `<svg onload=alert(1)>` | Si se permite un manejador de carga SVG. |

### 4.1. Marcador alfanumérico

```text
xss-prueba-123
```

**Funcionamiento:** no ejecuta código. Permite localizar la entrada en la respuesta o en el DOM.

Es el primer paso recomendado para rastrear datos.

### 4.2. Prueba de interpretación HTML

```html
<b>Prueba</b>
```

**Funcionamiento:** permite comprobar si el navegador interpreta la etiqueta como HTML.

Si aparece en negrita, el contenido se ha interpretado como marcado. Esto no demuestra por sí solo que exista ejecución de JavaScript, pero puede indicar un problema de tratamiento de HTML.

### 4.3. Prueba con script

```html
<script>alert(1)</script>
```

**Funcionamiento:** intenta ejecutar JavaScript y mostrar un aviso local.

Si funciona, demuestra que el navegador interpretó ese bloque en un contexto ejecutable. Algunos contextos, políticas CSP y métodos de inserción impiden su ejecución.

### 4.4. Prueba con imagen

```html
<img src=x onerror=alert(1)>
```

**Funcionamiento:** intenta provocar un error de carga y ejecutar el manejador `onerror`.

Si el navegador procesa el elemento y permite el evento, aparece un aviso.

### 4.5. Prueba con SVG

```html
<svg onload=alert(1)>
```

**Funcionamiento:** utiliza el evento de carga de SVG para comprobar si el marcado y sus manejadores se interpretan.

El resultado depende del contexto, el navegador y las protecciones aplicadas.

---

## 5. Payloads HTML y atributos

Estas pruebas permiten comprobar cómo trata la aplicación distintos elementos y atributos HTML.

| Payload | Propósito |
|---|---|
| `<div>Prueba</div>` | Comprobar la interpretación de un elemento contenedor. |
| `<span>Prueba</span>` | Comprobar la interpretación de un elemento en línea. |
| `<strong>Prueba</strong>` | Comprobar el formato HTML. |
| `<img src=x>` | Comprobar si se permite un elemento de imagen. |
| `<a href="https://example.com">Enlace</a>` | Comprobar cómo se representan los enlaces. |
| `<input value="Prueba">` | Comprobar la interpretación de un elemento de formulario. |
| `<details open><summary>Prueba</summary></details>` | Comprobar si se interpreta un elemento HTML interactivo. |

### 5.1. Salir de un atributo entre comillas

Ejemplo de contexto:

```html
<input value="VALOR">
```

Prueba de laboratorio:

```html
"><svg onload=alert(1)>
```

**¿Qué intenta comprobar?**

1. Si el valor se inserta dentro del atributo `value`.
2. Si las comillas permiten cerrar ese atributo.
3. Si el navegador interpreta el contenido posterior como nuevo marcado.
4. Si el elemento introducido puede ejecutar un evento.

Si la aplicación escapa correctamente las comillas y trata la entrada como dato, la prueba no debería ejecutar código.

### 5.2. Atributos de eventos

Ejemplo:

```html
<img src=x onerror=alert(1)>
```

El atributo `onerror` contiene JavaScript que se intenta ejecutar cuando falla la carga de la imagen.

Otros nombres de eventos que pueden aparecer en auditorías incluyen:

- `onload`
- `onerror`
- `onclick`
- `onfocus`

No todos funcionan en todos los elementos o contextos. La presencia de un atributo no garantiza que el evento se dispare.

### 5.3. URLs controladas por el usuario

Cadena de prueba:

```text
javascript:alert(1)
```

**¿Qué comprueba?**

Si una aplicación acepta una URL con un esquema ejecutable en un contexto donde el navegador permite su ejecución.

Debe comprobarse especialmente en enlaces y otros atributos que reciben URLs.

Una aplicación segura debe validar los esquemas permitidos y rechazar los que no correspondan a su funcionalidad.

---

## 6. Payloads SVG y eventos

SVG admite elementos gráficos y distintos atributos. En ciertos contextos, algunos manejadores de eventos pueden ejecutar JavaScript.

| Payload de prueba | Qué intenta comprobar |
|---|---|
| `<svg onload=alert(1)>` | Evento de carga de SVG. |
| `<svg><script>alert(1)</script></svg>` | Tratamiento de elementos script dentro de SVG. |
| `<img src=x onerror=alert(1)>` | Evento de error de imagen. |
| `<iframe srcdoc="<script>alert(1)</script>"></iframe>` | Tratamiento del atributo `srcdoc` de un iframe. |

Estos ejemplos son pruebas de comportamiento, no una lista de cadenas que funcionen universalmente.

Los navegadores pueden aplicar restricciones según el contexto de inserción, las políticas de contenido y la forma en que se construye el documento.

**Qué revisar:**

1. Si el elemento se conserva.
2. Si sus atributos permanecen después de la sanitización.
3. Si el navegador crea el elemento esperado.
4. Si el evento se ejecuta.
5. Si el HTML final contiene estructuras no permitidas.

Cuando una aplicación permite HTML enriquecido, utiliza un sanitizador reconocido y actualizado, configurado con una lista permitida de elementos y atributos.

---

## 7. Payloads para XSS basado en DOM

Las siguientes pruebas son útiles para estudiar cómo una aplicación procesa parámetros y fragmentos de URL.

### 7.1. Prueba con `location.search`

URL de ejemplo:

```text
https://ejemplo.test/?q=xss-prueba-123
```

Si el código de la aplicación lee el parámetro `q`, rastrea cómo utiliza su valor.

Ejemplo vulnerable:

```javascript
const params = new URLSearchParams(location.search);
const q = params.get("q");

document.querySelector("#resultado").innerHTML = q;
```

En este ejemplo, el dato termina en `innerHTML`, que interpreta contenido como HTML.

Prueba de laboratorio:

```text
?q=%3Cb%3Exss-prueba%3C%2Fb%3E
```

El valor decodificado es:

```html
<b>xss-prueba</b>
```

Si el navegador muestra el texto en negrita, el dato se está interpretando como HTML.

### 7.2. Prueba con `location.hash`

URL de ejemplo:

```text
https://ejemplo.test/#xss-prueba-123
```

El fragmento situado después de `#` puede ser leído por JavaScript mediante `location.hash`.

Busca referencias a esa propiedad y rastrea si su valor llega a un destino inseguro.

El fragmento no se envía normalmente al servidor como parte de la solicitud HTTP, por lo que puede revelar vulnerabilidades que no aparecen al revisar solamente las respuestas del servidor.

### 7.3. Ejemplo de DOM vulnerable

```javascript
const valor = new URLSearchParams(location.search).get("nombre");

document.querySelector("#saludo").innerHTML =
    `Hola, ${valor}`;
```

Si el parámetro `nombre` contiene HTML, este código puede interpretarlo como marcado.

### 7.4. Ejemplo corregido

```javascript
const valor = new URLSearchParams(location.search).get("nombre") ?? "";

document.querySelector("#saludo").textContent =
    `Hola, ${valor}`;
```

La propiedad `textContent` trata el contenido como texto y no lo analiza como HTML.

### 7.5. Otras fuentes que conviene revisar

```javascript
location.search
location.hash
document.referrer
window.name
```

Busca dónde se leen estas fuentes y sigue los valores hasta su uso final.

No todas las fuentes son vulnerables por sí mismas. El riesgo depende del tratamiento posterior.

---

## 8. Codificación de payloads

La codificación puede cambiar la representación de una cadena durante su transporte. No constituye una protección automática contra XSS.

### 8.1. Ejemplo de codificación URL

Texto original:

```html
<b>Prueba</b>
```

Representación codificada:

```text
%3Cb%3EPrueba%3C%2Fb%3E
```

En JavaScript:

```javascript
encodeURIComponent("<b>Prueba</b>");
```

La codificación URL representa caracteres de una cadena para su uso en componentes de una URL.

Si una aplicación decodifica el valor y posteriormente lo inserta de forma insegura en HTML, la vulnerabilidad puede seguir existiendo.

### 8.2. Ejemplo con un parámetro de búsqueda

```text
https://ejemplo.test/?q=%3Cb%3EPrueba%3C%2Fb%3E
```

Al procesar el parámetro, la aplicación puede obtener:

```html
<b>Prueba</b>
```

La prueba consiste en comprobar si el valor se muestra como texto o se interpreta como HTML.

### 8.3. Diferencia entre codificar y sanitizar

**Codificación de salida:** transforma caracteres para que se interpreten correctamente en un contexto específico.

**Sanitización:** analiza un contenido, como HTML enriquecido, y elimina o modifica las estructuras no permitidas.

**Validación:** comprueba que un valor cumple las reglas de la aplicación.

Estas técnicas tienen propósitos distintos y no deben tratarse como equivalentes.

---

## 9. Cómo interpretar los resultados

| Resultado observado | Interpretación |
|---|---|
| El marcador aparece como texto literal. | La entrada puede estar tratándose como texto; sigue revisando otros contextos. |
| Las etiquetas HTML cambian la presentación. | El navegador está interpretando marcado HTML. |
| Se muestra un aviso de JavaScript en el laboratorio. | Se ha confirmado ejecución en ese contexto concreto. |
| El dato aparece en el DOM, pero no se ejecuta. | Puede estar neutralizado, en un contexto no ejecutable o bloqueado por otras protecciones. |
| La entrada no aparece en la página. | Puede haberse eliminado, transformado, almacenado o procesado en otra parte. |
| Una prueba no funciona en un navegador. | No demuestra por sí sola que la aplicación sea segura. |

### Factores que pueden afectar al resultado

- Contexto HTML donde se inserta el dato.
- Codificación aplicada por la aplicación.
- Sanitización de HTML.
- Navegador utilizado.
- Políticas de seguridad de contenido (CSP).
- Forma de creación de los elementos en el DOM.
- Atributos y eventos permitidos.
- Framework o biblioteca utilizados.

Una prueba positiva debe documentarse con sus pasos de reproducción y el contexto exacto. Una prueba negativa aislada no demuestra que no exista XSS.

---

## 10. Herramientas para detectar XSS

### 10.1. DevTools

Las herramientas de desarrollador del navegador permiten inspeccionar el DOM y seguir el comportamiento de JavaScript.

**Elementos:**
- Inspeccionar HTML renderizado.
- Buscar marcadores.
- Revisar atributos y elementos.
- Observar cambios realizados por JavaScript.

**Consola:**
- Revisar errores.
- Identificar problemas de ejecución.
- Observar mensajes de seguridad.

**Sources o Depurador:**
- Buscar fuentes controlables.
- Establecer puntos de interrupción.
- Seguir variables.
- Inspeccionar los valores antes de llegar a un destino.

**Network o Red:**
- Inspeccionar solicitudes y respuestas HTTP.
- Revisar parámetros.
- Comprobar cómo responde la aplicación.

### 10.2. Burp Suite

Permite interceptar y examinar solicitudes HTTP en pruebas autorizadas.

Puede utilizarse para:
- Identificar parámetros.
- Modificar valores de prueba.
- Comparar respuestas.
- Repetir solicitudes.
- Documentar comportamientos.

### 10.3. OWASP ZAP

Es una herramienta de pruebas de seguridad web que ayuda a detectar posibles vulnerabilidades.

Puede utilizarse para:
- Explorar una aplicación.
- Examinar solicitudes y respuestas.
- Ejecutar análisis autorizados.
- Investigar posibles problemas de XSS.

Los resultados automatizados deben confirmarse manualmente.

### 10.4. Análisis del código

Busca usos de APIs que requieren atención especial:

```javascript
innerHTML
outerHTML
document.write
eval
new Function
```

También revisa cómo se procesan los datos procedentes de URLs, formularios, almacenamiento local y otras fuentes controlables.

La presencia de estas APIs no demuestra una vulnerabilidad: es necesario comprobar el origen del dato y el contexto donde se utiliza.

---

## 11. Cómo prevenir XSS

### 11.1. Preferir `textContent`

Código vulnerable:

```javascript
const nombre = new URLSearchParams(location.search).get("nombre");

document.querySelector("#saludo").innerHTML =
    `Hola, ${nombre}`;
```

Código corregido:

```javascript
const nombre = new URLSearchParams(location.search).get("nombre") ?? "";

document.querySelector("#saludo").textContent =
    `Hola, ${nombre}`;
```

`textContent` muestra el contenido como texto en lugar de analizarlo como HTML.

### 11.2. Codificación contextual

Aplica la codificación apropiada al lugar donde se introduce la entrada.

El tratamiento necesario para texto HTML puede ser distinto del necesario para atributos, URLs o código JavaScript.

No construyas fragmentos de HTML o JavaScript concatenando directamente datos no confiables.

### 11.3. Sanitización de HTML

Si la aplicación necesita permitir formato HTML, utiliza una biblioteca de sanitización reconocida, actualizada y configurada para permitir únicamente los elementos y atributos necesarios.

No intentes crear un sanitizador HTML mediante expresiones regulares.

### 11.4. Evitar la ejecución dinámica

Evita construir código a partir de datos controlados por el usuario mediante `eval()` o `new Function()`.

Mantén los datos separados del código y utiliza APIs que no interpreten la entrada como instrucciones ejecutables.

### 11.5. Validar URLs

Cuando los usuarios puedan proporcionar enlaces:

- Analiza la URL.
- Permite únicamente los esquemas necesarios.
- Rechaza esquemas no autorizados.
- Valida el destino según los requisitos de la aplicación.

### 11.6. Configurar Content Security Policy

CSP puede reducir el impacto de determinadas formas de XSS al restringir los recursos y las ejecuciones permitidas.

Debe configurarse cuidadosamente y utilizarse como una capa adicional, no como sustituto de corregir la vulnerabilidad.

### 11.7. Proteger las cookies

Cuando corresponda, configura los atributos:

- `HttpOnly`
- `Secure`
- `SameSite`

`HttpOnly` impide que JavaScript lea directamente esa cookie, pero no impide todos los efectos que puede tener un XSS sobre una sesión activa.

### 11.8. Añadir pruebas de regresión

Después de corregir una vulnerabilidad:

1. Repite la prueba original.
2. Comprueba que la entrada se trata como texto o se sanitiza correctamente.
3. Revisa las páginas que reutilizan el dato.
4. Añade una prueba automatizada.
5. Confirma que no se ha introducido otra vulnerabilidad.

---

## 12. Lista de verificación

- [ ] Tengo autorización para probar la aplicación.
- [ ] He identificado las entradas controlables por el usuario.
- [ ] He utilizado marcadores inocuos para rastrear los datos.
- [ ] He revisado el HTML recibido y el DOM actualizado.
- [ ] He identificado el contexto de cada entrada.
- [ ] He revisado fuentes y destinos de JavaScript.
- [ ] He comprobado los resultados en un entorno controlado.
- [ ] He documentado los pasos para reproducir el problema.
- [ ] He aplicado una corrección apropiada.
- [ ] He repetido las pruebas después de corregirlo.
- [ ] He añadido pruebas de regresión.
- [ ] He evitado extraer información o afectar a usuarios reales.

---

## 13. Referencias

- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [PortSwigger Web Security Academy: Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
- [MDN: Cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/XSS)
- [MDN: Element.innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)
- [MDN: Node.textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)

---

**Nota de seguridad:** esta guía es educativa. Ejecuta las pruebas únicamente en sistemas propios, laboratorios de práctica o aplicaciones para las que tengas autorización explícita.

