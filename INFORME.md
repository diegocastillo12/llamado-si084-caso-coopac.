# Informe de auditoría · Core financiero de la COOPAC Santa Rosa

**SI-084 · Auditoría de Sistemas** · Examen práctico de Unidad I

| | |
|---|---|
| **Apellidos y nombres** | Castillo Mamani Diego Fernando |
| **Código de estudiante** | 2022073895 |
| **URL del repositorio** | `https://github.com/diegocastillo12/llamado-si084-caso-coopac.`(https://github.com/diegocastillo12/llamado-si084-caso-coopac) |
| **Fecha** | 30/09/2026 |

## 1. Resultados de los procedimientos

| Regla | Resultado, con cifras | ¿Cumple? | Archivo de evidencia |
|---|---|---|---|
| R1 | El contenedor `sr_bd` publica el puerto `5432` en `0.0.0.0:55432` (y `[::]:55432`), exponiendo el servicio de base de datos a todas las interfaces de red externas y no únicamente al host local. | No | `evidencias/P1_puertos.txt` |
| R2 | La contraseña del administrador se encuentra en texto plano en el archivo `docker-compose.yml` (línea 10: `POSTGRES_PASSWORD`) y cuenta únicamente con 10 caracteres, incumpliendo el requisito de longitud mínima de 12 caracteres. | No | `evidencias/P2_credenciales.txt` |
| R3 | Además de la cuenta administradora nativa `postgres`, la cuenta `app_core` posee asignado el atributo `Superuser`, otorgando privilegios ilimitados sobre el motor. | No | `evidencias/P3_roles.txt` |
| R4 | Se detectaron 16 cuentas activas pertenecientes a 10 empleados cesados (el cese más antiguo data del 18/12/2015, usuario `u059`). Asimismo, existen 22 cuentas activas sin documento de identidad asignado (sin responsable), de las cuales 4 poseen perfil `ADMIN` (`backup`, `backup_3`, `consulta01_3`, `temporal`). | No | `evidencias/P4_cesados.txt` · `evidencias/P4_genericas.txt` |
| R5 | Se identificaron 23 desembolsos que superan el umbral de aprobación y fueron aprobados por el mismo usuario que los registró (`usuario_registra = usuario_aprueba`), sumando un monto total de S/ 709,370.47, realizados por 21 usuarios distintos. | No | `evidencias/P5_segregacion.txt` |
| R6 | Los parámetros de auditoría están inhabilitados en el motor: `log_connections = off` y `log_statement = none`, configurados explícitamente en la línea 11 de `docker-compose.yml`. | No | `evidencias/P6_registro.txt` |
| R7 | El último respaldo generado es del 14/11/2025 (47 días sin respaldos válidos hasta el corte del 31/12/2025 por fallos de disco `No space left on device`). En la prueba de restauración no se recuperó la tabla crítica `desembolsos` debido a que `respaldos/respaldo.sh` la excluyó intencionalmente (`--exclude-table=desembolsos`). | No | `evidencias/P7_respaldos.txt` · `evidencias/P7_restauracion.txt` |

## 2. Hallazgo 1

| Elemento | Contenido |
|---|---|
| Título | Vulneración de la segregación de funciones en 23 desembolsos de créditos por S/ 709,370.47 aprobados por el mismo usuario que los registró |
| Condición | En la evaluación de la tabla `desembolsos`, se identificaron 23 operaciones de crédito que superaron su umbral de aprobación y fueron aprobadas por el mismo colaborador que efectuó el registro (`usuario_registra = usuario_aprueba`), totalizando un importe de S/ 709,370.47 e involucrando a 21 usuarios distintos (evidencia: `evidencias/P5_segregacion.txt`). |
| Criterio | Política de Seguridad del Core Financiero de la COOPAC Santa Rosa, Regla R5: *"Un desembolso que supera el umbral de aprobación no puede aprobarlo quien lo registró"*, en concordancia con NTP-ISO/IEC 27001:2022, control **A.5.3 Segregación de funciones**. |
| Causa | El aplicativo del core financiero carece de restricciones a nivel de base de datos (`CHECK constraints` o triggers) y de validaciones lógicas en la capa de negocio que impidan que un mismo usuario actúe como registrador y aprobador cuando la operación rebasa el umbral autorizado. |
| Efecto | Exposición directa de la cooperativa a riesgo de fraude interno, malversación de fondos o colocación de créditos indebidos por hasta S/ 709,370.47, además de posibles sanciones administrativas y regulatorias por parte de la SBS por debilidades materiales de control interno. |
| Recomendación | Que el Jefe de Sistemas, en un plazo no mayor a 15 días hábiles, implemente validaciones mandatorias a nivel de base de datos (trigger/constraint) y software core que impidan la autoaprobación de desembolsos; y que el Oficial de Cumplimiento / Auditoría Interna inicie una investigación exhaustiva sobre las 23 operaciones detectadas. |

## 3. Hallazgo 2

| Elemento | Contenido |
|---|---|
| Título | Inoperatividad del sistema de respaldos del core financiero durante 47 días y exclusión deliberada de la tabla de desembolsos |
| Condición | Se verificó que el último respaldo generado data del 14/11/2025, acumulando 47 días sin copias de seguridad al corte del 31/12/2025 debido a fallos recurrentes por saturación de almacenamiento (`No space left on device`). Asimismo, al ejecutar la restauración del último archivo disponible (`core_2025-11-14.sql`) en la base `restauracion`, se constató la ausencia total de la tabla `desembolsos` (evidencias: `evidencias/P7_respaldos.txt` y `evidencias/P7_restauracion.txt`). |
| Criterio | Política de Seguridad del Core Financiero de la COOPAC Santa Rosa, Regla R7: *"Respaldo diario completo, que incluye la tabla de desembolsos. Su restauración se prueba cada trimestre"*, en concordancia con NTP-ISO/IEC 27001:2022, control **A.8.13 Respaldo de la información**. |
| Causa | El script de respaldo (`respaldos/respaldo.sh`) fue alterado el 01/11/2025 por el Jefe de Sistemas para omitir la tabla de desembolsos con el argumento de que "tarda mucho"; adicionalmente, no existe un monitoreo del almacenamiento asignado a los respaldos ni supervisión de los registros de error (`respaldos/respaldo.log`), permitiendo que el fallo persista desapercibido desde el 15/11/2025. |
| Efecto | Riesgo inminente de pérdida irrecuperable de la cartera de colocaciones de la cooperativa ante cualquier contingencia o siniestro en el servidor, provocando la imposibilidad de operar, pérdida financiera incuantificable y nulidad de continuidad de negocio (RPO de más de mes y medio). |
| Recomendación | Que el Jefe de Sistemas en un plazo perentorio de 24 horas modifique `respaldos/respaldo.sh` para incluir todas las tablas sin exclusión, amplíe la capacidad del almacenamiento de respaldos con políticas de depuración y configure alertas automáticas por fallo; y que el Comité de TI establezca y audite un protocolo trimestral obligatorio de restauración de pruebas. |
