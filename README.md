# 🏦 Proyecto de Migración de Seguros (COBOL/JCL/DB2)

## 📘 Descripción General
Proyecto de COBOL orientado a la **migración y transformación de datos del sector seguros**, simulando un entorno bancario real con procesos batch y uso de **DB2, COBOL y JCL**.

El objetivo principal es la **extracción, validación, transformación y carga** de información proveniente de distintas tablas de seguros y clientes, garantizando integridad y consistencia en la migración.

---

## ⚙️ Tecnologías Utilizadas
- **COBOL** (programas batch)
- **JCL (Job Control Language)**
- **DB2 (tablas y ficheros de entrada/salida)**
- **Mainframe z/OS (simulado en entorno de doméstico)**
- **Ficheros secuenciales (UNLOAD, TRANS, INC, VAL, etc.)**

---

## 🧩 Estructura del Proyecto

El sistema se divide en **cuatro fases principales**, además de las **rutinas auxiliares** empleadas durante el flujo de ejecución.

---

### 1️⃣ Fase de Extracción

**Objetivo:** Obtener los datos iniciales de las tablas del sistema original y generar los ficheros de entrada para validación.

**Programas principales:**
- `EXTCLI / EXTJCLI` → Extracción de `CLIENTES_PEPITO_SEG`
- `EXTCOM / EXTJCOM` → Extracción de `COMPANIAS_SEGUROS`
- `EXTSEG / EXTJSEG` → Extracción de `SEGUROS_PEPITO_SEG`
- `EXTSIN / EXTJSIN` → Extracción de `SINIESTROS_PEPITO_SEG`

**Salidas generadas:**
- `DES.UNLOAD.ENT.CLIENTES.F220519`
- `DES.UNLOAD.ENT.COMPANIA.F220519`
- `DES.UNLOAD.ENT.SEGUROS.F220519`
- `DES.UNLOAD.ENT.SINIEST.F220519`

---

### 2️⃣ Fase de Validación

**Objetivo:** Cruzar la información extraída para verificar integridad y filtrar registros no pertenecientes a "MAPFRE".

**Programas principales:**
- `VALSNMAP / VALJSNMA` → Cruce de **COMPANIAS_SEGUROS** vs **SEGUROS_PEPITO_SEG**
- `VALCNMAP / VALJCNMA` → Cruce de **VALSNMAP** vs **CLIENTES_PEPITO_SEG**

**Salidas:**
- `DES.VAL.SEG.NMAPFRE.F280519` → Seguros No MAPFRE  
- `DES.VAL.SEG.DESCART.F280519` → Descartes  
- `DES.VAL.CLI.NMAPFRE.F280519` → Clientes No MAPFRE  
- `DES.VAL.CLI.DESCART.F280519` → Descartes  

---

### 3️⃣ Fase de Transformación

**Objetivo:** Mapear los datos validados a los formatos requeridos por MAPFRE y generar los ficheros finales clasificados por tipo de producto.

**Programas principales:**
- `TRACLIM / TRAJCLIM` → Transforma seguros en ficheros **VIDA**, **HOGAR**, **AUTO**
- `TRASEGM / TRAJSEGM` → Transforma los **CLIENTES** asociados
- Llamada a la rutina `RUTAGEN` → Obtiene datos de agentes asociados

**Salidas:**
- `DES.TRANS.VIDA.MAP.F300519`  
- `DES.TRANS.HOGAR.MAP.F300519`  
- `DES.TRANS.AUTO.MAP.F300519`  
- `DES.TRANS.CLI.MAP.F300519`  
- `DES.TRANS.AGE.MAP.F300519`  

**Etiquetas y estructuras:**
- **01 VIDA** → PRIMA, EDAD, COBERTURAS  
- **02 HOGAR** → PRIMA, CONTINENTE, CONTENIDO, COBERTURAS  
- **03 AUTO** → PRIMA, EDAD, CATEGORÍA, COBERTURAS  
- **CATEGORÍAS:** Todo Riesgo / Terceros / Terceros con Lunas  
- **Fechas:** `FECHA_INICIO` y `FECHA_VENCIMIENTO` = Fecha de entrada  

---

### 4️⃣ Fase de Carga

**Objetivo:** Cargar los ficheros transformados en sus tablas DB2 correspondientes mediante programas batch con reposicionamiento.

**Programas principales:**
- `LOADVID / LOADJVID` → Carga tabla **VIDA MAPFRE**
- `LOADHOG / LOADJHOG` → Carga tabla **HOGAR MAPFRE**
- `LOADAUT / LOADJAUT` → Carga tabla **AUTO MAPFRE**
- `LOADCLI / LOADJCLI` → Carga tabla **CLIENTES MAPFRE**
- `LOADAGE / LOADJAGE` → Carga tabla **AGENTES MAPFRE**

**Ficheros de incidencias:**
- `DES.INC.VIDA.MAP.F030619`  
- `DES.INC.HOGAR.MAP.F030619`  
- `DES.INC.AUTO.MAP.F030619`  
- (equivalentes para CLIENTES y AGENTES)

---

### ⚙️ Rutinas Auxiliares

#### `RUTCONT`
Rutina de contravaloración (conversión monetaria).
- Entrada: `IMPORT-ORIG`, `DIV-ORIG`
- Salida: `IMPORT-DEST`, `DIV-DEST`
- Control de errores mediante `RETORNO`, `SUBRETORNO`, `SQLCODE`, etc.

#### `RUTAGEN`
Devuelve información del agente a partir del número aleatorio (`NUM-ALEA`) basado en el DNI del cliente.
- Salida: `NUM-AGE`, `DNI-AGE`, `NOMBRE`, `APE-1`, `APE-2`, `TLF`

---

## 🧾 Flujo General del Proceso
