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


---

## 2026-09-30 — Modelo por familias (Fase 2, primera versión)

**Qué probé:**
- XGBoost multi-clase sobre 7 familias (BENIGN, DoS, Probe, DDoS, Brute-force, Web, Bot).
- Features: las mismas 77 del modelo binario B (sin Destination_Port).
- Muestreo de train al 50% (990.947 filas) por límites de RAM en Colab.
- Balanceo con `compute_sample_weight(class_weight="balanced")` → pesos entre 0.18 (BENIGN) y 207.27 (Bot).

**Resultados en validation:**
- F1 macro: 0.9575
- F1 weighted: 0.9988
- F1 por clase:
  - BENIGN: 0.9992 | DoS: 0.9980 | Probe: 0.9965
  - DDoS: 0.9994 | Brute-force: 0.9993 | Web: 0.9726 | Bot: 0.7378

**Análisis de la clase Bot:**
- Recall 0.9931 (detecta todos los ataques reales).
- Precision 0.5869 → 202 BENIGN clasificados como Bot (0.06% del total BENIGN).
- Interpretación: el peso 207× provoca sobredetección en dirección Bot. Operativamente es ruido tolerable, metodológicamente es un hallazgo.

**Decisiones:**
- No reajustamos los pesos: el resultado es defendible y el análisis de la clase Bot es un aporte.
- El modelo se guarda como `familias_sin_puerto.json` con metadatos completos.
- Se mantiene la coherencia metodológica con el binario (mismas 77 features).

**Limitaciones conocidas:**
- Train al 50% por RAM.
- Clase Bot con precisión baja por peso desproporcionado.
- Evaluación pendiente sobre test (aún no ejecutada).

**Próximos pasos:**
- Experimento con/sin Destination_Port también en familias, para completar la Fase 1.
- SHAP por clase (Fase 2).


---

## 2026-10-01 — Explicabilidad SHAP del modelo por familias

**Qué probé:**
- SHAP TreeExplainer sobre el modelo por familias sin puerto.
- Cálculo sobre muestra aleatoria de 2000 flujos de validación.
- Importancia global y por clase.

**Top 5 features globales:**
1. Init_Win_bytes_forward: 1.4094
2. Init_Win_bytes_backward: 1.0819
3. min_seg_size_forward: 0.9185
4. Flow_IAT_Min: 0.3111
5. Bwd_Packet_Length_Min: 0.3033

**Artefactos generados:**
- `docs/shap_importancia_global.png`
- `docs/shap_por_clase.png`
- `src/models/shap_top_features_por_clase.json`

**Próximos pasos:**
- Dashboard con explicaciones SHAP en cada alerta.
- Evasión adversarial controlada (Fase 3).


---

## 2026-10-01 — Cierre del notebook 07: evaluación en test y decisión formal

**Evaluación en TEST (modelo sin puerto, baseline elegido):**
- F1 macro: 0.9600
- F1 weighted: 0.9987
- F1 por clase:
  - BENIGN: 0.9992
  - DoS: 0.9976
  - Probe: 0.9967
  - DDoS: 0.9993
  - Brute-force: 0.9988
  - Web: 0.9749
  - Bot: 0.7537

**Comparativa con/sin puerto:** realizada en validación (notebook 07, sección 5). No se repitió en test para preservar la regla de uso único del conjunto de test.

**Experimento de generalización (familia "Otros", no vista en train):**
- Total muestras: 47
- Detectados como ataque: 2 (4.3%)
- Clasificados como BENIGN: 45 (95.7%)

**Hallazgos SHAP documentados:**
- Las features dominantes son de comportamiento real del flujo (tamaños, flags TCP, timing), no identificadores.
- `Init_Win_bytes_forward` y `Init_Win_bytes_backward` dominan la mayoría de clases. Estas features dependen del sistema operativo del emisor y son manipulables por un atacante. Se documentan como limitación conocida y se plantea experimento adicional sin ellas (notebook 08).

**Decisión formal:**
- Modelo de familias oficial del proyecto: **sin Destination_Port**.
- Coherencia con el binario y con el análisis de fuga.

**Limitaciones conocidas:**
- Train al 50% por RAM.
- Clase Bot con precision baja (0.59) por peso desproporcionado.
- Modelo con puerto no persistido en disco (solo análisis en validación).

**Próximos pasos (notebook 08):**
- Experimento sin `Init_Win_bytes`.
- Evasión adversarial controlada.
- Dashboard con SHAP integrado.
