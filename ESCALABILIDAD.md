# Escalabilidad — Requerimientos Funcionales pendientes

Backlog de lo que falta para cumplir la lista de Requerimientos Funcionales (FL-RF-01 a FL-RF-19).
No es código, es la hoja de ruta para cuando se decida escalar el sistema más allá del flujo
actual de "levantar ticket + pedir vehículo".

Estado actual: 0 de 19 cumplidos por completo, 6 parciales, 13 sin cumplir. El bloqueo de fondo
es que hoy no existe un módulo de **Reportes** como entidad consultable — el formulario solo hace
un `INSERT` a una hoja vía Apps Script; no hay forma de releer, editar o cerrar esos reportes
después. Por eso el trabajo se agrupa en bloques en vez de requerimiento por requerimiento.

## Decisión de fondo (antes de estimar)

Todo lo de abajo se puede construir de dos formas:

- **Extender lo que ya existe** (Google Sheets + Apps Script). Barato, consistente con la
  arquitectura actual, iterable por partes.
- **Migrar a una base de datos real con backend propio.** Más limpio para manejar 6 estados,
  filtros, adjuntos y analítica a mediano plazo, pero implica prácticamente reescribir el sistema.

No está decidido. Cada bloque de abajo asume la ruta "extender Apps Script"; si se opta por
backend propio, el trabajo es equivalente pero las piezas concretas (endpoints, tablas) cambian.

## Bloque A — Backend: módulo de Reportes (habilita 02, 07, 08, 09, 10, 11, 15, 16, 17, 18)

- Acción `listar_reportes` en el Apps Script (mismo patrón que ya existe `listar_pendientes`
  para asignaciones) que lea la hoja de reportes y devuelva folio, unidad, tipo, estado, fecha,
  responsable, etc.
- Acción `actualizar_reporte` para cambiar estado, asignar responsable, capturar
  diagnóstico/acciones y registrar cierre (fecha + quién cierra).
- Columnas nuevas en la hoja de reportes: `estado` (Nuevo / En revisión / Asignado / En proceso /
  Resuelto / Cerrado — FL-RF-09), `responsable_atencion`, `diagnostico`, `acciones_realizadas`,
  `fecha_cierre`, `responsable_cierre`.

Con esto resuelto, salen casi gratis:

- **FL-RF-02** (historial por vehículo) — la misma lista filtrada por placa.
- **FL-RF-15** (pendientes) — la misma lista filtrada por estado ≠ Cerrado.
- **FL-RF-16** (recurrentes) — agregación simple (contar por placa + tipo) sobre esos datos.
- **FL-RF-18** (filtros) — filtros de unidad/tipo/fecha/responsable/estado sobre esa lista.
- **FL-RF-08, 09, 10, 11, 17** — campos/acciones de edición de un reporte una vez que existe la
  pantalla para verlo y tocarlo (Bloque B).

## Bloque B — Frontend: pantalla "Reportes" en el panel admin (habilita 07, 08, 09, 10, 15, 18)

Módulo nuevo tipo "Solicitudes pendientes" pero para reportes: tabla con filtros, clic en un
folio → detalle con estado, diagnóstico, acciones, responsable, botón "Cerrar".

## Bloque C — Evidencia / archivos (habilita 05, y parte de 03 y 06)

No existe `<input type="file">` en ningún formulario. Falta:

- Campo de adjuntos en el formulario de reporte/siniestro.
- El Apps Script subiendo el archivo a Google Drive (o similar) y guardando el link en la fila
  del reporte. Esto es lo más nuevo técnicamente — no existe hoy ni en frontend ni en backend.

## Bloque D — Clasificación y siniestro completos (04, 06)

- Ajustar las opciones de tipo a exactamente: Preventivo, Correctivo, Asistencia, Solicitud,
  Ayuda, Siniestro (hoy son: Falla/Problema, Siniestro, Asistencia Urgente, Mant. Preventivo,
  Mant. Correctivo, más el flujo aparte de "Pedir un vehículo").
- Para siniestro: agregar campo de **ubicación** (texto o geolocalización del navegador) y
  ligarlo al mismo mecanismo de seguimiento del Bloque A/B.

## Bloque E — Mantenimiento programado (12, 13)

Hoy "Preventivo"/"Correctivo" son solo una etiqueta dentro de un reporte reactivo. Falta una
entidad distinta: mantenimiento con **fecha futura** (unidad, tipo, fecha programada) consultable
por unidad, y que un correctivo pueda quedar ligado al folio del reporte/falla que lo originó en
vez de ser solo texto libre.

## Bloque F — Reportes operativos y exportación (14, 19)

Con la lista filtrable del Bloque A/B ya armada, un botón "Exportar" (mismo patrón que ya existe
con `vehiculos.json`, pero para el resultado filtrado de reportes) resuelve ambos casi sin
trabajo extra.

## Bloque G — Vehículos con persistencia real (01)

Alta/edición en el panel admin hoy vive solo en el navegador de quien lo usa (ver aviso en
`panel-99919507.html`) — para publicarse hay que exportar el JSON, reemplazar el archivo en el
repo y redesplegar. Para que sea un alta real sin ese paso manual, el panel necesita escribir
contra un backend: extender el Apps Script con `guardar_vehiculo` / `editar_vehiculo`, guardando
en una hoja en vez de en el `vehiculos.json` estático que sirve el sitio.

## Mapa rápido de requerimientos → bloque

| FL-RF | Bloque(s) |
|---|---|
| 01 | G |
| 02 | A |
| 03 | C, D |
| 04 | D |
| 05 | C |
| 06 | C, D |
| 07 | A, B |
| 08 | A, B |
| 09 | A, B |
| 10 | A, B |
| 11 | A, B |
| 12 | E |
| 13 | E |
| 14 | F |
| 15 | A, B |
| 16 | A |
| 17 | A, B |
| 18 | A, B |
| 19 | F |
