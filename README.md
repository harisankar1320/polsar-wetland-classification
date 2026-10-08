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

## PolSAR Representation

The UAVSAR multilooked products provide six independent components of the polarimetric covariance matrix.

The $3\times3$ covariance matrix is:

$$
\mathbf{C}_3 =
\begin{bmatrix}
HHHH & HHHV & HHVV\\
HHHV^* & HVHV & HVVV\\
HHVV^* & HVVV^* & VVVV
\end{bmatrix}
$$

where:

- $HHHH = \langle |S_{HH}|^2\rangle$
- $HVHV = \langle |S_{HV}|^2\rangle$
- $VVVV = \langle |S_{VV}|^2\rangle$
- $HHHV = \langle S_{HH}S_{HV}^{*}\rangle$
- $HHVV = \langle S_{HH}S_{VV}^{*}\rangle$
- $HVVV = \langle S_{HV}S_{VV}^{*}\rangle$

The complex off-diagonal components were represented using their real and imaginary parts for the machine-learning features.

---

## Yamaguchi 4-Component Decomposition

The covariance matrix is transformed into the Pauli coherency representation:

$$
\mathbf{T}_3 =
\mathbf{D}\mathbf{C}_3\mathbf{D}^{\dagger}
$$

where

$$
\mathbf{D} =
\frac{1}{\sqrt{2}}
\begin{bmatrix}
1&0&1\\
1&0&-1\\
0&\sqrt{2}&0
\end{bmatrix}
$$

The Yamaguchi decomposition separates the observed scattering into four components:

$$
P_{\mathrm{SPAN}} \approx P_s+P_d+P_v+P_h
$$

where:

- $P_s$ — Surface / odd-bounce scattering
- $P_d$ — Double-bounce scattering
- $P_v$ — Volume scattering
- $P_h$ — Helix scattering

The total polarimetric power is:

$$
P_{\mathrm{SPAN}} = C_{11}+C_{22}+C_{33}
$$

The decomposition was performed using the **PolSARtools Yamaguchi 4-component implementation**.
### Decomposition Results


| Surface / Odd-bounce | Double-bounce |
|---|---|
| ![Surface](Images/y_odd.png) | ![Double-bounce](Images/y_double_bounce.png) |

| Volume | Helix |
|---|---|
| ![Volume](Images/y_volume.png) | ![Helix](Images/y_helix.png) |

A power-conservation check was performed by comparing:

`HHHH + HVHV + VVVV`

with:

`Ps + Pd + Pv + Ph`

The decomposition showed good agreement for pixels with sufficient total power.

---

## Random Forest

The same polygon-level train/test split was used for all experiments:

- **81 training polygons**
- **20 test polygons**
- **67,693 training pixels**
- **23,016 test pixels**

---

## Classification Features

### A. Yamaguchi Features

The first experiment uses the four decomposition components:

$$
\mathbf{x}_{Y}=[P_s,P_d,P_v,P_h]
$$

**4 features**

### B. Original PolSAR Features

The original full-polarimetric feature vector contains the three diagonal power terms and the real and imaginary parts of the three complex correlations:

$$
\mathbf{x}_{P} =
[
HHHH,HVHV,VVVV,
\Re(HHHV),\Im(HHHV),
\Re(HHVV),\Im(HHVV),
\Re(HVVV),\Im(HVVV)
]
$$

**9 features**

### C. Combined Features

The combined experiment uses both original PolSAR and Yamaguchi features:

$$
\mathbf{x}_{C} =
[\mathbf{x}_{P},P_s,P_d,P_v,P_h]
$$

**13 features**

---

## Training and Testing

The labelled polygons were divided at the **polygon level**, ensuring that pixels from the same polygon were not split between training and testing.

- **81 training polygons**
- **20 test polygons**
- **67,693 training pixels**
- **23,016 test pixels**

The same split was used for all three experiments.

### Classes

| Class | Description |
|---|---|
| 1 | Land |
| 2 | Open water |
| 3 | Wet graminoid |
| 4 | Wet shrubs |
| 5 | Wet forest |

---

## Random Forest

For each pixel, the Random Forest predicts the wetland class from its feature vector:

$$
\hat{y}=f_{\mathrm{RF}}(\mathbf{x})
$$

The models were trained using:

- **300 trees**
- `random_state = 42`
- `n_jobs = -1`

---

# Classification Results

| Feature configuration | Features | Overall Accuracy | Macro-F1 |
|---|---:|---:|---:|
| Yamaguchi | 4 | **80.66%** | **0.34** |
| Original PolSAR | 9 | **85.60%** | **0.37** |
| **Original + Yamaguchi** | **13** | **86.78%** | **0.38** |

The combined feature set produced the best overall performance.

Compared with the original PolSAR features, the combined model improved OA by:

$$86.78\%-85.60\%=1.18$$

percentage points.

Compared with Yamaguchi-only features, the improvement was:

$$86.78\%-80.66\%=6.12$$

percentage points.

---

# Per-Class Performance

For the combined feature configuration, the observed test performance was approximately:

| Class | Precision | Recall | F1 |
|---|---:|---:|---:|
| Land | 0.95 | 0.91 | **0.93** |
| Open water | 0.68 | 0.77 | **0.72** |
| Wet graminoid | 0.21 | 0.17 | **0.19** |
| Wet shrubs | 0.02 | 0.22 | **0.04** |
| Wet forest | — | — | **Not evaluated** |

The low F1 values for Wet graminoid and Wet shrubs are an important result rather than something that should be hidden by the overall accuracy.

The test set contains many more Land pixels than minority wetland vegetation pixels. Consequently, the classifier can achieve high overall accuracy while still having poor performance on some wetland classes.

This is why **Macro-F1 is reported alongside OA**.

---

# Interpretation of Classification Performance

### 1. Original PolSAR information is stronger than decomposition alone

Original PolSAR features achieved **85.60% OA** compared with **80.66%** for Yamaguchi-only features.

This indicates that the original covariance information contains useful information that is not fully represented by the four decomposition powers.

### 2. Yamaguchi features provide complementary information

Combining the original PolSAR features with Yamaguchi features increased OA from **85.60% to 86.78%** and Macro-F1 from **0.37 to 0.38**.

This suggests that the decomposition features provide additional information to the original covariance features.

### 3. Wetland vegetation remains difficult to classify

The very low F1 values for Wet graminoid and Wet shrubs indicate substantial class confusion.

Possible contributing factors include:

- strong class imbalance
- limited polygon samples
- spatial correlation between neighboring pixels
- overlap in polarimetric scattering behaviour
- heterogeneous wetland vegetation
- mixed pixels and transition zones

Therefore, the main conclusion is **not** that Yamaguchi decomposition solves wetland classification.

Instead:

> **Yamaguchi decomposition provides complementary polarimetric information and produces a small improvement when combined with the original PolSAR features, while minority wetland vegetation classes remain difficult to discriminate.**

---

# Feature Importance

Random Forest impurity-based feature importance was calculated for the combined 13-feature model.

![Feature importance](Images/feature_importance.png)

The most important features included both original PolSAR and Yamaguchi-derived variables.

Feature importance should not be interpreted as a direct physical percentage contribution to classification. Correlated features can distribute importance among one another.

---

# Permutation Importance

Permutation importance was evaluated using Macro-F1 as the scoring metric.

![Permutation importance](Images/permutation_importance.png)

The permutation results showed particularly strong contributions from features including:

- Helix $P_h$
- imaginary HHHV
- Surface $P_s$
- HHHH
- VVVV
- imaginary HVVV

Permutation importance indicates how much model performance changes when a feature's information is disrupted. It should not be interpreted as direct physical dominance of that scattering mechanism.

---

# Confusion Matrix

The combined model produced the following pixel-level confusion matrix:

```text
                 Predicted
               Land  Water  WG  WS  WF
Actual Land    17399 1149  324 322   0
       Water     691 2463   36   1   0
       WG        197    6   97 268   0
       WS         48    0    1  14   0
       WF          0    0    0   0   0
```

The matrix shows:

- strong Land classification
- strong Open Water classification
- substantial confusion involving Wet graminoid and Wet shrubs
- no independent Wet Forest test samples

The confusion matrix supports the Macro-F1 results and demonstrates why OA alone does not adequately describe performance.

---

# Limitations

### Limited minority-class samples

Wet shrubs and Wet forest have very few polygons. Wet Forest has only one polygon, which was retained for training and therefore has no independent test sample.

### Class imbalance

The test pixels are dominated by Land. This can inflate overall accuracy relative to minority-class performance.

### Spatial dependence

Pixels within a polygon are spatially correlated. A polygon-level split reduces direct pixel overlap between training and testing, but spatial dependence can still exist between nearby polygons.

### Wetland heterogeneity

Wetland vegetation classes can contain substantial internal variability in structure, moisture, and scattering behaviour.

### Feature correlation

The original PolSAR and decomposition features are not completely independent. Consequently, Random Forest feature importance should be interpreted carefully.

---

# Project Structure

```text
polsar-wetland-classification/
│
├── README.md
│
├── images/
│   ├── Yam4co_odd.png
│   ├── Yam4co_dbl.png
│   ├── Yam4co_vol.png
│   ├── Yam4co_hlx.png
│   ├── feature_importance.png
│   └── permutation_importance.png
│
└── ...
```

---

# Main Results

```text
Yamaguchi only
      │
      └── OA: 80.66%
          Macro-F1: 0.34

Original PolSAR
      │
      └── OA: 85.60%
          Macro-F1: 0.37

Original PolSAR + Yamaguchi
      │
      └── OA: 86.78%
          Macro-F1: 0.38
```

The combined feature configuration performs best.

However, the relatively low Macro-F1 compared with OA demonstrates that the model's performance is uneven across classes. Land and Open Water are classified considerably better than Wet graminoid and Wet shrubs.

---

# Conclusion

This project evaluates the contribution of Yamaguchi 4-component polarimetric decomposition to UAVSAR-based wetland classification.

The results show that:

1. **Original full-polarimetric SAR features outperform Yamaguchi-only features.**
2. **Combining original PolSAR and Yamaguchi features provides the best overall performance.**
3. **Yamaguchi features provide complementary information rather than replacing the original PolSAR information.**
4. **Overall accuracy reaches 86.78%, while Macro-F1 is 0.38.**
5. **The low Macro-F1 highlights the difficulty of classifying minority wetland vegetation classes.**
6. **Wet graminoid and Wet shrubs remain substantially harder to classify than Land and Open Water.**
7. **Wet Forest cannot be independently evaluated because only one polygon was available and it was retained for training.**

The analysis therefore supports the conclusion that **polarimetric decomposition can add useful information to full-polarimetric SAR classification, but improved discrimination of minority wetland vegetation classes requires better class representation and/or additional information.**




