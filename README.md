# Full-Polarimetric SAR Wetland Classification

**Study area:** Peace–Athabasca Delta (PAD), Canada  
**SAR data:** UAVSAR L-band full-polarimetric SAR  
**Classifier:** Random Forest  
**Classes:** Land, Open water, Wet graminoid, Wet shrubs, Wet forest

---

## Workflow

```text
UAVSAR Full-Pol
      ↓
C3 PolSAR features
      ↓
Yamaguchi 4-component decomposition
      ↓
Original / Yamaguchi / Combined features
      ↓
Random Forest
      ↓
5-class wetland classification
```

## Yamaguchi 4-Component Decomposition

The four Yamaguchi components used as features were:

- **Ps** — Surface / odd-bounce
- **Pd** — Double-bounce
- **Pv** — Volume
- **Ph** — Helix

### Decomposition Results


| Surface / Odd-bounce | Double-bounce |
|---|---|
| ![Surface](images/y_odd.png) | ![Double-bounce](images/y_double_bounce.png) |

| Volume | Helix |
|---|---|
| ![Volume](images/y_volume.png) | ![Helix](images/y_helix.png) |

A power-conservation check was performed by comparing:

`HHHH + HVHV + VVVV`

with:

`Ps + Pd + Pv + Ph`

The decomposition showed good agreement for pixels with sufficient total power.

---

## Random Forest Experiments

The same polygon-level train/test split was used for all experiments:

- **81 training polygons**
- **20 test polygons**
- **67,693 training pixels**
- **23,016 test pixels**

| Experiment | Features | Overall Accuracy | Macro-F1 |
|---|---:|---:|---:|
| **A — Yamaguchi** | 4 | **80.66%** | **0.34** |
| **B — Original PolSAR** | 9 | **85.60%** | **0.37** |
| **C — Original + Yamaguchi** | 13 | **86.78%** | **0.38** |

### Main Result

The **combined 13-feature model performed best**.

- Original PolSAR → Combined: **+1.18 percentage points OA**
- Yamaguchi → Combined: **+6.12 percentage points OA**

The wet vegetation classes remained difficult to separate, particularly **Wet graminoid and Wet shrubs**.

---

## Combined Model — Feature Importance

The impurity-based Random Forest importance ranked the features approximately:

`HVHV > HHHH > Pv > VVVV > Pd > Ph > ...`

![Feature Importance](images/feature_importance.png)
---

## Permutation Importance

Using **macro-F1** as the scoring metric, the strongest permutation importance was observed for **Ph**, followed by **Im_HHHV** and **Ps**.

![Permutation Importance](images/permutation_importance.png)

The two importance methods measure different things: tree-split contribution versus the change in test performance after shuffling a feature.

---

## Conclusion

- Original full-polarimetric features performed better than Yamaguchi features alone.
- Yamaguchi features provided additional information when combined with the original PolSAR features.
- The **13-feature combined model achieved the best overall performance (86.78% OA)**.
- Wet vegetation classes remained challenging.

**Limitation:** Wet forest had only one labelled polygon, which was kept in the training set. Therefore, independent test performance for Wet forest could not be evaluated.
