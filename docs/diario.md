# Diario de decisiones — VIGÍA (Sentry)

**Repositorio:** [AguCS231/sentry](https://github.com/AguCS231/sentry)

---

## 2026-09-21 — Día 1: Montaje del proyecto

**Qué probé:**
- Crear repositorio en GitHub bajo la cuenta `AguCS231`.
- Montar estructura de carpetas.
- Configurar Git en Colab.
- Instalar librerías (PySpark, pandas, numpy, scikit-learn, matplotlib, seaborn).

**Qué funcionó:**
- Clonado correcto del repo `AguCS231/sentry`.
- PySpark 3.5.0 instalado tras bajar desde 4.0.4.
- pandas 2.2.3 aceptada (versión de Colab).

**Qué descarté:**
- Nombre ARGOS: requería explicación mitológica.
- PySpark 4.0.4: demasiado nuevo.
- Forzar pandas 2.1.4: rompía Colab.

**Decisiones:**
- Nombre: **VIGÍA (Sentry)**.
- Repositorio: `AguCS231/sentry` (público).
- Licencia: MIT.
- Estructura: `docs/`, `src/{data,models,security}/`, `app/`, `tests/`, `notebooks/`, `data/`.


# Diario de decisiones — VIGÍA (Sentry)

**Repositorio:** [AguCS231/sentry](https://github.com/AguCS231/sentry)

---


---

## 2026-09-23 — Verificación del dataset CIC-IDS2017

**Qué probé:**
- Verificar los 8 CSV en Google Drive.
- Calcular checksums SHA256.
- Documentar en `docs/checksums.txt`.

**Qué funcionó:**
- Los 8 CSV están en `/content/drive/MyDrive/cic-ids2017/`.
- Checksums calculados correctamente.

**Archivos del dataset:**
- Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv
- Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv
- Tuesday-WorkingHours.pcap_ISCX.csv
- Wednesday-workingHours.pcap_ISCX.csv
- Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv
- Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv
- Friday-WorkingHours-Morning.pcap_ISCX.csv
- Monday-WorkingHours.pcap_ISCX.csv

**Próximos pasos:**
- EDA (análisis exploratorio) del dataset.
- Distribución de clases, valores nulos, infinitos, negativos.


---

## 2026-09-27 — Verificación del dataset CIC-IDS2017

**Qué probé:**
- Verificar los 8 CSV en Google Drive.
- Calcular checksums SHA256.
- Documentar en `docs/checksums.txt`.

**Qué funcionó:**
- Los 8 CSV están en `/content/drive/MyDrive/cic-ids2017/`.
- Checksums calculados correctamente.

**Archivos del dataset:**
- Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv
- Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv
- Tuesday-WorkingHours.pcap_ISCX.csv
- Wednesday-workingHours.pcap_ISCX.csv
- Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv
- Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv
- Friday-WorkingHours-Morning.pcap_ISCX.csv
- Monday-WorkingHours.pcap_ISCX.csv

**Próximos pasos:**
- EDA (análisis exploratorio) del dataset.
- Distribución de clases, valores nulos, infinitos, negativos.


---

## 2026-09-28 — Preprocesado y diseño del target

**Qué probé:**
- Diseño del target binario (BENIGN=0, ATAQUE=1).
- Diseño del target por familias (8 clases operativas).
- Identificación y transformación log1p de features de cola larga.
- Split estratificado 70/15/15 con seed=42.
- Guardado de train/val/test como Parquet separados.

**Qué encontré:**
- Ratio binario: 4.08:1 (BENIGN vs ATAQUE).
- Ratio familias: 48.364:1 (BENIGN vs Otros).
- Familias entrenables: BENIGN, DoS, Probe, DDoS, Brute-force, Web, Bot.
- Familia reservada para test: Otros (Infiltration + Heartbleed, 47 muestras).

**Decisiones:**
- Binario como baseline principal (Fase 1).
- Familias como aporte (Fase 2).
- "Otros" íntegro en test → experimento de generalización a ataques no vistos.
- test.parquet separado físicamente del train y val.
- log1p aplicado a features con ratio max/mediana > 1000.

**Próximos pasos:**
- Notebook 05: modelado binario con XGBoost + experimento con/sin Destination_Port.


---

## 2026-09-29 — Modelado binario (baseline)

**Qué probé:**
- XGBoost binario (BENIGN vs ATAQUE) con dos conjuntos de features:
  - Modelo A: 78 features (incluye Destination_Port).
  - Modelo B: 77 features (excluye Destination_Port).
- Muestreo estratificado del 50% del train (991.712 filas) por límites de RAM en Colab.
- scale_pos_weight = 4.07 (compensa desbalance BENIGN/ATAQUE).
- Validación sobre val.parquet completo (424.173 filas).

**Resultados en validation:**
| Métrica          | Modelo A | Modelo B | Δ (A-B) |
|------------------|----------|----------|---------|
| Recall ATAQUE    | 0.9993   | 0.9984   | +0.0009 |
| Precision ATAQUE | 0.9949   | 0.9917   | +0.0032 |
| F1 ATAQUE        | 0.9971   | 0.9950   | +0.0021 |
| AUC-ROC          | 0.9999   | 0.9999   | +0.0001 |

**Interpretación:**
- El puerto NO es un atajo crítico en el modelo binario.
- Diferencia de recall: 0.09% (77 ataques de 83K en val).
- Métricas casi saturadas (AUC 0.9999) → el binario es demasiado fácil en este dataset.

**Decisión:**
- Se adopta el modelo B (sin Destination_Port) como baseline del proyecto por ser
  más robusto frente a evasión por cambio de puerto.
- El experimento con/sin puerto se repetirá en el modelado por familias, donde
  las clases FTP-Patator y SSH-Patator son independientes y el puerto puede ser
  más relevante.

**Limitaciones conocidas:**
- Métricas saturadas en binario → no permiten diferenciar bien los dos modelos.
- Train al 50% por restricciones de memoria (no por decisión metodológica).
- Sin ajuste de hiperparámetros (baseline con valores por defecto razonables).

**Próximos pasos:**
- Evaluación final sobre test.parquet (una sola vez).
- Notebook 06: modelado por familias (Fase 2).


---

## 2026-09-30 — Modelado binario (baseline)

**Qué probé:**
- XGBoost binario (BENIGN vs ATAQUE) con dos conjuntos de features:
  - Modelo A: 78 features (incluye Destination_Port).
  - Modelo B: 77 features (excluye Destination_Port).
- Muestreo estratificado del 50% del train (991.712 filas) por límites de RAM en Colab.
- scale_pos_weight = 4.07 (compensa desbalance BENIGN/ATAQUE).
- Validación sobre val.parquet completo (424.173 filas).

**Resultados en validation:**
| Métrica          | Modelo A | Modelo B | Δ (A-B) |
|------------------|----------|----------|---------|
| Recall ATAQUE    | 0.9993   | 0.9984   | +0.0009 |
| Precision ATAQUE | 0.9949   | 0.9917   | +0.0032 |
| F1 ATAQUE        | 0.9971   | 0.9950   | +0.0021 |
| AUC-ROC          | 0.9999   | 0.9999   | +0.0001 |

**Interpretación:**
- El puerto NO es un atajo crítico en el modelo binario.
- Diferencia de recall: 0.09% (77 ataques de 83K en val).
- Métricas casi saturadas (AUC 0.9999) → el binario es demasiado fácil en este dataset.

**Decisión:**
- Se adopta el modelo B (sin Destination_Port) como baseline del proyecto por ser
  más robusto frente a evasión por cambio de puerto.
- El experimento con/sin puerto se repetirá en el modelado por familias, donde
  las clases FTP-Patator y SSH-Patator son independientes y el puerto puede ser
  más relevante.

**Limitaciones conocidas:**
- Métricas saturadas en binario → no permiten diferenciar bien los dos modelos.
- Train al 50% por restricciones de memoria (no por decisión metodológica).
- Sin ajuste de hiperparámetros (baseline con valores por defecto razonables).

**Próximos pasos:**
- Evaluación final sobre test.parquet (una sola vez).
- Notebook 06: modelado por familias (Fase 2).


---

## 2026-09-30 — Evaluación final del modelo binario sobre test

**Qué probé:**
- Evaluación única del modelo B (sin Destination_Port) sobre test.parquet.
- Cálculo del recall específico sobre la familia "Otros" (Infiltration + Heartbleed, 47 muestras no vistas en entrenamiento).

**Resultados en test:**
- (rellenar con las cifras de la celda 2)
- Recall familia "Otros": (rellenar)

**Interpretación:**
- (rellenar tras ver los resultados)

**Decisión:**
- Modelo B queda cerrado como baseline del proyecto. No se reajusta.
- El notebook 06 arranca el modelado por familias (Fase 2).

**Próximos pasos:**
- Modelo multi-clase por familias (7 clases).
