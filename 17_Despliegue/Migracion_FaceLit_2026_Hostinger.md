# Migración del despliegue a FaceLit-2026

**Estado:** activado y comprobado en Hostinger.
**Verificación:** 2026-11-07 00:39:11 UTC-05:00.
**Sitio:** https://facelit.tech/
**Publicación:** `facelit-2026-20261107T053356Z`.

## Origen de la versión activa

| Repositorio nuevo | Rama | SHA completo desplegado |
|---|---|---|
| [frontend](https://github.com/FaceLit-2026/frontend) | `main` | `bfdb14147ca77ae389be6ea37a1494ce9dfd6184` |
| [backend](https://github.com/FaceLit-2026/backend) | `main` | `495bb043a0d71bc673ca9e0d7fcbbabef5d5b61b` |
| [db](https://github.com/FaceLit-2026/db) | `main` | `dd595dbe653e5d05f112a89e6c22307f4e0d745b` |
| [microservice](https://github.com/FaceLit-2026/microservice) | `main` | `dc6b8851b1d74171779d552e5f1ccccb3d2610a3` |

La configuración de infraestructura está versionada en
[`backend/deploy/hostinger`](https://github.com/FaceLit-2026/backend/tree/main/deploy/hostinger).
Las ramas de trabajo se integraron en los `main` de la nueva organización.
La publicación usa commits confirmados y disponibles en GitHub.

## Qué cambió en la conexión del despliegue

El VPS anterior recibía paquetes por SSH y no contenía copias Git con un remoto
asociado. Se cambió el origen de las fuentes y la configuración operativa:

1. Se trasladaron al frontend y DB nuevos los ajustes de consultas académicas
   por lotes y compatibilidad de migraciones legales ya utilizados en producción.
2. Se sustituyeron los checkouts de CI de la organización anterior y las rutas
   entre repositorios por las de `frontend`, `backend`, `db` y `microservice`.
3. Se versionaron el empaquetador y el controlador del VPS. El paquete se genera
   mediante `git archive`, comprueba las URL de `FaceLit-2026`, exige `main`
   sincronizado y excluye archivos privados. Su SHA-256 es
   `8175122502994262822a097022a4c3a02641782086d150b698cc7910ce25b276`.
4. Se construyeron imágenes desde esos commits. La imagen web ejecuta las
   pruebas del frontend, TypeScript, lint y exportación Expo antes de publicarse.
5. Se respaldó PostgreSQL y se validó Liquibase antes de activar. Los volúmenes
   existentes conservaron base, modelos faciales y certificados. No se
   reinicializaron registros del sistema.
6. Se activaron PostgreSQL, FastAPI, Spring Boot y Caddy desde el directorio
   `/opt/facelit-v2/releases/facelit-2026-20261107T053356Z`. El enlace operativo
   `/opt/facelit-v2/current` apunta a esta versión comprobada.
7. Las credenciales ya rotadas se reutilizaron dentro del VPS en
   `/etc/facelit/production.env`, con permisos privados. Los valores no forman
   parte de los repositorios ni del archivo comprimido.

## Procedencia de las imágenes activas

| Servicio | Identificador de imagen | Revisión fuente |
|---|---|---|
| web | `sha256:341dadae91264a777b24564613e63e63163092a477ec95d9aee362e1299daaa5` | `bfdb14147ca77ae389be6ea37a1494ce9dfd6184` |
| backend | `sha256:a570409a70ae856f6eb2766751c78fa3fbac8b7308d494abfc3b9a223526d97e` | `495bb043a0d71bc673ca9e0d7fcbbabef5d5b61b` |
| facial | `sha256:57cc7dce0311bb20ea84f4efca7b6cf24d2f707b381755da13990df110976c70` | `dc6b8851b1d74171779d552e5f1ccccb3d2610a3` |

Las etiquetas OCI `org.opencontainers.image.source` y
`org.opencontainers.image.revision` coinciden con los repositorios nuevos y
sus commits. PostgreSQL identifica la revisión de `db` mediante sus etiquetas
de despliegue; Liquibase aplica y registra el esquema.

## Comprobaciones de la publicación

| Comprobación | Resultado |
|---|---|
| Sitio y login por HTTPS | HTTP 200. |
| JavaScript publicado | HTTP 200. |
| Backend | Salud HTTP 200. |
| Servicio facial | Salud HTTP 200. |
| Consulta de perfil con sesión coordinadora existente | HTTP 200. |
| PostgreSQL | Conexión y SELECT 1 correctos. |
| Gmail SMTP | Autenticación aceptada con la credencial activa; no se enviaron mensajes. |
| Liquibase | Changelog validado y estado de migraciones comprobado. |
| Origen público | Encabezado `X-FaceLit-Release` correspondiente a esta publicación. |

Estas comprobaciones prueban arranque, procedencia e integración técnica.
El recorrido manual completo de CSV, cámara y asistencia corresponde a las
evidencias funcionales de la carpeta 07 y a las confirmaciones del equipo;
esta migración no inventa nuevas capturas ni mediciones biométricas.

## Actualizaciones y recuperación

El procedimiento vigente está en el README de `backend/deploy/hostinger`.
Para verificar y respaldar desde el VPS:

```sh
python3 /opt/facelit-v2/current/backend/deploy/hostinger/deploy.py verify
python3 /opt/facelit-v2/current/backend/deploy/hostinger/deploy.py backup
```

Los respaldos y registros completos permanecen privados en
`/var/lib/facelit-deploy/facelit-2026-20261107T053356Z`. La configuración y las imágenes
anteriores se conservaron para recuperación. Los repositorios de la organización
anterior no se modificaron para realizar esta migración.

El [informe anterior](Despliegue_Hostinger.md) conserva el registro de la entrega
previa. Este documento identifica el origen de la publicación nueva.
