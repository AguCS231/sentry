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
