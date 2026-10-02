# TallerAspirantes

## Objetivo
Desarrollar un sistema de registro de aspirantes con **HTML5, Bootstrap 5.3.8 y PHP**, que valide los datos en el backend, estandarice los textos y guarde la fotografía del aspirante de forma segura **sin usar base de datos**. El sitio se modulariza con `include` (header y footer) y usa etiquetas semánticas (`<header>`, `<main>`, `<section>`, `<footer>`).

---

## Requisitos Previos

| Tecnología | Versión |
|---|---|
| PHP | 8.0 o superior (probado con 8.3) |
| Apache | 2.4 (incluido en WAMP) |
| WampServer | 64-bit |
| Extensiones PHP | `mbstring` y `fileinfo` activas |
| Bootstrap | 5.3.8 (vía CDN) |
| Visual Studio Code | Recomendado |
| Git | Para el repositorio |
| Sistema Operativo | Windows |

> No requiere base de datos, Composer ni Node.js.

---

## Estructura de Carpetas

```
TallerAspirantes/
├── includes/
│   ├── header.php        # <header>, metadatos, Navbar y Breadcrumb dinámico
│   └── footer.php        # <footer> con enlaces y año dinámico
├── uploaded_files/       # Fotos subidas (bloqueada desde el navegador)
│   ├── .htaccess         # Deniega el acceso desde el navegador
│   └── .gitkeep          # Mantiene la carpeta vacía en Git
├── assets/               # Capturas para este README
├── index.php             # Página principal con el formulario de registro
├── procesar.php          # Backend: valida, procesa y muestra el resultado
├── .gitignore
└── README.md
```

| Archivo | Función |
|---|---|
| `includes/header.php` | Metadatos, Bootstrap, navbar y migas de pan según la página actual (`basename`) |
| `includes/footer.php` | Eslogan institucional, enlaces, copyright con `date('Y')` y cierre de `</body></html>` |
| `index.php` | Formulario dentro de `<main><section>` con `enctype="multipart/form-data"` |
| `procesar.php` | Validaciones, estandarización de textos, cálculo de edad y guardado de la foto |
| `uploaded_files/` | Destino de las fotografías, protegido con `.htaccess` |

---

## Instalación y Configuración

### 1. Clonar el repositorio dentro de WAMP
```bash
cd C:\wamp64\www
git clone https://github.com/TU_USUARIO/TallerAspirantes.git
```

### 2. Iniciar WampServer
Verificar que los servicios estén en verde (Apache y PHP).

### 3. Activar extensiones de PHP
En el menú de WampServer: **PHP → Extensiones PHP** y marcar `mbstring` y `fileinfo`. Reiniciar los servicios.

### 4. Permitir el uso de `.htaccess`
En `httpd.conf`, el bloque del directorio `www` debe tener:
```apache
AllowOverride All
```

### 5. Verificar que exista el archivo `.htaccess`
Debe llamarse exactamente `uploaded_files/.htaccess` (con el punto al inicio).

---

## Funcionamiento

### Campos del formulario

| Campo | Tipo | Requerido |
|---|---|---|
| Nombre | `text` | Sí |
| Apellido | `text` | Sí |
| Identificación | `text` | Sí |
| Fecha de nacimiento | `date` | Sí |
| Sexo | `radio` (Hombre / Mujer) | Sí |
| Fotografía | `file` (png, jpg, jpeg, gif, webp) | Sí |

### Validaciones en `procesar.php`

| Validación | Detalle |
|---|---|
| Campos vacíos | Ningún campo puede quedar vacío (incluye solo espacios) |
| Formato de texto | Nombre y apellido solo con letras; identificación con letras, números y guiones |
| Edad | Calculada desde la fecha de nacimiento; debe estar **entre 18 y 70 años** |
| Foto: extensión | jpg, jpeg, png, gif, webp |
| Foto: contenido | Se verifica el tipo MIME real (`finfo`) y `getimagesize()` |
| Foto: tamaño | Máximo 2 MB |

### Funciones de saneamiento y normalización

| Función | Uso en el proyecto |
|---|---|
| `trim()` | Elimina espacios al inicio y al final |
| `strip_tags()` | Elimina etiquetas HTML/PHP de los textos |
| `htmlspecialchars()` | Escapa toda la salida para prevenir XSS |
| `mb_convert_case(mb_strtolower())` | Nombre y apellido en formato título (equivale a `ucwords(strtolower())`, compatible con tildes) |
| `mb_strtoupper()` | Identificación en mayúsculas |

### Seguridad de la carpeta de fotos
- `uploaded_files/.htaccess` deniega cualquier acceso desde el navegador (**403 Forbidden**).
- Las fotos se guardan con **nombre aleatorio** (`fecha_hora_codigo.ext`), no con el nombre original.
- La foto se muestra en el resultado leyéndola desde el servidor, sin enlazarla directamente.
- El formulario incluye un **token CSRF** validado en el backend.

---

## Ejecución del proyecto

1. Iniciar WampServer.
2. Abrir en el navegador:

```
http://localhost/TallerAspirantes/
```

> Al abrir la carpeta, Apache carga automáticamente `index.php`.

---

## Resultado

**Formulario de registro**

![Formulario](assets/formulario.png)

**Registro exitoso**

![Registro exitoso](assets/resultado.png)

**Validación de edad**

![Error de edad](assets/error_edad.png)

**Carpeta de fotos protegida (403)**

![Acceso denegado](assets/forbidden.png)

---

## Dificultades y Soluciones

**Problema 1: El `.htaccess` mostraba su contenido en el navegador**
> Al abrir la carpeta de fotos se leía el texto del archivo en vez de bloquear el acceso.

**Solución:** El archivo había perdido el punto inicial y se llamaba `htaccess`. Renombrarlo a `.htaccess` (desde VS Code o con `ren htaccess .htaccess`). Después de eso la carpeta responde con **403 Forbidden**.

---

**Problema 2: Caracteres raros en los comentarios (`ejecuciÃ³n`)**
> El navegador mostraba mal las tildes de los comentarios del `.htaccess`.

**Solución:** Es un problema de codificación al mostrar el archivo. Se evita escribiendo los comentarios del `.htaccess` sin tildes.

---

**Problema 3: "sofia" no se convertía en "Sofía"**
> El formato título pone la inicial en mayúscula, pero PHP no agrega tildes que el usuario no escribió.

**Solución:** Usar `mb_convert_case(mb_strtolower($texto), MB_CASE_TITLE)`, que conserva las tildes escritas ("SOFÍA" pasa a "Sofía"). Las funciones sin `mb_` no manejan bien los caracteres con tilde.

---

**Problema 4: No aparecían fotos en `uploaded_files/`**
> Después de enviar el formulario, la carpeta solo tenía el `.htaccess`.

**Solución:** La foto solo se guarda cuando todas las validaciones pasan. Se hizo un registro con datos válidos (edad entre 18 y 70 y una imagen menor a 2 MB) y la foto apareció en la carpeta.

---

**Problema 5: Evitar subir las fotos de prueba al repositorio**
> Las fotos contienen datos personales y no deben quedar en GitHub.

**Solución:** Crear un `.gitignore` que ignora el contenido de `uploaded_files/` pero conserva `.gitkeep` y `.htaccess`:
```
uploaded_files/*
!uploaded_files/.gitkeep
!uploaded_files/.htaccess
```

---

## Referencias

- [PHP: Manejo de subida de archivos](https://www.php.net/manual/es/features.file-upload.php)
- [PHP: htmlspecialchars](https://www.php.net/manual/es/function.htmlspecialchars.php)
- [PHP: mb_convert_case](https://www.php.net/manual/es/function.mb-convert-case.php)
- [Bootstrap 5.3](https://getbootstrap.com/docs/5.3/)
- [Bootstrap Icons](https://icons.getbootstrap.com)
- [Apache: .htaccess](https://httpd.apache.org/docs/2.4/howto/htaccess.html)

---

## Fecha de Ejecución
1 de Octubre de 2026

---

| | |
|---|---|
| **Nombre** | Marcos Navarro |
| **Correo** | marcos.navarro@utp.ac.pa |
| **Curso** | Desarrollo Web |
| **Instructor** | Ing. Irina Fong |
