# Auditoría de Sistemas · Core Financiero de la COOPAC Santa Rosa

> **SI-084 · Auditoría de Sistemas** · Examen práctico de Unidad I  
> **Universidad Privada de Tacna** · Facultad de Ingeniería · Escuela Profesional de Ingeniería de Sistemas  
> **Docente:** Dr. Oscar Juan Jimenez Flores

---

## 👤 Datos del Estudiante

| Campo | Detalle |
|---|---|
| **Apellidos y nombres** | Castillo Mamani Diego Fernando |
| **Código de estudiante** | `2022073895` |
| **Repositorio** | [diegocastillo12/llamado-si084-caso-coopac.](https://github.com/diegocastillo12/llamado-si084-caso-coopac.) |
| **Etiqueta (Tag)** | [`examen-u1`](https://github.com/diegocastillo12/llamado-si084-caso-coopac./tree/examen-u1) |
| **Fecha de corte** | 31/12/2025 |

---

## 📋 Resumen Ejecutivo de la Auditoría

El Consejo de Administración de la **COOPAC Santa Rosa** solicitó responder a la pregunta:  
> *¿Cumple el servidor de base de datos del core financiero la política de seguridad que el Consejo aprobó al 31/12/2025?*

### ⚠️ Dictamen: NO CUMPLE
Tras ejecutar los 7 procedimientos técnicos de auditoría contra la **Política de Seguridad del Core Financiero (Versión 2.0)** y la norma técnica **NTP-ISO/IEC 27001:2022**, se determinó que **el servidor incumple la totalidad de las reglas (R1 a R7)** evaluadas.

---

## 📊 1. Matriz de Cumplimiento de Procedimientos

| Regla | Control ISO 27001:2022 | Resultado Obtenido (con cifras) | ¿Cumple? | Archivo de Evidencia |
|:---:|---|---|:---:|---|
| **R1** | A.8.20 Seguridad de redes | Publica en `0.0.0.0:55432` (y `[::]:55432`), expuesto a todas las redes externas. | ❌ **No** | [`evidencias/P1_puertos.txt`](evidencias/P1_puertos.txt) |
| **R2** | A.5.17 Autenticación | Contraseña en texto plano en `docker-compose.yml` (línea 10) con solo **10 caracteres** (`coopac2023`). | ❌ **No** | [`evidencias/P2_credenciales.txt`](evidencias/P2_credenciales.txt) |
| **R3** | A.8.2 Acceso privilegiado | Cuenta de aplicación **`app_core`** posee atributo **`Superuser`**. | ❌ **No** | [`evidencias/P3_roles.txt`](evidencias/P3_roles.txt) |
| **R4** | A.5.16 / A.5.18 Identidades y accesos | **16 cuentas activas** de **10 cesados** (cese más antiguo: 18/12/2015). **22 cuentas huérfanas**, 4 con perfil **`ADMIN`**. | ❌ **No** | [`evidencias/P4_cesados.txt`](evidencias/P4_cesados.txt)<br>[`evidencias/P4_genericas.txt`](evidencias/P4_genericas.txt) |
| **R5** | A.5.3 Segregación de funciones | **23 desembolsos** aprobados por el mismo que los registró superando el umbral, por un total de **S/ 709,370.47** (21 usuarios). | ❌ **No** | [`evidencias/P5_segregacion.txt`](evidencias/P5_segregacion.txt) |
| **R6** | A.8.15 Registro de eventos | Auditoría inhabilitada: `log_connections = off` y `log_statement = none` en línea 11 de `docker-compose.yml`. | ❌ **No** | [`evidencias/P6_registro.txt`](evidencias/P6_registro.txt) |
| **R7** | A.8.13 Respaldo de información | **47 días sin respaldo** por disco lleno. En la restauración falta la tabla **`desembolsos`** por exclusión deliberada en script. | ❌ **No** | [`evidencias/P7_respaldos.txt`](evidencias/P7_respaldos.txt)<br>[`evidencias/P7_restauracion.txt`](evidencias/P7_restauracion.txt) |

---

## 🚨 2. Hallazgos Críticos de Auditoría

### 🔴 Hallazgo 1: Vulneración de la segregación de funciones en 23 desembolsos de créditos por S/ 709,370.47
* **Condición:** 23 operaciones de crédito que superaron su umbral de aprobación fueron auto-aprobadas por el mismo colaborador que las registró (`usuario_registra = usuario_aprueba`), sumando **S/ 709,370.47** e involucrando a 21 usuarios distintos.
* **Criterio:** Política R5 y control **A.5.3 Segregación de funciones** (NTP-ISO/IEC 27001:2022).
* **Causa:** Carencia de validaciones y restricciones (`CHECK constraints` / triggers) tanto en la base de datos como en la lógica del software core.
* **Efecto:** Riesgo material de fraude interno, malversación de fondos y sanciones regulatorias por parte de la SBS.
* **Recomendación:** Implementación obligatoria de validaciones en base de datos en máximo 15 días hábiles e investigación pericial sobre las 23 operaciones.

### 🔴 Hallazgo 2: Inoperatividad del sistema de respaldos durante 47 días y exclusión deliberada de la tabla de desembolsos
* **Condición:** El último respaldo disponible data del 14/11/2025 (**47 días sin copias de seguridad** al corte por fallo de espacio `No space left on device`). La prueba de restauración reveló que la tabla crítica `desembolsos` no existe en la copia.
* **Criterio:** Política R7 y control **A.8.13 Respaldo de la información** (NTP-ISO/IEC 27001:2022).
* **Causa:** Modificación intencional en `respaldos/respaldo.sh` con el flag `--exclude-table=desembolsos` por parte del Jefe de Sistemas; ausencia total de monitoreo de capacidad de almacenamiento y supervisión de logs de error.
* **Efecto:** Pérdida irrecuperable de la cartera de colocaciones ante siniestros y quiebre de la continuidad operativa de la cooperativa.
* **Recomendación:** Corrección inmediata en 24 horas del script de respaldo, ampliación de almacenamiento con políticas de rotación y establecimiento de auditorías trimestrales de restauración.

---

## 🛠️ Reproducción y Verificación Técnica

### 1. Levantar el entorno
```bash
docker compose up -d --wait
```

### 2. Verificar la integridad de las evidencias generadas
```bash
sha256sum -c evidencias/SHA256SUMS.txt
```

### 3. Apagar el entorno
```bash
docker compose down -v
```

---

## 📁 Archivos Entregables

* 📄 **[INFORME.md](INFORME.md):** Plantilla oficial del informe de auditoría completada.
* 📝 **[SI084-EXPRAC-U1-CastilloDiego.docx](SI084-EXPRAC-U1-CastilloDiego.docx):** Informe de auditoría en formato Microsoft Word listo para exportar a PDF.
* 🗂️ **[`evidencias/`](evidencias/):** Salidas de consola de los 7 procedimientos y sumas de verificación SHA-256.
