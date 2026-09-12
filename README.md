# Dense WLAN AP Quality Evaluation

> A state-switching hybrid AP quality-score study for dense indoor WLAN environments.

This project analyzes the limitation of RSSI-only access point (AP) evaluation in a dense 5 GHz WLAN environment. It implements a hybrid quality score that retains RSSI in normal conditions and uses RTT and channel utilization (UTIL) in congested conditions.

> **Current scope**
> The available dataset records one associated target AP per measurement session. Therefore, this repository evaluates the relationship between AP quality scores and measured throughput; it does not claim Top-1 AP-selection accuracy among simultaneous multiple AP candidates.

---

## 1. Project overview

RSSI-only evaluation can favor an AP with a strong radio signal even when its channel is congested. This project evaluates a state-switching quality score:

- **Normal state:** RSSI-based quality score
- **Congested state:** weighted RTT and UTIL score, without RSSI

The goal is to determine whether the score better reflects measured communication throughput in conditions where RSSI alone is insufficient.

### Core workflow

1. Collect RSSI, link information, UTIL, RTT, packet loss, and upload/download throughput.
2. Classify each sample as normal or congested using RTT and UTIL conditions.
3. Apply RSSI scoring in normal conditions and RTT/UTIL scoring in congested conditions.
4. Determine RTT/UTIL weights using a genetic algorithm (GA) and verify them with grid search.
5. Compare RSSI, partially reproduced APQI, and hybrid scores against measured throughput.

---

## 2. Measurement record format

Each record is collected in this order: signal/link information, channel utilization, Ping-based RTT and packet loss, then actual upload/download throughput.

![Measurement record format](assets/measurement_record_format.png)

Session locations, target AP areas, measurement modes, and sample counts are listed in [docs/measurement_sessions.md](docs/measurement_sessions.md).

---

## 3. Dataset

The integrated workbook contains 400 raw samples. After excluding five incomplete records and two identified measurement-error records, the submitted-version analysis uses 393 valid samples.

| Environment | Valid samples | Normal | Congested |
|---|---:|---:|---:|
| RSSI-variation environment | 312 | 298 | 14 |
| Good-RSSI, congestion-dominant environment | 81 | 23 | 58 |
| Mixed environment | 393 | 321 | 72 |

The congestion-dominant samples were collected from induced-congestion conditions in the Open Reading Room 2 AP area. The detailed congestion-generation procedure is being verified for the paper revision.

---

## 4. Method

### State classification

The submitted-version implementation classifies a sample as congested when either condition is met:

```text
RTT >= 22 ms OR UTIL >= 56%
```

These values are environment-specific, data-derived operating thresholds rather than universal WLAN thresholds. The revision work will add training-only threshold selection and sensitivity analysis.

### Hybrid score

```text
Normal state:    Score = normalized RSSI
Congested state: Score = w_RTT * normalized RTT + w_UTIL * normalized UTIL
```

For RTT and UTIL, lower raw values correspond to higher quality scores.

### Weight search

The submitted-version full-data fit obtained:

```text
w_RTT  = 0.581
w_UTIL = 0.419
```

The GA uses the Pearson correlation between the hybrid score and upload-plus-download throughput as its fitness. Grid search at 0.001 intervals reached the same optimum. GA is therefore treated as an offline weight-search procedure rather than the main claimed contribution.

---

## 5. Submitted-version results

| Environment | RSSI | APQI partial reproduction | Hybrid score |
|---|---:|---:|---:|
| RSSI-variation | 0.817 | 0.759 | 0.795 |
| Good-RSSI, congestion-dominant | 0.222 | 0.530 | 0.586 |
| Mixed | 0.703 | 0.700 | 0.766 |

Values are Pearson correlation coefficients between the quality score and measured upload-plus-download throughput. These full-data results are retained for traceability; they must not be interpreted as independent generalization performance.

![Submitted-version score comparison](results/figures/method_comparison.png)

---

## 6. Revision status and limitations

The paper is under revision. The next analysis version will add:

- Session-level hold-out validation with train-only weight, threshold, and normalization estimation
- Threshold sensitivity analysis
- Non-switching linear-score and soft-switching comparison baselines
- Bootstrap stability analysis for RTT/UTIL weights
- Clear APQI reproduction scope and measurement-protocol documentation

Important limitations:

- No simultaneous multi-AP candidate set exists in the current dataset.
- The study is limited to one indoor library building, one laptop, and a 5 GHz WLAN environment.
- The submitted full-data results have training/evaluation overlap and are not standalone evidence of generalization.

---

## 7. Repository structure

```text
.
├── src/                    # Main reproducible analysis script
├── data/                   # Integrated measurement workbook
├── results/
│   ├── figures/            # Generated figures
│   └── tables/             # Generated CSV results
├── assets/                 # README visual assets
├── docs/                   # Session summary and project context
├── paper/                  # Submitted manuscript and review documents
├── requirements.txt
└── README.md
```

---

## 8. Reproduction

```bash
python -m venv .venv
.venv\\Scripts\\activate
pip install -r requirements.txt
python src/final_ga_analysis.py
```

The script reads `data/HOPE_측정자료_상황별정리.xlsx` and writes analysis artifacts to `results/`.

---

## 9. Source materials

- The current manuscript and review documents are retained under `paper/` for revision tracking.
- Third-party reference papers are not redistributed in this repository.
- Original measurement screenshots remain outside this repository; only one measurement-record-format image is included for documentation.
