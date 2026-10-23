# Feedback Anónimo

Una aplicación web para recopilar feedback anónimo de clientes, usuarios o equipo interno, sin exigir registro ni iniciar sesión para enviar comentarios. El panel administrativo está protegido con autenticación básica, roles simples y un backend realista desarrollado en PHP puro.

## Descripción

Esta app permite:

- Recibir feedback anónimo desde una página pública.
- Permitir que cualquier persona envíe un mensaje sin crear cuenta.
- Guardar los comentarios en una base de datos segura.
- Dar acceso a un administrador autenticado para revisar, filtrar y responder comentarios.
- Mantener un control básico por roles: administrador, moderador y visitante/lector.

La idea es una plataforma simple, útil y fácil de desplegar en un hosting con PHP y MySQL.

---

## Objetivo del proyecto

Crear una herramienta práctica para recoger opiniones honestas sin fricción para el usuario que envía el comentario, mientras el dueño o equipo administrativo puede gestionar la información desde un panel privado.

Ejemplos de uso:

- Feedback de clientes sobre un producto o servicio.
- Sugerencias internas para un equipo.
- Encuestas rápidas con comentarios abiertos.
- Reportes de bugs o mejora de experiencia.

---

## Funcionalidades

### Público

- Formulario para enviar feedback anónimo.
- Campo para nombre opcional (si se desea), pero no obligatorio.
- Categoría o tipo de comentario.
- Mensaje libre con validación básica.
- Captcha o validación simple de spam (opcional en la versión final).
- Protección contra contenido ofensivo o vacío.

### Panel administrativo

- Login con usuario y contraseña.
- Roles básicos:
  - Administrador: acceso total.
  - Moderador: puede revisar, filtrar y marcar comentarios.
  - Lector: solo puede ver reportes y estadísticas básicas.
- Listado de feedback recibido.
- Filtros por categoría, estado y fecha.
- Marcado como leido / revisado / resuelto.
- Eliminar o archivar comentarios.
- Dashboard con estadísticas generales.

### Seguridad mínima

- Autenticación con sesión PHP.
- Protección de rutas privadas.
- Acceso restringido por roles.
- CSRF en formularios de login y administración.
- Hash de contraseñas con password_hash().
- Sanitización y validación de entrada.
- Ocultamiento de rutas a través de front controller y reglas de servidor.

---

## Stack tecnológico

- PHP 8.x
- MySQL o MariaDB
- HTML5 y CSS3
- JavaScript vanilla (opcional para interacciones pequeñas)
- PDO para acceso a base de datos
- Bootstrap o un diseño CSS propio (según prefiera el proyecto)

### Arquitectura sugerida

- `public/`: archivos web accesibles desde el navegador.
- `app/`: lógica de negocio y controladores.
- `src/`: helpers, validaciones, modelo y servicios.
- `config/`: configuración y conexión a la base de datos.
- `templates/`: vistas HTML.
- `storage/`: logs, archivos temporales o exports.

---

## Estructura propuesta del proyecto

```text
feedback-anonimo/
├── public/
│   ├── index.php
│   ├── assets/
│   └── .htaccess
├── app/
│   ├── controllers/
│   ├── middleware/
│   └── services/
├── src/
│   ├── db/
│   ├── helpers/
│   ├── auth/
│   └── validation/
├── config/
│   └── database.php
├── templates/
│   ├── public/
│   └── admin/
├── storage/
│   └── logs/
├── .htaccess
├── composer.json
├── README.md
├── .env.example
└── schema.sql
```

---

## Requisitos

- PHP 8.1 o superior
- Servidor web Apache o Nginx
- MySQL 8 o MariaDB
- Extensión PDO habilitada
- Modulo `rewrite` activo en Apache

---

## Instalación

1. Clona el repositorio:

```bash
git clone https://tu-dominio.com/feedback-anonimo.git
cd feedback-anonimo
```

2. Crea la base de datos:

```sql
CREATE DATABASE feedback_anonimo;
```

3. Importa el esquema inicial:

```bash
mysql -u usuario -p feedback_anonimo < schema.sql
```

4. Configura la conexión a la base de datos:

- Copia `.env.example` a `.env` o crea `config/database.php` con tus credenciales.
- Ajusta host, usuario, contraseña y nombre de la base de datos.

5. Configura el servidor web para apuntar a la carpeta `public/`.

6. Inicia el proyecto y accede desde el navegador.

---

## Configuración básica

Ejemplo de configuración aproximada para `config/database.php`:

```php
<?php
return [
    'host' => 'localhost',
    'dbname' => 'feedback_anonimo',
    'user' => 'root',
    'pass' => '',
    'charset' => 'utf8mb4',
];
```

Ejemplo de variables de entorno:

```env
DB_HOST=localhost
DB_NAME=feedback_anonimo
DB_USER=root
DB_PASS=
APP_URL=http://localhost/feedback-anonimo
```

---

## Seguridad mínima recomendada

Aunque el proyecto está hecho en PHP puro, se recomienda implementar estas medidas básicas:

- Redirigir todo el tráfico a `public/index.php`.
- Bloquear acceso directo a archivos sensibles.
- No guardar secretos en rutas públicas.
- Usar `password_hash()` y `password_verify()`.
- Usar `session_start()` con cookies seguras.
- Generar `CSRF tokens` en formularios.
- Validar todas las entradas del usuario.
- Evitar mostrar errores de PHP en producción.
- Usar `headers` básicos para seguridad (`X-Frame-Options`, `Content-Security-Policy`).

### Ocultar las rutas

La idea es evitar que alguien acceda directamente a archivos como:

- `app/controllers/*.php`
- `src/auth/*.php`
- `config/*.php`

Esto se logra con:

- Una carpeta `public/` como único punto de entrada.
- Un `index.php` frontal.
- Un `.htaccess` con rewrites y bloqueo de acceso.
- Reglas de servidor que dirijan todas las peticiones a un único controlador.

Ejemplo de `.htaccess` básico:

```apache
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/public/ [NC]
RewriteRule ^(.*)$ public/index.php [QSA,L]
```

Y en la carpeta `app/` o `config/`:

```apache
Deny from all
```

---

## Roles sugeridos

```text
admin
moderator
viewer
```

### Permisos básicos

- Admin: crea usuarios, elimina comentarios, modifica roles, acceso total.
- Moderator: revisa y clasifica comentarios, marca estados.
- Viewer: solo consulta reportes y estadística.

---

## Modelo de datos sugerido

### Tabla `feedback`

```sql
CREATE TABLE feedback (
    id INT AUTO_INCREMENT PRIMARY KEY,
    category VARCHAR(50) NOT NULL,
    message TEXT NOT NULL,
    is_anonymous TINYINT(1) DEFAULT 1,
    status ENUM('nuevo', 'leido', 'resuelto', 'archivado') DEFAULT 'nuevo',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla `users`

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('admin', 'moderator', 'viewer') DEFAULT 'viewer',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Flujo de uso

### Envío de feedback anónimo

1. El usuario accede a la página pública.
2. Completa el formulario.
3. El sistema valida el contenido.
4. Se guarda en la base de datos como comentario anónimo.
5. El administrador puede revisarlo luego desde su panel.

### Login del administrador

1. El usuario entra a la ruta privada del panel.
2. Se valida la sesión.
3. Según el rol, se muestran diferentes vistas y permisos.
4. El acceso a rutas no autorizadas se bloquea.

---

## Mapa de rutas sugerido

```text
/                    -> Página pública de feedback
/login              -> Login administrativo
/logout             -> Cierre de sesión
/admin              -> Dashboard principal
/admin/feedback     -> Lista de comentarios
/admin/feedback/:id -> Detalle del comentario
/admin/users        -> Gestión de usuarios (solo admin)
/admin/reports      -> Reportes y estadísticas
```

---

## Consideraciones de UX

- Página pública enfocada en simplicidad.
- Botón claro para enviar feedback.
- Mensajes de confirmación tras enviar el formulario.
- Diseño limpio, sin mostrar información personal del remitente.
- Panel administrativo con filtros y orden por fecha.

---

## Mejoras futuras

- Respuestas del administrador a comentarios.
- Moderación automática de mensajes ofensivos.
- Exportación a CSV o Excel.
- Notificaciones por email.
- Captcha o verificación anti-spam.
- Api REST con autenticación JWT (más adelante).
- Dashboard con gráficos y métricas.

---

## Nota de desarrollo

Este proyecto está pensado para ser una base sólida, realista y fácil de mantener, usando PHP puro sin frameworks, pero con una estructura más organizada y segura que un prototipo simple.

La intención es mantener un equilibrio entre:

- simplicidad para comenzar,
- seguridad mínima para evitar accesos no autorizados,
- estructura clara para crecer con nuevas funcionalidades.

---

## Licencia

Este proyecto se distribuye bajo licencia MIT, salvo que decidas cambiarla para tu caso de uso.

---

## Autor

Proyecto de Edwin Rodriguez para practicar PHP, rutas protegidas, autenticación básica y administración de feedback anónimo.

