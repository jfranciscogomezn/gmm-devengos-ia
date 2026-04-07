# Requerimientos GMM-Devengos — v3

> Dominio: Nómina colombiana · Versión: v3 · Abril 2026  
> Actualizaciones respecto a v2: filtro por rango de fechas libre, registros INCOMPLETOS, corrección directa de registros por administrador, creación de registros por administrador, bloqueo de reportes por incompletos, resaltado de horas extendidas, turno nocturno con cruce de medianoche, requerimientos no funcionales de acceso multiplataforma (navegador web + aplicación móvil).

---

## 1. Módulo de seguridad

**1.1** La contraseña se guarda cifrada con bcrypt en la base de datos junto con la información del usuario. Las credenciales de acceso son correo electrónico y contraseña.

**1.2** Características de calidad de la contraseña: mínimo 8 caracteres, al menos 1 mayúscula, 1 minúscula, 1 dígito y 1 carácter especial.

**1.3** El usuario solo puede acceder a las funcionalidades que tenga el rol asociado. El menú de navegación se construye dinámicamente en función de las opciones asignadas al rol del usuario.

**1.4** Solo los usuarios habilitados podrán ingresar al sistema con su correo electrónico y contraseña. Los usuarios deshabilitados son rechazados en el login.

**1.5** El registro de usuarios debe ser realizado exclusivamente por un administrador. Al registrar un usuario se definen sus datos personales (nombres, apellidos, correo electrónico, celular), una contraseña inicial y el rol asignado. El correo electrónico debe ser único en el sistema y actúa como identificador de acceso.

**1.6** El usuario puede cambiar su propia contraseña después del primer ingreso. El administrador puede resetear la contraseña de cualquier usuario. Los cambios de contraseña quedan registrados en auditoría sin almacenar el hash.

**1.7** Un administrador puede actualizar su propio perfil de usuario (datos personales). Esta acción queda registrada en auditoría igual que cualquier otra actualización de perfil.

**1.8** Un usuario puede consultar su propio perfil (datos personales, rol) en modo solo lectura. Para modificar sus datos debe solicitar al administrador.

**1.9** La eliminación de un usuario está bloqueada si tiene registros asociados (registros de tiempo, perfil de empleado, entradas de auditoría). Si no tiene registros asociados, el administrador puede eliminarlo.

**1.10** El sistema permite definir y gestionar roles de usuario. Un usuario solo puede tener un rol a la vez.

**1.11** A cada rol se le asignan opciones de menú. El menú visible para cada usuario se genera en función de las opciones asignadas a su rol.

**1.12** Los usuarios pueden ser configurados adicionalmente como empleados (trabajadores) por un administrador. Para poder registrar horas de inicio y fin de jornada, el usuario debe estar configurado como empleado y tener sesión activa.

---

## 2. Módulo de configuración

### 2.1 Parámetros globales de nómina

La configuración es por año calendario. El administrador puede crearla o actualizarla para cualquier año.

**2.1.1** Cantidad de horas de trabajo al mes.  
**2.1.2** Cantidad de horas diarias en jornada normal semanal.  
**2.1.3** Cantidad máxima de horas extra diarias.

#### Franjas horarias (formato HH:MM)

**2.1.4** Hora inicio diurno.  
**2.1.5** Hora fin diurno.  
**2.1.6** Hora inicio extra diurna.  
**2.1.7** Hora fin extra diurna.  
**2.1.8** Hora inicio recargo nocturno (por defecto 21:00 según CST Art. 160).  
**2.1.9** Hora fin recargo nocturno (por defecto 06:00 según CST Art. 160).  
**2.1.10** Hora inicio extra nocturna.  
**2.1.11** Hora fin extra nocturna.  
**2.1.12** Hora inicio extra domingo.  
**2.1.13** Hora fin extra domingo.

#### Factores de recargo

**2.1.14** Factor hora extra diurna (mínimo legal 1,25).  
**2.1.15** Factor hora extra nocturna (mínimo legal 1,75).  
**2.1.16** Factor recargo nocturno (mínimo legal 1,35).  
**2.1.17** Factor extra diurna domingos y festivos (mínimo legal 2,00).  
**2.1.18** Factor extra nocturna domingos y festivos (mínimo legal 2,50).  
**2.1.19** Factor recargo normal domingos y festivos (mínimo legal 1,75).

#### Otros parámetros

**2.1.20** Listado de domingos y festivos por año (gestión individual por fecha: agregar / eliminar).  
**2.1.21** Cantidad de minutos de descanso no computables por jornada (se descuentan del total antes de clasificar las horas).  
**2.1.22** Salario mínimo mensual del año.  
**2.1.23** Auxilio de transporte mensual del año.

### 2.2 Configuración por empleado

**2.2.1** Salario mensual del trabajador.  
**2.2.2** Tipo y número de documento de identificación del empleado (para efectos de nómina).  
**2.2.3** Un empleado debe estar previamente registrado como usuario en el sistema. Los datos personales (nombres, apellidos, correo, celular) provienen del registro de usuario.  
**2.2.4** Un usuario puede o no ser empleado. Solo los usuarios configurados como empleados usan el módulo de registro de tiempo.

---

## 3. Módulo de registro de tiempo de trabajo

### 3.1 Marcación de entrada y salida

**3.1.1** Los empleados con sesión activa pueden marcar el inicio y fin de su jornada laboral una sola vez por día. El sistema rechaza (HTTP 409) cualquier intento de segunda marcación de entrada o salida en el mismo día calendario.

**3.1.2** La hora de entrada y la hora de salida las toma el sistema en el momento en que el empleado realiza la marcación; el empleado no ingresa los tiempos manualmente.

**3.1.3** Una vez que el empleado registra la salida, el registro queda inmutable para el empleado. Para corregirlo se requiere intervención del administrador (ver sección 3.3).

**3.1.4** El sistema soporta turnos nocturnos que cruzan la medianoche: un único registro almacena la entrada de un día y la salida del día siguiente. El devengado se divide en medianoche: las horas previas usan el tipo de día de la entrada; las horas posteriores usan el tipo de día de la salida. La duración máxima de un turno es de 24 horas; un turno de mayor duración se marca inválido y genera notificación al administrador. En los reportes, el turno nocturno aparece con la fecha de entrada como referencia y un indicador de cruce de medianoche.

**3.1.5** El empleado no puede registrar entrada si tiene una marcación activa sin salida del mismo día.

### 3.2 Registros INCOMPLETOS (olvido de marcación de salida)

**3.2.1** A las 00:01 del día siguiente, un proceso automático del servidor detecta todos los registros en estado ABIERTO de días anteriores y los transiciona al estado **INCOMPLETO**. Un registro INCOMPLETO indica que el empleado nunca registró su salida ese día.

**3.2.2** El estado INCOMPLETO **no bloquea** la asistencia del empleado. El empleado puede continuar marcando entrada y salida en los días siguientes con total normalidad.

**3.2.3** El panel del empleado muestra un aviso persistente con la lista de fechas con registros INCOMPLETOS, indicándole que contacte al administrador para la resolución. El aviso desaparece automáticamente cuando todos sus registros INCOMPLETOS han sido resueltos.

**3.2.4** En el momento en que se detectan registros INCOMPLETOS, el sistema envía notificaciones en la aplicación y por correo electrónico a todos los administradores con la lista de empleados y fechas afectadas.

**3.2.5** Los registros INCOMPLETOS quedan excluidos de todos los cálculos de devengado y de la generación de reportes hasta que el administrador los resuelva (ver 3.3.4).

**3.2.6** Solo el administrador puede resolver un registro INCOMPLETO. El empleado recibe HTTP 403 si intenta cerrarlo directamente.

### 3.3 Corrección directa de registros por administrador

**3.3.1** El administrador puede sobrescribir la hora de entrada y/o la hora de salida de cualquier registro de tiempo de cualquier empleado, en cualquier fecha y en cualquier estado (ABIERTO, CERRADO o INCOMPLETO). Se debe proporcionar un **motivo de corrección obligatorio**; si está vacío el sistema retorna HTTP 400 sin modificar nada.

**3.3.2** Validaciones de la corrección:
- Si la hora de salida es anterior o igual a la hora de entrada en el mismo día calendario, el sistema retorna HTTP 400.
- Los turnos nocturnos válidos (salida al día siguiente) son aceptados.
- Se debe proporcionar al menos uno de los dos campos (entrada o salida).

**3.3.3** Antes de aplicar cualquier corrección, el sistema almacena los valores originales. El registro recibe el indicador **corregido = verdadero**. El estado resultante del registro es:
- Si se proporcionan entrada y salida: CERRADO (corregido).
- Si el registro era INCOMPLETO y se agrega la salida: CERRADO (corregido).
- Si solo se corrige la entrada de un registro CERRADO: CERRADO (corregido).

**3.3.4** Si no existe ningún registro para el empleado y la fecha indicada, el administrador puede crear uno completo proporcionando hora de entrada, hora de salida y motivo. El registro resultante tiene estado CERRADO (creado por admin).

**3.3.5** Después de cualquier corrección o creación, el sistema recalcula automáticamente el devengado del registro usando la configuración de nómina activa en esa fecha.

**3.3.6** Toda corrección genera una entrada de auditoría que incluye: ID y nombre del administrador, fecha y hora de la corrección, empleado y fecha afectados, valores originales de entrada/salida, nuevos valores de entrada/salida y motivo de corrección.

**3.3.7** Solo los usuarios con rol de administrador pueden realizar correcciones. Otros roles reciben HTTP 403.

**3.3.8** Los registros corregidos muestran un indicador visual de corrección en los reportes en pantalla. Al pasar el cursor sobre el indicador (o al tapearlo) se muestra el motivo y el nombre del administrador que realizó la corrección.

### 3.4 Reapertura de registros cerrados

**3.4.1** El administrador puede reabrir un registro CERRADO para que el empleado pueda corregir su marcación de salida, o para intervenir directamente mediante la corrección de la sección 3.3. La reapertura queda registrada en auditoría.

---

## 4. Módulo de cálculo de devengado

**4.1** La tarifa por hora del empleado se calcula como: salario mensual ÷ horas de trabajo mensuales configuradas.

**4.2** Antes de clasificar las horas, se descuentan los minutos de descanso no computables configurados para la jornada.

**4.3** Las horas facturables se clasifican en franjas según la configuración activa para el año del registro:
- Horas normales diurnas.
- Horas extra diurnas.
- Horas con recargo nocturno.
- Horas extra nocturnas.

**4.4** Cuando dos recargos aplican simultáneamente (por ejemplo, hora extra nocturna en domingo), se combinan ambos factores según las reglas del Código Sustantivo del Trabajo colombiano.

**4.5** Si la jornada cae en domingo o en un festivo del calendario configurado, se aplica el factor de recargo de domingo/festivo sobre todas las horas trabajadas ese día.

**4.6** El **Reporte A** aplica el tope diario de **8 horas normales + 2 horas extra diurnas** y el tope semanal de **12 horas extra** (CST Art. 162). Las horas que excedan el tope no se incluyen en el cálculo de devengado del Reporte A.

**4.7** El **Reporte B** corresponde a los datos reales registrados sin ningún tope aplicado.

**4.8** Los registros INCOMPLETOS se excluyen de todos los cálculos hasta que sean resueltos por el administrador.

---

## 5. Módulo de reportes

### 5.1 Reportes disponibles

**5.1.1 Reporte A en pantalla — empleado (vista limitada):** el empleado ve sus propios registros con el tope de 8 h normales + 2 h extra diurna aplicado. Las horas normales y las extra son visualmente distinguibles. Se muestra el indicador de tope semanal cuando se alcanzan 12 h/semana. Incluye fila de totales.

**5.1.2 Reporte A en Excel — empleado:** descarga en formato `.xlsx` de los propios registros del empleado con el tope aplicado. Una fila por día: fecha, entrada, salida, horas normales limitadas, horas extra limitadas, devengado. Encabezado con nombre e ID del empleado. Fila de totales al final.

**5.1.3 Reporte B en pantalla — empleado (datos reales):** el empleado ve sus propios registros sin ningún tope. Se muestra el desglose de horas por categoría (normal, extra, nocturno, etc.) y el devengado real.

**5.1.4 Reporte A en pantalla — administrador:** el administrador ve los registros de cualquier empleado con el tope de 8 h + 2 h extra diurna. Mismas columnas y comportamiento que 5.1.1.

**5.1.5 Reporte A en Excel — administrador:** descarga en `.xlsx` para cualquier empleado con el tope aplicado. Misma estructura que 5.1.2.

**5.1.6 Reporte B en pantalla — administrador:** el administrador ve los registros totales reales de cualquier empleado sin tope. Incluye desglose por categoría y registros nocturnos etiquetados.

**5.1.7 Reporte B en Excel — administrador:** descarga en `.xlsx` para cualquier empleado con horas y devengado reales sin tope. Fila de totales y registros nocturnos marcados.

### 5.2 Filtros de período

Todos los reportes (en pantalla y Excel, Reporte A y B) soportan los siguientes modos de filtro. Exactamente **un modo de filtro** debe estar activo por solicitud; enviar más de uno retorna HTTP 400.

**5.2.1** Día específico (una sola fecha).  
**5.2.2** Semana calendario (todos los días de la semana indicada).  
**5.2.3** Mes calendario (todos los días del mes indicado).  
**5.2.4** Rango de fechas libre: `fechaInicio` y `fechaFin` en formato ISO-8601 (YYYY-MM-DD). Permite generar reportes para períodos arbitrarios como quincenas o cualquier intervalo personalizado.
- `fechaInicio` debe ser ≤ `fechaFin`; de lo contrario el sistema retorna HTTP 400.
- El rango máximo es de 366 días; rangos mayores retornan HTTP 400.

### 5.3 Bloqueo de reportes por registros INCOMPLETOS

**5.3.1** Si el período solicitado contiene uno o más registros en estado INCOMPLETO para el empleado consultado, **todos los endpoints de reporte retornan HTTP 409** en lugar de datos o archivo. El cuerpo del error lista las fechas afectadas.

**5.3.2** El bloqueo aplica por igual a los reportes en pantalla y a las exportaciones Excel, tanto para el Reporte A como para el Reporte B.

**5.3.3** El bloqueo aplica tanto a los reportes propios del empleado como a los reportes del administrador sobre cualquier empleado.

**5.3.4** Una vez que el administrador resuelve todos los registros INCOMPLETOS del período (ingresando hora de salida manual), el reporte se habilita automáticamente en la siguiente solicitud.

**5.3.5** La interfaz intercepta el HTTP 409 y muestra al empleado un mensaje con las fechas incompletas y la instrucción de contactar al administrador; al administrador le muestra un enlace directo a la pantalla de resolución.

### 5.4 Resaltado de registros con horas extendidas

Los reportes en pantalla (Reporte A y B) resaltan visualmente las filas con horas fuera de lo habitual para facilitar la detección de jornadas extraordinarias o posibles errores de captura. El umbral se lee de la configuración de nómina activa en la fecha del registro.

**5.4.1** Sin resaltado: horas facturables ≤ horas normales diarias configuradas.

**5.4.2** Nivel advertencia (fondo amarillo/ámbar o ícono): horas facturables > horas normales diarias configuradas. Indica que se trabajaron horas extra.

**5.4.3** Nivel alerta (fondo rojo/naranja o ícono): horas facturables > horas normales diarias + máximo de horas extra diarias configurado. Indica una jornada inusualmente larga que puede ser un error de captura.

**5.4.4** El resaltado se basa en las horas facturables brutas (después de la deducción de descanso, antes del tope del Reporte A). Se aplica en las vistas en pantalla del Reporte A y del Reporte B.

**5.4.5** Los registros corregidos por el administrador siempre muestran el indicador de corrección descrito en 3.3.8, con independencia del nivel de resaltado de horas.

### 5.5 Columna Notas en exportaciones Excel

**5.5.1** Los archivos `.xlsx` generados incluyen una columna **Notas** con el siguiente contenido:
- Registros corregidos por admin: `"Corrección admin: [motivo]"`.
- Registros con nivel alerta de horas: `"Horas extendidas"`.
- Registros que cumplen ambas condiciones: ambas notas combinadas en la misma celda.
- Registros normales sin corrección: celda vacía.

---

## 6. Módulo de auditoría

**6.1** El sistema registra automáticamente toda modificación sobre:
- Configuración de nómina (parámetros globales, franjas horarias, factores de recargo, festivos).
- Registros de tiempo (entradas, salidas, reaperturas, cierres manuales por admin, correcciones directas con valores antes/después y motivo).
- Datos de usuarios, empleados y roles.
- Cambios de contraseña (sin almacenar el hash).
- Habilitación/deshabilitación de cuentas.
- Actualizaciones del propio perfil por parte del administrador.

**6.2** Los registros de auditoría son inmutables; no pueden ser modificados ni eliminados.

**6.3** Los administradores pueden consultar el historial de auditoría filtrado por fecha, usuario y tipo de entidad.

**6.4** Los empleados pueden consultar el historial de auditoría de sus propios registros de tiempo.

**6.5** Las entradas de auditoría para correcciones directas de registros incluyen: ID y nombre del administrador, fecha y hora de la corrección, empleado y fecha afectados, valores originales de entrada/salida, nuevos valores de entrada/salida y motivo de corrección.

---

## 7. Requerimientos No Funcionales

### 7.1 Acceso multiplataforma

**7.1.1** La aplicación debe ser accesible mediante **navegador web** en cualquier equipo de escritorio o portátil, sin necesidad de instalar software adicional. La interfaz web debe funcionar correctamente en las últimas dos versiones estables de los navegadores Chrome, Firefox, Edge y Safari.

**7.1.2** La aplicación debe ser accesible mediante una **aplicación móvil nativa o multiplataforma** disponible para los sistemas operativos Android e iOS. La app móvil no reemplaza a la interfaz web; ambos canales coexisten.

**7.1.3** Ambos clientes (web y móvil) consumen el **mismo backend REST API**. No se desarrollan backends separados por canal.

**7.1.4** El mecanismo de autenticación es unificado. El mismo par de credenciales (correo electrónico y contraseña) y el mismo token JWT son válidos para acceder desde cualquier canal.

**7.1.5** La **interfaz web** es la plataforma completa: soporta todas las funcionalidades de todos los módulos, incluyendo administración de usuarios, configuración de nómina, corrección de registros, auditoría y generación de reportes y archivos Excel.

**7.1.6** La **aplicación móvil** está optimizada para el uso en campo por parte del empleado. Debe ofrecer como mínimo las siguientes funciones:
- Inicio de sesión y cierre de sesión.
- Marcación de entrada (clock-in) y salida (clock-out) de la jornada.
- Consulta de los propios registros de tiempo (vista en pantalla, Reporte A y B).
- Visualización de alertas de registros INCOMPLETOS con instrucción de contactar al administrador.
- Recepción de notificaciones push sobre registros incompletos.

**7.1.7** Las funciones exclusivamente administrativas (configuración de nómina, corrección directa de registros, gestión de usuarios y roles, descarga de reportes Excel, auditoría) son accesibles únicamente desde la interfaz web. La app móvil no necesita exponerlas.

**7.1.8** La interfaz web debe ser **responsive**: el layout debe adaptarse adecuadamente a pantallas de escritorio (≥ 1280 px), portátil (≥ 1024 px) y tableta (≥ 768 px). La visualización en resoluciones inferiores a 768 px es opcional; en esos casos se sugiere al usuario utilizar la app móvil.

**7.1.9** La aplicación móvil debe funcionar en condiciones de conectividad intermitente: las operaciones de marcación deben quedar en cola local y sincronizarse automáticamente cuando se restablezca la conexión. Si no hay conexión en el momento de la marcación, el sistema debe mostrar un aviso claro al empleado e intentar la sincronización en segundo plano.

**7.1.10** La sesión del usuario en ambos canales expira según el tiempo de vida del JWT configurado en el servidor. Al expirar, el usuario es redirigido al login de forma transparente.

---

## Apéndice — Resumen de cambios respecto a v2

| # | Funcionalidad nueva o modificada |
|---|---|
| A | **Filtro por rango de fechas libre** (fechaInicio / fechaFin): permite reportes de quincenas o cualquier período personalizado en todos los reportes en pantalla y Excel. |
| B | **Registros INCOMPLETOS**: detección automática a las 00:01, aviso persistente al empleado, no bloquea asistencia del día siguiente, solo admin resuelve, notificación al admin. |
| C | **Bloqueo de reportes por INCOMPLETOS**: HTTP 409 con lista de fechas cuando el período solicitado contiene registros sin resolver; se desbloquea al resolverlos todos. |
| D | **Corrección directa de registros por administrador**: el admin puede sobrescribir hora de entrada y/o salida en cualquier registro (cualquier estado) con motivo obligatorio; se preservan valores originales y se genera auditoría completa. |
| E | **Creación de registros completos por administrador**: para fechas sin ningún registro, el admin puede crear entrada + salida con motivo, generando un registro CERRADO (admin-creado). |
| F | **Indicador visual de corrección** en reportes en pantalla: muestra motivo y nombre del admin al interactuar. |
| G | **Resaltado de horas extendidas**: advertencia (ámbar) cuando hay horas extra; alerta (rojo) cuando las horas superan el máximo configurable. Aplica en Reporte A y B en pantalla. |
| H | **Columna Notas en Excel**: incluye texto de corrección y/o alerta de horas extendidas por fila. |
| I | **Turno nocturno con cruce de medianoche**: un solo registro para turnos que cruzan las 00:00; devengado dividido en medianoche; duración máxima 24 h. |
| J | **Acceso multiplataforma**: la aplicación es accesible mediante navegador web (interfaz completa, responsive) y mediante aplicación móvil (Android/iOS, optimizada para empleados). Ambos canales comparten el mismo backend REST y el mismo sistema de autenticación JWT. La app móvil soporta funcionamiento con conectividad intermitente y notificaciones push. |
