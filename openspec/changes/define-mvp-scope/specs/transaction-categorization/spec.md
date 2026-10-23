## Purpose

Permite asignar a cada gasto una categoría que describe en qué se gastó, usando un catálogo editable por espacio y una clasificación híbrida que aprende de las correcciones del usuario. Las categorías alimentan los dashboards de la fase Histórico y, en la fase Captura, los apartados y el bot.

## ADDED Requirements

### Requirement: Catálogo base editable
El sistema SHALL sembrar un catálogo base de categorías al crear el espacio de un usuario. El catálogo MUST tener un solo nivel, sin subcategorías. El catálogo base inicial MUST contener: Supermercado, Restaurantes, Comida a domicilio, Transporte, Vivienda, Servicios, Suscripciones, Salud, Educación, Ropa y cuidado personal, Entretenimiento, Viajes, Regalos y familia, Mascotas, Compras en línea y Otros. El usuario MUST poder renombrar categorías y agregar las suyas. Cada espacio MUST tener su propio catálogo, aislado del de otros espacios.

#### Scenario: Espacio nuevo
- **WHEN** un usuario acepta su invitación y se crea su espacio
- **THEN** su catálogo contiene las categorías del catálogo base

#### Scenario: Categoría propia
- **WHEN** el usuario agrega la categoría "Gimnasio"
- **THEN** "Gimnasio" aparece en su catálogo, en los formularios y entre las opciones que se usan para clasificar sus movimientos

#### Scenario: Renombrar sin perder historial
- **WHEN** el usuario renombra "Supermercado" a "Despensa"
- **THEN** todos los movimientos, reglas y pagos recurrentes que tenían "Supermercado" muestran "Despensa"

### Requirement: Categoría "Otros" protegida
La categoría "Otros" SHALL existir siempre en cada espacio. El usuario MUST poder renombrarla, pero MUST NOT poder archivarla, borrarla ni fusionarla con otra.

#### Scenario: Intento de archivar "Otros"
- **WHEN** el usuario intenta archivar la categoría "Otros"
- **THEN** el sistema rechaza la acción e indica que es la categoría de respaldo

### Requirement: Tipo de movimiento separado de la categoría
Cada movimiento SHALL tener un tipo, asignado por el sistema y no editable como categoría. Los tipos de movimiento de un estado de cuenta de TDC son: compra, mensualidad MSI, devolución, pago a la tarjeta, intereses (incluye su IVA), comisión y disposición de efectivo. Los movimientos de tipo compra, mensualidad MSI y devolución MUST tener categoría. Los movimientos de tipo pago a la tarjeta, intereses, comisión y disposición de efectivo MUST NOT tener categoría. Intereses y comisiones MUST reportarse como costo financiero, separado del gasto por categoría.

#### Scenario: Pago a la tarjeta
- **WHEN** un estado de cuenta incluye un pago de $8,000 a la tarjeta
- **THEN** el movimiento se guarda con tipo pago a la tarjeta, sin categoría, y no se suma al gasto por categoría

#### Scenario: Intereses
- **WHEN** un estado de cuenta incluye $1,070 de intereses y $170 de IVA de intereses
- **THEN** ambos se guardan sin categoría y el periodo reporta $1,240 de costo financiero

#### Scenario: Compra sin categoría
- **WHEN** se intenta guardar un movimiento de tipo compra sin categoría
- **THEN** el sistema MUST rechazar el guardado

### Requirement: Categoría de planes MSI
La categoría de una compra a meses sin intereses SHALL asignarse una sola vez a su plan MSI, y cada mensualidad del plan MUST tener la categoría del plan. Si el usuario cambia la categoría del plan, MUST cambiar la de todas sus mensualidades.

#### Scenario: Mensualidad de un plan ya categorizado
- **WHEN** llega la mensualidad 4 de 12 de un plan MSI categorizado como "Viajes"
- **THEN** la mensualidad se guarda con categoría "Viajes" sin volver a clasificarse

#### Scenario: Recategorizar un plan
- **WHEN** el usuario cambia la categoría del plan de "Viajes" a "Entretenimiento"
- **THEN** todas las mensualidades del plan, pasadas y futuras, quedan en "Entretenimiento"

### Requirement: Categoría de devoluciones
Una devolución o bonificación SHALL heredar la categoría de la compra original cuando el sistema pueda ligarla a ella. Si no puede ligarla, la devolución MUST clasificarse como cualquier otro movimiento. Una devolución MUST restar del gasto de su categoría.

#### Scenario: Devolución ligada
- **WHEN** llega una devolución de $1,200 de Liverpool ligada a una compra categorizada como "Ropa y cuidado personal"
- **THEN** la devolución toma esa categoría y el gasto de "Ropa y cuidado personal" del periodo baja $1,200

### Requirement: Clasificación híbrida
El sistema SHALL clasificar cada movimiento que requiere categoría en este orden:
1. La memoria del espacio: si existe una regla para el comercio, se usa su categoría.
2. Si no hay regla, un modelo de lenguaje propone una categoría del catálogo activo del espacio, indica si el caso es ambiguo y da una segunda opción.
3. El usuario puede revisar y corregir el resultado.

Una regla de la memoria MUST prevalecer siempre sobre la propuesta del modelo de lenguaje. El modelo de lenguaje MUST elegir solo entre categorías activas del espacio.

#### Scenario: Comercio con regla
- **WHEN** el espacio tiene la regla "Uber → Transporte" y llega un movimiento de Uber
- **THEN** el movimiento se clasifica como "Transporte" sin consultar al modelo de lenguaje

#### Scenario: Comercio sin regla
- **WHEN** llega un movimiento de un comercio que no tiene regla en el espacio
- **THEN** el modelo de lenguaje propone una categoría del catálogo activo del espacio

### Requirement: Comercio normalizado
Cada movimiento que requiere categoría SHALL tener un comercio normalizado, derivado de su descripción cruda, que elimina sucursales, folios, dominios y prefijos de agregadores de pago. El comercio normalizado MUST ser la llave de la memoria de clasificación. Las reglas de la memoria MUST pertenecer a un espacio y MUST NOT aplicarse en otros espacios.

#### Scenario: Normalización de descripciones
- **WHEN** llegan los movimientos "UBER *TRIP HELP.UBER.COM" y "MERPAGO*TACOSELGUERO"
- **THEN** sus comercios normalizados son "Uber" y "Tacos El Guero"

#### Scenario: Comercios distintos de una misma marca
- **WHEN** llegan los movimientos "UBER *TRIP" y "UBER* EATS"
- **THEN** sus comercios normalizados son distintos ("Uber" y "Uber Eats") y pueden tener reglas distintas

#### Scenario: Aislamiento de la memoria
- **WHEN** un usuario crea la regla "OXXO → Supermercado"
- **THEN** los movimientos de OXXO de otros espacios no se ven afectados

### Requirement: Corrección y aprendizaje
Cuando el usuario cambia la categoría de un movimiento, el sistema SHALL ofrecer recordar la corrección para el comercio, con esa opción seleccionada por defecto, o aplicarla solo a ese movimiento. Si se recuerda, el sistema MUST crear o actualizar la regla del comercio en la memoria del espacio. Los movimientos anteriores del mismo comercio MUST NOT cambiar a menos que el usuario lo pida explícitamente. El sistema MUST indicar cuántos movimientos anteriores se verían afectados.

#### Scenario: Recordar una corrección
- **WHEN** el usuario cambia un movimiento de Uber Eats de "Restaurantes" a "Comida a domicilio" y deja marcada la opción de recordar
- **THEN** se crea la regla "Uber Eats → Comida a domicilio" y los próximos movimientos de Uber Eats se clasifican así

#### Scenario: Corrección de un solo movimiento
- **WHEN** el usuario cambia una compra de Amazon a "Mascotas" y elige "solo este movimiento"
- **THEN** solo ese movimiento cambia y no se crea ni modifica ninguna regla para Amazon

#### Scenario: Aplicar al pasado
- **WHEN** el usuario recuerda una corrección para Uber Eats y existen 7 movimientos anteriores de Uber Eats
- **THEN** el sistema ofrece aplicar la corrección a esos 7 movimientos y solo los cambia si el usuario lo confirma

### Requirement: Nivel de confianza
Cada categoría asignada SHALL tener un nivel de confianza calculado a partir de señales observables, no de un puntaje que el modelo reporte sobre sí mismo:
- **Alta:** la categoría vino de una regla de la memoria del espacio o la asignó el usuario.
- **Media:** la propuso el modelo de lenguaje y no se cumple ninguna condición de confianza baja.
- **Baja:** la propuso el modelo de lenguaje y se cumple al menos una de estas condiciones: el modelo marcó el caso como ambiguo, el comercio es multigiro conocido, la descripción es genérica o ilegible, o la categoría propuesta es "Otros".

#### Scenario: Comercio multigiro
- **WHEN** el modelo de lenguaje clasifica una compra de Amazon como "Compras en línea"
- **THEN** la categoría se guarda con confianza baja

#### Scenario: Descripción genérica
- **WHEN** llega un movimiento con descripción "COMPRA POS 0012"
- **THEN** la categoría propuesta se guarda con confianza baja

#### Scenario: Categoría por regla
- **WHEN** un movimiento se clasifica con una regla de la memoria
- **THEN** su confianza es alta y se marca como clasificado "por tu regla"

### Requirement: Bandeja de movimientos por revisar
Los movimientos con categoría de confianza baja SHALL guardarse con la categoría propuesta y aparecer en una bandeja de movimientos por revisar. La bandeja MUST NOT bloquear el guardado del estado de cuenta ni los dashboards. Cada elemento de la bandeja MUST mostrar la segunda opción del modelo de lenguaje como sugerencia. Confirmar o corregir un movimiento MUST sacarlo de la bandeja y aplicar las reglas de corrección y aprendizaje.

#### Scenario: Estado de cuenta que cuadra con categorías dudosas
- **WHEN** un estado de cuenta cuadra con sus totales y 4 de sus 38 compras tienen confianza baja
- **THEN** el estado de cuenta se guarda, los dashboards ya incluyen las 38 compras y se muestra el aviso "4 movimientos por revisar"

#### Scenario: Confirmar la propuesta
- **WHEN** el usuario confirma la categoría propuesta de un movimiento en la bandeja
- **THEN** el movimiento sale de la bandeja con confianza alta

### Requirement: Ciclo de vida de una categoría
Una categoría sin movimientos, reglas ni pagos recurrentes asociados SHALL poder borrarse definitivamente. Una categoría con historial MUST archivarse o fusionarse en otra, no borrarse.
- **Archivar:** la categoría deja de ofrecerse en formularios y al modelo de lenguaje, los movimientos anteriores la conservan y las reglas que apuntaban a ella se desactivan.
- **Fusionar:** los movimientos, reglas, pagos recurrentes y apartados que usaban la categoría pasan a la categoría destino y la categoría original desaparece. Antes de fusionar, el sistema MUST mostrar cuántos movimientos se moverán y pedir confirmación explícita.

#### Scenario: Archivar una categoría usada
- **WHEN** el usuario archiva "Comida a domicilio", que tiene movimientos y la regla "Uber Eats → Comida a domicilio"
- **THEN** los movimientos anteriores conservan "Comida a domicilio", la regla se desactiva y el próximo movimiento de Uber Eats se clasifica con el modelo de lenguaje

#### Scenario: Fusionar categorías
- **WHEN** el usuario fusiona "Comida a domicilio" en "Restaurantes" y confirma el aviso de 23 movimientos afectados
- **THEN** los 23 movimientos y la regla de Uber Eats pasan a "Restaurantes" y "Comida a domicilio" desaparece del catálogo

#### Scenario: Borrar una categoría sin uso
- **WHEN** el usuario borra una categoría que creó y nunca usó
- **THEN** el sistema la borra definitivamente

### Requirement: Catálogo compartido
El mismo catálogo de categorías del espacio SHALL usarse para los movimientos de estados de cuenta y para los pagos recurrentes. Los movimientos que se registren en la fase Captura MUST usar el mismo catálogo.

#### Scenario: Pago recurrente con categoría
- **WHEN** el usuario registra el pago recurrente "Netflix"
- **THEN** solo puede elegir su categoría entre las categorías activas de su espacio
