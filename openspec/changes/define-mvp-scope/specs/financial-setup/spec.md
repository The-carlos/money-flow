## Purpose

Permite a cada usuario describir su estructura financiera: los medios de pago desde donde gasta (tarjetas de crédito, cuentas de débito, vales y efectivo), sus ingresos recurrentes y sus pagos recurrentes. Estos datos alimentan la liquidez estimada y la asociación de estados de cuenta, sin exigir estados de cuenta de débito ni de vales.

## ADDED Requirements

### Requirement: Medios de pago personalizables
El sistema SHALL permitir que cada usuario administre, dentro de su espacio, una lista de medios de pago con nombre libre. Cada medio MUST tener un tipo entre: tarjeta de crédito (TDC), débito, vales o efectivo. El usuario MUST poder tener varios medios del mismo tipo.

#### Scenario: Alta de varios medios del mismo tipo
- **WHEN** el usuario crea un medio de débito "BBVA nómina" y otro de débito "Nu cuenta"
- **THEN** ambos aparecen como medios independientes en su lista de medios de pago

#### Scenario: Aislamiento por espacio
- **WHEN** un usuario consulta sus medios de pago
- **THEN** el sistema MUST mostrar únicamente los medios de su propio espacio

### Requirement: Efectivo creado automáticamente
El sistema SHALL crear un medio de tipo efectivo llamado "Efectivo" cuando se crea el espacio de un usuario. El usuario MUST poder renombrarlo o archivarlo.

#### Scenario: Espacio nuevo
- **WHEN** un usuario acepta su invitación y se crea su espacio
- **THEN** su lista de medios de pago contiene un medio "Efectivo" de tipo efectivo

### Requirement: Tarjetas de crédito creadas desde el estado de cuenta
Cuando el usuario sube un estado de cuenta de una tarjeta que aún no existe en su espacio, el sistema SHALL proponer la creación de la tarjeta con los datos extraídos del estado de cuenta: emisor, últimos 4 dígitos, día de corte, día límite de pago y límite de crédito. La tarjeta MUST crearse solo después de que el usuario confirme o corrija esos datos. Una tarjeta MUST identificarse por emisor + últimos 4 dígitos, y el sistema MUST NOT almacenar el número completo.

#### Scenario: Primer estado de cuenta de una tarjeta nueva
- **WHEN** el usuario sube un estado de cuenta de Nu con terminación 1234 y no tiene una TDC Nu 1234
- **THEN** el sistema muestra los datos detectados (emisor, últimos 4, corte, límite de pago, límite de crédito) y pide confirmación antes de crear la tarjeta

#### Scenario: Estado de cuenta de una tarjeta existente
- **WHEN** el usuario sube un estado de cuenta de una TDC cuyo emisor y últimos 4 dígitos ya existen en su espacio
- **THEN** el sistema asocia el estado de cuenta a esa tarjeta sin proponer una tarjeta nueva

#### Scenario: El usuario corrige los datos detectados
- **WHEN** el usuario edita el día de corte propuesto antes de confirmar
- **THEN** la tarjeta se crea con el valor corregido por el usuario

### Requirement: Alta manual de tarjetas de crédito
El sistema SHALL permitir crear una TDC manualmente con nombre, emisor, últimos 4 dígitos, día de corte, día límite de pago y límite de crédito, sin haber subido un estado de cuenta.

#### Scenario: Tarjeta sin estado de cuenta todavía
- **WHEN** el usuario da de alta manualmente una TDC Rappi terminación 5678
- **THEN** la tarjeta aparece en sus medios de pago y un estado de cuenta posterior de Rappi 5678 se asocia a ella

### Requirement: Vales como dinero restringido
Los medios de tipo vales SHALL tratarse como dinero restringido: su saldo MUST mostrarse separado de la liquidez libre y MUST NOT sumarse a ella.

#### Scenario: Liquidez con vales
- **WHEN** el usuario tiene $12,400 entre débito y efectivo y $3,000 en un medio de vales
- **THEN** la liquidez libre muestra $12,400 y los vales se muestran aparte como $3,000

### Requirement: Saldo actual opcional y ajustable
Al crear un medio de tipo débito, vales o efectivo, el sistema SHALL permitir capturar un saldo actual opcional. En cualquier momento el usuario MUST poder ajustar el saldo de esos medios a su valor real. Cada ajuste MUST registrarse como un movimiento de tipo "ajuste" por la diferencia, conservando el historial. Las TDC MUST NOT permitir ajuste manual de saldo, porque su saldo proviene del estado de cuenta.

#### Scenario: Medio creado sin saldo
- **WHEN** el usuario crea un medio de débito sin capturar saldo
- **THEN** el medio se crea y su saldo se considera desconocido hasta que el usuario lo capture o ajuste

#### Scenario: Ajuste de saldo
- **WHEN** el saldo estimado de "BBVA nómina" es $10,000 y el usuario lo ajusta a $9,200
- **THEN** el saldo pasa a $9,200 y se registra un ajuste de -$800 con su fecha

### Requirement: Ingresos recurrentes
El sistema SHALL permitir registrar múltiples ingresos recurrentes con: nombre, tipo (nómina, vales, beneficio, incentivo, extraordinario u otro), monto neto, indicador de monto estimado para ingresos variables, regla de fecha y el medio de pago donde se recibe ("cae en"). Un ingreso MUST NOT requerir un estado de cuenta.

#### Scenario: Varios ingresos
- **WHEN** el usuario registra "Nómina empresa A" de $18,000 que cae en "BBVA nómina" y "Freelance" estimado en $5,000 que cae en "Nu cuenta"
- **THEN** ambos ingresos se guardan y cada uno queda asociado a su medio de destino

#### Scenario: Ingreso de vales
- **WHEN** el usuario registra un ingreso de tipo vales que cae en el medio "Edenred"
- **THEN** el ingreso incrementa el saldo estimado de vales y no la liquidez libre

### Requirement: Regla de fecha según periodicidad
La fecha de un ingreso o pago recurrente SHALL expresarse como una regla según su periodicidad:
- Quincenal: dos días del mes, p. ej. 15 y último día.
- Mensual: un día del mes.
- Semanal: un día de la semana.
- Anual o extraordinario: mes y día.

Cuando el día configurado no existe en un mes, el sistema MUST usar el último día de ese mes.

#### Scenario: Quincena en febrero
- **WHEN** un ingreso es quincenal con días 15 y 30
- **THEN** en febrero el sistema lo espera los días 15 y 28 (o 29 en año bisiesto)

#### Scenario: Ingreso extraordinario anual
- **WHEN** el usuario registra "Aguinaldo" anual el 20 de diciembre
- **THEN** el sistema lo espera solo el 20 de diciembre de cada año

### Requirement: Pagos recurrentes
El sistema SHALL permitir registrar pagos recurrentes con: nombre, monto, regla de fecha, categoría y el medio de pago con el que se pagan. Un pago recurrente cobrado en una TDC MUST considerarse informativo y MUST NOT restarse de la liquidez, porque el cargo llegará en el estado de cuenta. Un pago recurrente pagado con débito, vales o efectivo MUST restarse del saldo estimado de ese medio.

#### Scenario: Recurrente fuera de tarjeta
- **WHEN** el usuario registra "Renta" de $9,000 mensual pagada con "BBVA nómina"
- **THEN** la liquidez estimada descuenta $9,000 de "BBVA nómina" en cada fecha esperada

#### Scenario: Recurrente en TDC sin doble conteo
- **WHEN** el usuario registra "Netflix" de $299 mensual pagado con la TDC Nu
- **THEN** el pago aparece en los pagos recurrentes esperados y no se descuenta de la liquidez libre

### Requirement: Archivado en lugar de borrado
Un medio de pago, ingreso o pago recurrente que ya tiene movimientos o historial asociados SHALL archivarse en lugar de borrarse. Un elemento archivado MUST desaparecer de las listas activas y de los formularios, pero su historial MUST seguir disponible en los reportes.

#### Scenario: Archivar una tarjeta con historial
- **WHEN** el usuario elimina una TDC que tiene estados de cuenta cargados
- **THEN** el sistema la archiva, deja de ofrecerla en formularios y conserva sus movimientos en los dashboards históricos

#### Scenario: Borrar un medio sin historial
- **WHEN** el usuario elimina un medio recién creado sin movimientos
- **THEN** el sistema lo borra definitivamente
