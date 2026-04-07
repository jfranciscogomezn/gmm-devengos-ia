Una aplicacion que permita calcular el valor devengado por los empleados de una empresa con las siguientes consideraciones:
1. modulo de seguridad:
1.1 la contraseña se debe guardar cifrada bcryp en la base de datos junto con la informacion de miembros
1.2 Recomiendame caracteristicas estandar de calidad de la contraseña
1.4 El usuario solo puede acceder a las funcionalidades que tenga el rol asociado.
1.5 Al registrar un miembro en la aplicacion se puede o no habilitar el usuario, solo aquellos habilitados con usuario y contraseña podran ingresar.
1.6 Agrega una funcionalidad para definir/gestionar roles de usuario
1.7 un usuario solo puede tener 1 rol al tiempo
1.8 al rol se deben asignar opciones de menu

2. modulo de configuracion:
2.1 El sistema debe permitir configurar todos los datos transversales para la nomina como son:
2.1.3 Cantidad de horas de trabajo al mes
2.1.4 cantidad de horas diarias jornada normal semanal 
2.1.5 cantidad de horas diarias horario extra
--horas
2.1.6 hora inicio diurno 
2.1.7 hora fin diurno
2.1.8 hora inicio extra diurna 
2.1.9 hora fin extra diurna 
2.1.10 hora inicio recargo nocturno
2.1.11 hora fin recargo nocturno
2.1.12 hora inicio extra nocturna 
2.1.13 hora fin extra nocturna
2.1.14 hora inicio extra domingo
2.1.15 hora fin extra domingo
--factores recargos
2.1.16 factor extra diurna
2.1.17 factor extra nocturna
2.1.18 factor recargo nocturno
2.1.19 factor extra diurno domingos y festivos
2.1.20 factor extra nocturno domingos y festivos

2.1.21 listado de domingos y festivos por año
2.1.22 cantidad de horas no contables en jornada laboral
2.2 El sistema debe permitir al administrador configurar por empleado:
2.2.1 el salario del trabajador
2.2.2 nombres, apellidos, tipo y numero de documento de identificacion, correo electronico y numero de celular


3. modulo de registro de tiempo de trabajo:
3.1 Los empleados podran marcar el inicio y fin de su jornada laboral solo una vez al dia,no podrá ser editada luego de finalizar(a no ser que el administrador lo abra nuevamente)
3.1.1 el empleado solicita al administrador la reapetura, el sistema notifica al administrador, el administrador apertura y el sistema notifica al empleado para que el cierre nuevamente
3.2 Los empleados podran ver su tiempo de trabajo filtrado por dia, semana y mes con las siguientes consideraciones:
3.2.1 ver en pantalla el registro de su tiempo de trabajo con un tiempo maximo de trabajo diario de 8 horas diarias y 2 horas extra diurna(deben ser visiblemente reconocibles) y valor devengado(reporte A)
3.2.2 descargar su reporte A en formato excel de tiempo de trabajo y valor devengado limitando el tiempo trabajado a 8 horas diarias y 2 horas extra diurna
3.3 Los administradores podran:
3.3.1 ver en pantalla el registro de tiempos de trabajo de los empleados con un tiempo maximo de trabajo diario de 8 horas diarias y 2 horas extra diurna(deben ser visiblemente reconocibles) y valor devengado(reporte A)
3.3.2 ver en pantalla el registro de tiempos de trabajo de los empleados con los tiempos totales diarios registrados por los empleados(reporte B)
3.3.3 descargar el reporte en formato excel de tiempo de trabajo y valor devengado para un empleado en especifico con un tiempo maximo de trabajo diario de 8 horas diarias y 2 horas extra diurna(deben ser visiblemente reconocibles) y valor devengado(reporte A)
3.3.4 descargar el reporte en formato excel de tiempo de trabajo y valor devengado para un empleado en especifico con el tiempo de trabajo total y el valor devengado(reporte B)
3.4 para el valor devengado por el empleado tener en cuenta la seccion de configuracion:
3.4.1 la cantidad de horas diarias registradas tendran valor segun el rango y factor(diurna, extra diurna, nocturna, etc)
3.4.2 los factores de las franjas de horas de trabajo
3.4.3 los tiempos de descanso diarios configurados deben descontarse del total de tiempo trabajado 
3.4.4 el reporte A corresponde a los datos sin el tope semanal de 12 horas semanales de horas extra
3.4.5 el reporte B corresponde a los datos reales sin topes registrados por los empleados

