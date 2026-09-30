# Money Flow — Contexto consolidado y estado actual

> Documento fuente de verdad del proyecto. Última actualización: 2026-09-29.

## Fuentes
| Fuente | Estado | Link |
|---|---|---|
| Figma `Money_flow_x` | ✅ Leído (1 página: wireframes baja fidelidad + modelo de datos) | Pedir acceso al equipo |
| Google Doc (propuesta) | ✅ Leído (visión, arquitectura, backlog técnico) | Pedir acceso al equipo |
| Trello `Money Flow Backlog` | ✅ Leído vía API REST (credenciales en `.env.trello`, no versionar) | Pedir acceso al equipo |

---
## 1. Qué es Money Flow
App de finanzas personales para controlar el flujo de dinero con dos fuentes de datos:
- **Off-line (estados de cuenta PDF):** se cargan PDFs bancarios → *Extractor, parser & formatter* (extraer movimientos, extraer movimientos MSI, parsear, formatear) → persistencia. Debe soportar múltiples estados de cuenta y **detectar periodos duplicados**.
- **On-line (Telegram bot, "vivo"):** registro de gastos en el momento, almacenados en el "ciclo". Cambio pedido: registrar un gasto en 3 pasos → (1) monto + descripción (llama a la API de OpenAI para clasificar), (2) fecha con opciones sugeridas, (3) confirmar s/n.
- **Web App (no conversacional):** dashboards y configuración.

Idea central: **cuadre automático** entre lo registrado en el tracker (Telegram) y el estado de cuenta ("git diff" tracker vs. estado de cuenta → merge y corrección del balance).

## 2. Pantallas de la Web App (Figma)
| Pantalla | Contenido |
|---|---|
| **Login** | Usuario, contraseña, botón. Bloqueo de IP tras 3 intentos fallidos. |
| **Set up – Entidades** | Nombre; Tipo (Débito / Crédito / Valera / Beneficio / Incentivo); Periodicidad (Quincenal / Mensual / Semanal / Extraordinario); Fecha de recepción; Monto regular (no crédito); Fecha de corte MM-DD (solo crédito); Límite de crédito (solo TDC); ID auto. |
| **Set up – Apartados** | Concepto (dropdown: inversión, familia, restaurantes, salidas…), monto, listado. |
| **Liquidez** | Desglose entradas (entidades de ingreso) vs salidas (entidades de egreso) → liquidez; "monto disponible del periodo"; tarjetas de apartados; gráfica de liquidez en el tiempo (mes cerrado). |
| **Resumen** | Débito y crédito: gráfica ingresos vs egresos, gráfica compras vs pagos (barras apiladas con intereses), por mes/periodo; tablas cálculo de deuda TDC y resumen de estado de cuenta (información estática). |
| **Behavior** | KPIs: total compras, total pagos recibidos, % de límite de crédito usado; líneas apiladas de gasto diario (TDC / TDD / Valeras); barras de gasto por categoría; compras vs pagos del periodo (TDC); variación MoM de compras. |
| **Tracker** | KPIs: presupuesto del ciclo, gastado (% usado), disponible; barra de avance que cambia de color; líneas de gasto por fecha por entidad; tabla de gastos (Fecha, Monto, Descripción, Categoría, Entidad); barras por categoría apiladas por entidad. |
| **Movimientos** | Tabla estado de cuenta vs tabla tracker → diff → merge y corrección de balance. |
| **Recurrentes y MSI** | KPIs: pago MSI del mes, pagos recurrentes esperados, compromisos totales. |

## 3. Modelo de datos (borrador en Figma)
- `users (TBD)`: user_name, password, user_id
- `entities`: nombre, tipo, periodicidad, fecha_recepción, monto_regular, fecha_corte, límite_crédito, id_entity, user_id
- `movements_tracker`: id_movimiento, id_entity, amount, flow (In/Out), description, category
- `movements_bank_statement`: id_movimiento, id_entity, user_id, amount, flow, description, category
- `recurring_payments`: nombre, tipo, periodicidad, día, monto_regular, id_entity, user_id

Google Doc propone además: `profiles`, `organizations`, `organization_members`, `financial_entities`, `money_buckets`, `transactions` sobre `auth.users` de Supabase.

## 4. Arquitectura propuesta (Google Doc)
- **Supabase Auth** = identidad (login, hashing, JWT, refresh, recuperación, MFA a futuro).
- **Money Flow** = autorización de negocio (roles owner/admin/member/viewer, permisos en BD, no en claims del JWT).
- **Railway** = hosting de microservicios + API Gateway/middleware JWT.
- Microservicios: Auth/User Admin, API Gateway, capa de autorización compartida, Financial Entities, Money Buckets, Transactions, Reports.
- Backend con patrón **ports/adapters (hexagonal)**.
- 401 = no autenticado; 403 = autenticado sin permiso. Nunca confiar en `user_id` enviado por el frontend.
- Variables de entorno definidas para frontend (SUPABASE_URL, SUPABASE_ANON_KEY, MONEY_FLOW_API_URL) y servicios (SUPABASE_JWT_SECRET/JWKS, SERVICE_ROLE_KEY, DATABASE_URL, JWT_ISSUER…).

## 5. Roadmap/backlog documentado
- **Fase 1 (sem 1-2):** infraestructura + auth central (repo, interceptor JWT, pools de BD/Redis).
- **Fase 2:** "revisión de historias" (incompleta en el doc).
- **Feature A – Login:** alta de usuarios por admin, middleware, bloqueo por intentos/IP, sesiones (por investigar).
- **Feature B – Set up:** formularios de entidades y apartados + persistencia.
- Backlog técnico: configurar Supabase, pantalla de login, primer microservicio en Railway, middleware JWT, tablas profiles/orgs/members, onboarding, reglas de roles, proteger endpoints, logout y recuperación de contraseña.

## 5b. Backlog en Trello (leído 2026-09-29)
Flujo del tablero: **Backlog → Ready → In progress → Code review → Testing → Done**.
Etiquetas: *Infraestructura base*, *Tarea técnica*, *Historia de usuario*, *Administración de usuarios*, *Spike*.

| Lista | Tarjeta | Épica / tipo | Estado del contenido |
|---|---|---|---|
| Backlog | Definir pipeline en GitHub CI | Infra base · técnica | Solo título (TBD) |
| Backlog | Framework – IA | Infra base · técnica | Solo título (TBD) |
| Backlog | Definir arquitectura de microservicios | Infra base · técnica · **Alta** | Completa (CA + tareas) |
| Backlog | Inicializar repositorio (`etellez-workspace`) | Infra base · técnica · **Alta** | Completa |
| Backlog | Crear usuarios | Admin. usuarios · historia · **Alta** | Completa |
| Backlog | Consultar usuarios | Admin. usuarios · historia · **Alta** | Completa |
| Backlog | Actualizar usuario | Admin. usuarios · historia · **Alta** | Completa |
| Backlog | Eliminar usuario | Admin. usuarios · historia · **Alta** | Completa |
| Ready | Spike: Manejo de BD para gestión de usuarios | Admin. usuarios · spike | Solo título |

Puntos clave de las tarjetas:
- **Arquitectura de microservicios:** decidir monorepo vs multi-repo, estructura por servicio, patrón hexagonal (puertos, adaptadores, controladores, servicios, acceso a BD), cómo se exponen endpoints, manejo de env vars e **integración de MySQL, Redis y JWT**. Entregable: documento de arquitectura en el repo. Comentario (2026-06-11): se propone llevarla de forma asíncrona en el Google Doc.
- **Inicializar repositorio:** `etellez-workspace` con carpetas frontend/backend/docs/infra, README, `.gitignore`, ramas `main`/`develop`/feature. Comentario (2026-06-11): posiblemente ya esté hecho — **confirmar**.
- **CRUD de usuarios (admin):** crear (validar duplicados, hash de contraseña), listar/detalle (sin campos sensibles, estados de carga/vacío/error), actualizar (incluye estado activo/inactivo), eliminar/desactivar (confirmación, decidir borrado físico vs lógico, inactivos no pueden autenticarse, evitar auto-borrado). Todas con pruebas básicas.

Observaciones:
- El backlog **solo cubre infraestructura y administración de usuarios**; no hay tarjetas para Set up, Tracker, bot de Telegram, parser de PDFs, dashboards ni cuadre.
- Las historias asumen **usuarios y contraseñas propios** (hash en BD propia), lo que choca con delegar identidad a Supabase Auth (ver sección 7).
- No hay fechas, responsables ni estimaciones asignadas.

## 6. Estado actual
- Sin código; diseño en fase de wireframe (cajas + texto, sin UI de alta fidelidad ni design system).
- Arquitectura de auth bastante definida; dominio financiero (PDF parser, bot, cuadre) solo a nivel concepto.
- Trello: 8 tarjetas en *Backlog*, 1 spike en *Ready*; *In progress / Code review / Testing / Done* vacías. Nada iniciado formalmente (salvo quizá el repo, ver 5b).

## 7. Inconsistencias / decisiones abiertas detectadas
1. **Stack de BD en conflicto:** Fase 1 del Doc y la tarjeta de arquitectura en Trello mencionan MySQL (HostGator) + Upstash Redis; la decisión de arquitectura del Doc usa Supabase (Postgres) + Railway.
2. **Gestión de usuarios/contraseñas:** la tabla `users` en Figma (password Int) y el CRUD de usuarios en Trello (hash propio) chocan con delegar auth a Supabase. Si se usa Supabase, el CRUD se reduce a invitar/desactivar vía Admin API + tabla `profiles`.
3. **Multi-tenant:** Figma asume `user_id` por registro; el Doc introduce organizaciones y roles. ¿Es app personal o multi-usuario/familia?
4. **Microservicios vs MVP:** 7 servicios parece sobredimensionado para un MVP; valorar monolito modular hexagonal y separar después.
5. **Modelo de datos:** falta fecha en movimientos, `user_id` en tracker, MSI (meses, mensualidad), ciclos/periodos, categorías como catálogo, `statement_id` para detectar duplicados; montos deben ser `DECIMAL`, no Float; typo `id_entitiy`.
6. **Telegram bot + OpenAI + parser de PDFs** no aparecen en el Doc ni en Trello (solo existe "Framework – IA" sin definir).
7. **Repo `etellez-workspace`:** no se sabe si ya existe; confirmar antes de crear otro.
8. "Fecha de corte" aparece como Dropdown en la tabla y como MM-DD en el formulario.

## 8. Próximos pasos sugeridos (sin código)
1. Resolver decisiones abiertas (sección 7), sobre todo stack de BD y alcance multi-tenant.
2. Definir alcance del MVP (propuesta: Login + Set up + Tracker vía Telegram + Resumen; PDF parser y cuadre en fase 2).
3. Cerrar modelo de datos v1 (ERD).
4. Wireframes de media fidelidad en Figma para las pantallas del MVP.
5. Completar Trello: épicas y tarjetas para Set up, Tracker/Telegram, parser PDF, dashboards y cuadre; detallar las tarjetas TBD.


---

## Referencias Figma (node IDs)
- Descripción general / flujo off-line–on-line: `6:2`, `2:2`, `2:3`, `1:6713`
- Pipeline PDF (extractor/parser/formatter): `8:24`–`8:44`
- Web App (lista de módulos): `8:62`
- Login: `61:40` · Set up: `45:3` · Liquidez: `17:46` · Resumen: `12:63` · Behavior: `17:48` · Tracker: `23:75` · Movimientos: `17:44` · Recurrentes y MSI: `23:73`
- Tablas del modelo: `entities` `63:577`, `movements_tracker` `63:688`, `movements_bank_statement` `63:803`, `recurring_payments` `209:175`, `Users (TBD)` `80:150`
