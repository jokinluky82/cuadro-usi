# Cuadro USI · notas para trabajar en esta app

Estas notas son para Claude (o quien mantenga la app) en futuras conversaciones.
El dueño y administrador es Jokin (GitHub `jokinluky82`). Habla en español, sin tecnicismos,
y quiere instrucciones de una en una, diciendo exactamente dónde pulsar en su pantalla.

## Qué es
App web instalable (PWA) de turnos del equipo USI (16 personas): cuadro anual de 16 semanas,
quién trabaja cada día, peticiones/cambios/intercambios de turno, vacaciones, bajas, horas
semanales y anuales, fines de semana trabajados.

- Web: https://jokinluky82.github.io/cuadro-usi/ (GitHub Pages, rama `main`, raíz).
- Datos: Firebase, proyecto `cuadro-usis` (plan gratuito Spark), Firestore en `europe-southwest1`.
- Entrada: sesión anónima de Firebase + elegir nombre + PIN de 4 cifras.

## Archivos
- `index.html`: toda la app (HTML, CSS y JS en un archivo). El cuadro de turnos está en la
  constante `PATTERNS` (clave `u27` = cuadro 2027 de 16 semanas). Ajustes por defecto en `DEFAULT_CFG`.
- `firestore.rules`: reglas de seguridad. **No se publican solas**: hay que pegarlas en
  Firebase → Firestore → Reglas → (clic en el código) Ctrl+A, Ctrl+V → Publicar.
- `setup.html`: carga inicial de datos desde un JSON (solo funciona si no existe `config/main`).
- `sw.js`: service worker (red primero para `index.html`). Sube `CACHE` si cambian iconos o manifiesto.
- `manifest.webmanifest`, `icons/`: instalación en el móvil.
- No subir datos personales (nombres, PINs, fechas) al repositorio: es público.

## Modelo de datos (Firestore)
- `people/{id}`: `{name, startWeek, admin?}` (Jokin tiene `admin: true`).
- `pins/{personId}`: `{pin}`; solo el administrador los lee/escribe.
- `sessions/{uid}`: `{personId, pin, at}`; se crea solo si el PIN coincide (lo comprueban las reglas).
- `config/main`: ajustes (cuadro, anchor `2027-01-11`, startDate `2027-01-11`, vacWeek `36:04`,
  bajaHours, adminPerson, adminPinSet).
- `vacations/{id}`: `{personId, from, to, type: "vac"|"baja"}`.
- `requests/{id}`: peticiones de día libre `{personId, date, note, status: abierta|propuesta|cubierta, coverId, swapDate}`.
- `overrides/{personId_fecha}`: cambio de turno de un día `{personId, date, k, code, start, end, hours, fromReq?}`.

## Reglas de negocio acordadas
- El cuadro arranca y se aplica desde el lunes 11/01/2027 (confirmado); cada uno está esa semana
  en su semana de inicio. Días anteriores (1–10 de enero): gris, sin turno, no editables.
- Vacaciones: no suman horas trabajadas. En la columna semanal se muestra la cuenta con tope
  36:04 por semana (5:09 por día de vacaciones) y en rojo lo que se pierde ("palma").
  Nunca se ganan horas. Arriba: horas trabajadas y horas palmadas en vacaciones (sin comparar con jornada).
- Baja: por defecto computa las horas del turno que tocaba.
- Fin de semana trabajado: viernes de noche/tarde-noche (N1, N2, TN3) o sábado/domingo con turno.
  Los de reserva (RES) se cuentan aparte.
- TN3 se considera noche.

## Cómo publicar cambios
1. Editar `index.html` (y lo que toque), probar.
2. `git commit` y `git push` a `main`. GitHub Pages publica en 1–2 minutos.
3. Si cambian las reglas, pedir a Jokin que las pegue en Firebase (ver arriba), con instrucciones paso a paso.

## Pendiente / ideas
- Avisos al móvil (necesita plan Blaze de Firebase con tarjeta, gasto previsto 0 €).
- Festivos del año.
- Turnos reales del 1 al 12 de enero de 2027 de cada uno (vienen del calendario 2026).
