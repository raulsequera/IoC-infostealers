# Detección conductual de infostealers con Velociraptor

Indicadores de compromiso **de comportamiento** para la detección temprana de malware
tipo *infostealer*, formalizados como consultas VQL sobre Velociraptor.

La detección no usa hash, firma, familia ni relación con un proceso padre sospechoso.
Se apoya en una única conducta que cualquier ladrón de credenciales tiene que ejecutar:
**un proceso que no es el navegador abre las bases de datos SQLite del perfil**
(`Login Data`, `Web Data`, `Cookies`).

Material derivado de un Trabajo Fin de Máster (Universidad de Sevilla).

## Estructura

| Ruta | Contenido |
|---|---|
| `detection/A*.vql` | Capa A. Son las consultas que disparan la detección. |
| `detection/B*.vql` | Capa B. Corroboración y contexto; nunca disparan solas. |
| `artifacts/` | Artefactos `CLIENT_EVENT` de Velociraptor que recogen la telemetría. |

La detección se considera positiva si dispara **A1, o A2, o A3**.

## Requisito previo: sin esto no hay ningún evento

La capa A consume el evento **Security 4663**. Windows **no emite ninguno** salvo que
concurran dos condiciones:

1. La política de auditoría tiene habilitada la subcategoría **Acceso a objetos /
   Sistema de archivos**.
2. Los directorios de perfil de los navegadores llevan una **SACL** que solicite la
   auditoría de lectura.

Si falta cualquiera de las dos, `S1_Read_4663` devuelve **cero filas** y el detector
parece no funcionar. No es un fallo de las consultas.

## Antes de usarlo

Las consultas llevan un `client_id` de ejemplo:

```
client_id='C.XXXXXXXXXXXXXXXX'
```

Sustitúyelo por el identificador del cliente de tu propio despliegue.

## Limitaciones

- **El canal 4663 pierde eventos**, de forma no determinista, en torno a un 12-18 %.
  La pérdida está en el propio pipeline de auditoría de Windows, no en las consultas:
  se han observado lecturas confirmadas por otras fuentes que no generaron ninguna
  fila de 4663.
- **Mide robos consumados, no intentos.** La SACL audita solo accesos con éxito, así
  que un intento de apertura que falla no deja rastro en el canal.
- **Solo navegadores basados en Chromium.** Firefox usa NSS en lugar de DPAPI: el
  principio se traslada, las rutas no.
- **Los cero falsos positivos son de laboratorio**, no de una flota real con software
  heterogéneo.
- La capa B se apoya en un único caso observado por consulta. Refleja un rasgo de
  familia o de compilación concreta, no un patrón generalizable.

## Qué no incluye este repositorio

Muestras de malware, hashes, ni la configuración del laboratorio de análisis.
