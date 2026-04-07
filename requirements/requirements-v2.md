Una aplicacion que permita calcular el valor devengado por los empleados de una empresa con las siguientes consideraciones:

1. modulo de seguridad:
1.1 la contraseña se debe guardar cifrada bcrypt en la base de datos junto con la informacion del usuario. Las credenciales de acceso son correo electronico y contraseña.
1.2 Caracteristicas de calidad de la contraseña: minimo 8 caracteres, 1 mayuscula, 1 minuscula, 1 digito, 1 caracter especial.
1.3 El usuario solo puede acceder a las funcionalidades que tenga el rol asociado.
1.4 Solo los usuarios habilitados podran ingresar al sistema con su correo electronico y contraseña.
1.5 El registro de usuarios debe ser realizado exclusivamente por un administrador; al registrar un usuario se definen sus datos personales (nombres, apellidos, correo electronico, celular), una contraseña inicial y el rol asignado. El correo electronico debe ser unico en el sistema y actua como identificador de acceso.
1.6 El usuario puede cambiar su propia contraseña despues del primer ingreso. El administrador puede resetear la contraseña de cualquier usuario.
1.7 Un administrador puede actualizar su propio perfil de usuario (datos personales); esta accion debe quedar en el registro de auditoria igual que cualquier otra actualizacion de perfil.
1.8 Un usuario puede consultar su propio perfil (datos personales, rol) en modo solo lectura. Para modificar sus datos debe solicitar al administrador.
1.9 La eliminacion de un usuario esta bloqueada si tiene registros asociados (registros de tiempo, perfil de empleado, entradas de auditoria). Si no tiene registros asociados, el administrador puede eliminarlo.
1.10 Agrega una funcionalidad para definir/gestionar roles de usuario.
1.11 Un usuario solo puede tener 1 rol al tiempo.
1.12 Al rol se deben asignar opciones de menu.
1.13 Los usuarios pueden adicionalmente ser configurados como empleados (trabajadores) por un administrador. Para poder hacer el registro de horas de inicio y fin en un dia especifico, el usuario debe estar configurado como empleado y tener sesion activa.

2. modulo de configuracion:
2.1 El sistema debe permitir configurar todos los datos transversales para la nomina como son:

2.1.1 Cantidad de horas de trabajo al mes
2.1.2 cantidad de horas diarias jornada normal semanal
2.1.3 cantidad de horas diarias horario extra

--horas
2.1.4 hora inicio diurno
2.1.5 hora fin diurno
2.1.6 hora inicio extra diurna
2.1.7 hora fin extra diurna
2.1.8 hora inicio recargo nocturno (por defecto 21:00 segun CST Art. 160)
2.1.9 hora fin recargo nocturno (por defecto 06:00 segun CST Art. 160)
2.1.10 hora inicio extra nocturna
2.1.11 hora fin extra nocturna
2.1.12 hora inicio extra domingo
2.1.13 hora fin extra domingo

--factores recargos
2.1.14 factor extra diurna (minimo legal 1.25)
2.1.15 factor extra nocturna (minimo legal 1.75)
2.1.16 factor recargo nocturno (minimo legal 1.35)
2.1.17 factor extra diurno domingos y festivos (minimo legal 2.00)
2.1.18 factor extra nocturno domingos y festivos (minimo legal 2.50)
2.1.19 factor recargo normal domingos y festivos (minimo legal 1.75)

2.1.20 listado de domingos y festivos por año
2.1.21 cantidad de horas no contables en jornada laboral

2.2 El sistema debe permitir al administrador configurar por empleado:
2.2.1 el salario del trabajador
2.2.2 tipo y numero de documento de identificacion del empleado (para efectos de nomina)
2.2.3 Un empleado debe estar previamente registrado como usuario en el sistema; los datos personales del empleado (nombres, apellidos, correo, celular) provienen directamente del registro de usuario.
2.2.4 Un usuario puede o no ser empleado; solo los usuarios configurados como empleados usan el modulo de registro de tiempo.


3. modulo de registro de tiempo de trabajo:
3.1 Los empleados con sesion activa podran marcar el inicio y fin de su jornada laboral solo una vez al dia, no podra ser editada luego de finalizar (a no ser que el administrador lo abra nuevamente)
3.1.1 el empleado solicita al administrador la reapertura, el sistema notifica al administrador, el administrador apertura y el sistema notifica al empleado para que este pueda cerrar nuevamente
3.2 Los empleados podran ver su tiempo de trabajo filtrado por dia, semana y mes con las siguientes consideraciones:
3.2.1 ver en pantalla el registro de su tiempo de trabajo con un tiempo maximo de trabajo diario de 8 horas diarias y 2 horas extra diurna (deben ser visiblemente reconocibles) y valor devengado (reporte A)
3.2.2 descargar su reporte A en formato excel de tiempo de trabajo y valor devengado limitando el tiempo trabajado a 8 horas diarias y 2 horas extra diurna
3.2.3 ver en pantalla sus tiempos totales reales registrados sin tope y valor devengado real (reporte B)
3.3 Los administradores podran:
3.3.1 ver en pantalla el registro de tiempos de trabajo de los empleados con un tiempo maximo de trabajo diario de 8 horas diarias y 2 horas extra diurna (deben ser visiblemente reconocibles) y valor devengado (reporte A)
3.3.2 ver en pantalla el registro de tiempos de trabajo de los empleados con los tiempos totales diarios registrados por los empleados (reporte B)
3.3.3 descargar el reporte en formato excel de tiempo de trabajo y valor devengado para un empleado en especifico con un tiempo maximo de trabajo diario de 8 horas diarias y 2 horas extra diurna (deben ser visiblemente reconocibles) y valor devengado (reporte A)
3.3.4 descargar el reporte en formato excel de tiempo de trabajo y valor devengado para un empleado en especifico con el tiempo de trabajo total y el valor devengado (reporte B)
3.4 para el valor devengado por el empleado tener en cuenta la seccion de configuracion:
3.4.1 la cantidad de horas diarias registradas tendran valor segun el rango y factor (diurna, extra diurna, nocturna, recargo nocturno, etc)
3.4.2 los factores de las franjas de horas de trabajo
3.4.3 los tiempos de descanso diarios configurados deben descontarse del total de tiempo trabajado
3.4.4 el reporte A aplica el tope diario (8h normal + 2h extra diurna) Y el tope semanal de 12 horas extra (CST Art. 162)
3.4.5 el reporte B corresponde a los datos reales sin topes registrados por los empleados

4. modulo de auditoria:
4.1 el sistema debe registrar automaticamente toda modificacion sobre: configuracion de nomina (parametros, franjas, factores, festivos), registros de tiempo (entradas, salidas, reaperturas, cierres manuales), datos de usuarios/empleados/roles, cambios de contraseña (sin almacenar el hash), habilitacion/deshabilitacion de cuentas y actualizaciones del propio perfil por parte del administrador.
4.2 los registros de auditoria son inmutables.
4.3 los administradores pueden consultar el historial de auditoria filtrado por fecha, usuario y tipo de entidad.
4.4 los empleados pueden consultar el historial de auditoria de sus propios registros de tiempo.
