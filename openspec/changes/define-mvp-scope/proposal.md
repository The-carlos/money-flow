## Why

Money Flow tenía la definición repartida entre Figma, Trello y un Google Doc que se contradecían: el Figma asume datos por usuario, el Doc asume organizaciones con roles y Trello asume usuarios y contraseñas propios. Sin un alcance de MVP acordado no se puede diseñar en Figma ni tomar decisiones técnicas. Este change fija **para quién es el MVP, qué entra en cada fase y qué queda fuera**. Lo decidido salió de la sesión de exploración del 2026-09-29.

## What Changes

Decisiones confirmadas en la sesión de exploración:

- **Modelo de usuarios "B": cada quien sus propios datos, y un administrador invita.**
  - v1: el usuario principal y 2 colaboradores (3 personas).
  - v2: un grupo cerrado de *family & friends*, para mantener los costos controlados.
  - Sin registro abierto al público.
- **Los datos cuelgan de un espacio (`workspace`), no directamente del usuario.** En el MVP hay exactamente 1 espacio por usuario, creado automáticamente al aceptar la invitación, y no hay UI de espacios, miembros ni roles. Así se deja abierta la evolución a finanzas compartidas sin migrar datos.
- **Organizaciones y roles owner/admin/member/viewer (Doc) quedan fuera del MVP.** El único rol es *administrador del sistema*, que invita, lista y desactiva usuarios.
- **Se trabaja por fases, nombradas por el origen del dato:**
  - **Fase 1 — Histórico:** estados de cuenta en PDF (lo que dice el banco).
  - **Fase 2 — Captura:** registro al momento, inicialmente con un bot de Telegram (lo que registra el usuario). El nombre no se ata al canal.
- **Fase Histórico: solo tarjetas de crédito, de 5 emisores:** Rappi Card, Banamex, HSBC, Nu y BBVA. Hoy hay 1 estado de cuenta de cada uno.
- **Los PDFs nunca se almacenan:** se procesan en memoria, se extraen los datos y se descartan (sin disco ni logs). Solo se persisten los datos extraídos:
  - Movimientos.
  - Resumen del periodo.
  - Planes MSI.
  - Últimos 4 dígitos de la tarjeta.
- **Liquidez en la fase Histórico = estimada.** Los ingresos se declaran en Set up y los cargos son reales, tomados de los estados de cuenta. La UI la marca como "estimada".

Propuestas aceptadas en principio, que se confirman en design.md de su change:

- **Extracción híbrida:**
  - Un LLM extrae con salida estructurada.
  - Se valida contra los totales del propio estado de cuenta (saldo anterior, pagos, cargos, saldo nuevo).
  - Si cuadra, se guarda; si no, pasa a una pantalla de revisión manual.
- **Antes de enviar texto al LLM se eliminan datos personales** (nombre, dirección, RFC, número completo de tarjeta), y se usa un proveedor o plan sin retención de datos.
- **Detección de duplicados** por emisor + últimos 4 dígitos + fecha de corte, y/o por huella del archivo. Nunca por contenido almacenado.

Fuera del alcance del MVP (fase 1):

- Bot, tracker del ciclo, apartados y cuadre (van en la fase Captura).
- Estados de cuenta de débito, valeras y beneficios.
- Finanzas compartidas y registro abierto.

## Capabilities

### New Capabilities
Los specs se escriben en changes posteriores, una vez cerrados el Set up y la ingesta. Aquí se listan como contrato de alcance:

- `user-access`: invitación, activación y desactivación de usuarios por un administrador; inicio de sesión.
- `workspaces`: espacio personal creado automáticamente; todos los datos quedan aislados por espacio.
- `financial-setup`: alta de instrumentos (tarjetas de crédito: fecha de corte, límite) y de fuentes de ingreso (monto, periodicidad, fecha).
- `statement-ingestion`: subir un PDF de crédito, extraer, validar, revisar, guardar, detectar duplicados y descartar el PDF.
- `transaction-categorization`: asignar una categoría a cada movimiento extraído.
- `historical-insights`: dashboards de la fase Histórico (Resumen, Behavior, MSI y recurrentes, Liquidez estimada).

### Modified Capabilities
Ninguna: no existen specs previos.

## Impact

- **Figma:**
  - Hay que rediseñar Set up (separar instrumentos de ingresos).
  - Faltan la pantalla de subida y revisión de estados de cuenta y la etiqueta "estimada" en Liquidez.
  - Tracker, Movimientos y apartados se posponen a la fase Captura.
- **Trello:**
  - Las historias de CRUD de usuarios se reducen a invitar, listar y desactivar.
  - Faltan épicas para las capacidades de la fase Histórico.
- **Google Doc:** la sección de organizaciones y roles queda fuera del MVP.
- **Dependencias externas previstas:**
  - Proveedor LLM sin retención de datos.
  - Proveedor de autenticación, todavía por decidir en `platform-architecture`.
- **Datos de prueba:** PDFs reales con datos sensibles. Se guardan fuera de git (carpeta ignorada) y solo para desarrollo.

## Open Questions

Pendientes, en el orden sugerido para retomar:

1. **Instrumentos vs ingresos:** la "entidad" del Figma mezcla tarjetas (corte, límite) con fuentes de ingreso (monto regular, periodicidad). Hay que separar el modelo del Set up. Es lo siguiente a explorar.
2. **Categorización:** ¿con reglas, con LLM o híbrida? ¿Catálogo fijo o editable? ¿Se relaciona con los "apartados"?
3. **Qué dashboards y métricas entran exactamente en la fase Histórico.**
4. **Postura de privacidad para family & friends:** ¿los administradores pueden ver datos de otros usuarios? Hay que decidirlo antes de invitarlos.
5. **Datos de prueba:** conseguir 6 a 12 meses de estados de cuenta por emisor (hoy solo hay 1 por emisor) y confirmar si alguno está protegido con contraseña o escaneado.
6. **Límite de costo por espacio** (llamadas al LLM) antes de la v2 con family & friends.
7. **Decisiones técnicas** (change `platform-architecture`, después de Figma): Supabase/Postgres vs MySQL+Redis, monolito modular vs microservicios, repositorio (`etellez-workspace` vs este repo).

## Next Session

- Retomar con `/opsx:explore Set up: separar instrumentos (tarjetas) de fuentes de ingreso` (Open Question 1).
- En paralelo, fuera de sesión: reunir los PDFs de prueba (Open Question 5).
- Cuando se cierren las Open Questions 1 a 3: escribir los specs de este change (o proponer un change por capacidad) y pasar a los wireframes de Figma de la fase Histórico.
- Contexto de fuentes y hallazgos previos: `docs/CONTEXTO.md`.
