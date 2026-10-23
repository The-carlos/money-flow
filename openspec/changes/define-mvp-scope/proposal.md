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

Decisiones confirmadas en la sesión de exploración del Set up (2026-10-01). Detalle en `specs/financial-setup/spec.md`:

- **La "entidad" del Figma se separa en tres conceptos:** medios de pago, ingresos recurrentes y pagos recurrentes.
- **Medios de pago:**
  - Tipos: TDC, débito, vales y efectivo.
  - Nombre libre, varios por tipo y totalmente personalizables.
  - Solo las TDC tienen estado de cuenta y se pueden cuadrar.
- **Las TDC nacen del primer estado de cuenta:** el sistema propone emisor, últimos 4 dígitos, día de corte, día límite de pago y límite de crédito, y el usuario confirma. El alta manual queda como opción.
- **Débito y vales no requieren estado de cuenta.** Sus entradas se declaran como ingresos recurrentes que "caen en" un medio.
- **Los vales son dinero restringido:** se muestran separados de la liquidez libre.
- **Todo gasto o pago indica desde qué medio se paga.**
- **Saldo actual opcional** al crear medios de débito, vales y efectivo. "Ajustar saldo" registra la diferencia como un movimiento de ajuste.
- **La fecha de un ingreso o pago recurrente es una regla según la periodicidad:** quincenal "15 y último", mensual "día N", semanal o anual. El monto del ingreso es neto, o estimado si es variable.
- **Pagos recurrentes:**
  - Si se cobran en una TDC, son informativos y no se restan de la liquidez.
  - Si se pagan fuera de tarjeta, sí se restan.
- **"Efectivo" se crea automáticamente.** Los medios con historial se archivan, no se borran.

Decisiones confirmadas en la sesión de exploración de categorización (2026-10-23). Detalle en `specs/transaction-categorization/spec.md`:

- **Categoría y apartado son conceptos separados y ligados:**
  - La categoría describe en qué se gastó y existe desde la fase Histórico.
  - El apartado (fase Captura) es un presupuesto que agrupa una o varias categorías con un monto.
- **Catálogo base editable, de un solo nivel:**
  - Se siembra al crear el espacio y el usuario puede renombrar, agregar, archivar y fusionar.
  - "Otros" es la categoría de respaldo y no se puede archivar ni borrar.
- **El tipo de movimiento es distinto de la categoría:**
  - Tipos fijos: compra, mensualidad MSI, devolución, pago a la tarjeta, intereses, comisión y disposición de efectivo.
  - Solo compra, mensualidad MSI y devolución llevan categoría. Los demás la llevan vacía, no una categoría "NA".
  - Intereses y comisiones se reportan como costo financiero.
- **Los MSI se categorizan una vez por plan** y las mensualidades heredan la categoría. Las devoluciones heredan la de su compra cuando se pueden ligar.
- **Clasificación híbrida:**
  - Primero la memoria del espacio, después el LLM y al final la revisión del usuario.
  - La memoria siempre gana.
- **La memoria se indexa por comercio normalizado y es por espacio.**
  - Al corregir, "recordar para este comercio" viene marcado por defecto, con la opción "solo este movimiento".
  - El pasado solo cambia si el usuario lo pide.
- **La confianza se mide en tres niveles por señales observables, no con un puntaje del LLM:**
  - Alta: por regla o por el usuario.
  - Media: LLM con un comercio claro.
  - Baja: caso ambiguo, comercio multigiro, descripción genérica o "Otros".
- **Bandeja "por revisar" no bloqueante:**
  - Lo de confianza baja se guarda con la mejor propuesta y entra a la bandeja, con la segunda opción del LLM como sugerencia.
  - Es independiente de la revisión de extracción, que sí bloquea cuando los totales no cuadran.

Propuestas aceptadas en principio, que se confirman en design.md de su change:

- **Extracción híbrida:**
  - Un LLM extrae con salida estructurada.
  - Se valida contra los totales del propio estado de cuenta (saldo anterior, pagos, cargos, saldo nuevo).
  - Si cuadra, se guarda; si no, pasa a una pantalla de revisión manual.
- **Antes de enviar texto al LLM se eliminan datos personales** (nombre, dirección, RFC, número completo de tarjeta), y se usa un proveedor o plan sin retención de datos.
- **Detección de duplicados** por emisor + últimos 4 dígitos + fecha de corte, y/o por huella del archivo. Nunca por contenido almacenado.
- **La categorización va en la misma llamada al LLM que la extracción:**
  - El LLM devuelve por cada movimiento el tipo, el comercio normalizado, la categoría, si es ambiguo y una segunda opción.
  - Los movimientos con regla en la memoria no necesitan categoría del LLM.
- **El umbral de confianza se calibra con PDFs etiquetados a mano:** si el nivel media se equivoca más de ~1 de cada 10 veces, también entra a la bandeja. La lista de comercios multigiro y el umbral final se fijan en design.md.

Fuera del alcance del MVP (fase 1):

- Bot, tracker del ciclo, apartados y cuadre (van en la fase Captura).
- Estados de cuenta de débito, valeras y beneficios.
- Finanzas compartidas y registro abierto.

## Capabilities

### New Capabilities
Los specs se escriben en changes posteriores, una vez cerrados el Set up y la ingesta. Aquí se listan como contrato de alcance:

- `user-access`: invitación, activación y desactivación de usuarios por un administrador; inicio de sesión.
- `workspaces`: espacio personal creado automáticamente; todos los datos quedan aislados por espacio.
- `financial-setup`: medios de pago personalizables (TDC creadas desde el estado de cuenta o a mano; débito, vales y efectivo con saldo opcional y ajustable), ingresos recurrentes y pagos recurrentes. **Spec escrito.**
- `statement-ingestion`: subir un PDF de crédito, extraer, validar, revisar, guardar, detectar duplicados y descartar el PDF.
- `transaction-categorization`: catálogo de categorías base editable por espacio, tipo de movimiento separado de la categoría, clasificación híbrida (memoria por comercio + LLM), niveles de confianza y bandeja de movimientos por revisar. **Spec escrito.**
- `historical-insights`: dashboards de la fase Histórico (Resumen, Behavior, MSI y recurrentes, Liquidez estimada).

### Modified Capabilities
Ninguna: no existen specs previos.

## Impact

- **Figma:**
  - Hay que rediseñar Set up con tres secciones: medios de pago, ingresos y pagos recurrentes. A las TDC les falta el campo "día límite de pago".
  - Liquidez necesita una tarjeta aparte para los vales y saldos por medio.
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

1. ~~**Instrumentos vs ingresos**~~: **resuelto** el 2026-10-01 (ver `specs/financial-setup/spec.md`).
2. ~~**Categorización**~~: **resuelto** el 2026-10-23 (ver `specs/transaction-categorization/spec.md`).
3. **Qué dashboards y métricas entran exactamente en la fase Histórico.**
4. **Postura de privacidad para family & friends:** ¿los administradores pueden ver datos de otros usuarios? Hay que decidirlo antes de invitarlos.
5. **Datos de prueba:** conseguir 6 a 12 meses de estados de cuenta por emisor (hoy solo hay 1 por emisor) y confirmar si alguno está protegido con contraseña o escaneado.
6. **Límite de costo por espacio** (llamadas al LLM) antes de la v2 con family & friends.
7. **Decisiones técnicas** (change `platform-architecture`, después de Figma): Supabase/Postgres vs MySQL+Redis, monolito modular vs microservicios, repositorio (`etellez-workspace` vs este repo).
8. **Cálculo de liquidez** (sesión de dashboards):
   - ¿Se restan los cargos de la TDC en su fecha, o el pago del estado de cuenta en su fecha límite?
   - Hay que avisar en pantalla que no incluye gastos con débito o efectivo no declarados.
9. **Medio de pago en el bot** (fase Captura): ¿cómo se pide sin romper el flujo de 3 pasos? Opciones: que el LLM lo deduzca de la descripción, botones en el paso 2 o recordar el último medio usado.
10. **Onboarding:** cómo guiar el alta de medios, ingresos y pagos recurrentes sin abrumar al usuario. Incluye invitar, sin obligar, a revisar todas las categorías del primer estado de cuenta, cuando la memoria está vacía.
11. **Calibración de la confianza** (depende de la Open Question 5): etiquetar a mano las categorías de los PDFs de prueba, medir el error del nivel media y fijar la lista de comercios multigiro.
12. **Ligar devoluciones con su compra original:** ¿con qué datos se liga (comercio, monto, fechas)? Se resuelve en `statement-ingestion`.
13. **Nombres finales del catálogo base:** el spec usa un borrador de 16 categorías que se puede ajustar antes de los wireframes.

## Next Session

- Retomar con `/opsx:explore Dashboards y métricas de la fase Histórico` (Open Questions 3 y 8). La categorización ya está definida, así que Behavior puede contar con gasto por categoría y costo financiero.
- En paralelo, fuera de sesión: reunir los PDFs de prueba (Open Question 5) y etiquetar sus categorías (Open Question 11).
- Cuando se cierre la Open Question 3: escribir los specs restantes (`statement-ingestion`, `historical-insights`, `user-access`, `workspaces`) y pasar a los wireframes de Figma de la fase Histórico.
- Contexto de fuentes y hallazgos previos: `docs/CONTEXTO.md`.
