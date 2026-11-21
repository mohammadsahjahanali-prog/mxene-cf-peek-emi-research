# Research Methodology
## 1. Refined Research Question
How accurately can tree-based ensemble models and regularized linear regression—specifically Extra Trees Regressor and Ridge Regression—predict the total Electromagnetic Interference Shielding Effectiveness ($\text{EMI SE}_{\text{total}}$, in $\text{dB}$) of lightweight $\text{Ti}_3\text{C}_2\text{T}_x$ MXene / Short Carbon Fiber–PEEK composites across varying nanofiller concentrations, fiber volume fractions, and laminate thicknesses in the X-band ($8.2\text{–}12.4\text{ GHz}$)?
## 2. Dataset Description
* **Source & Experimental Design:** Structured around a multi-factor composite compounding and waveguide testing matrix comprising **96 experimental composite specimens** fabricated via melt-blending and compression molding.
* **Input Features:**
  1. `mxene_loading_wt_pct`: Weight percentage of $\text{Ti}_3\text{C}_2\text{T}_x$ MXene nanosheets (`0.0`, `1.0`, `2.5`, `5.0`, `7.5`, `10.0` wt%).
  2. `carbon_fiber_vol_pct`: Volume fraction of short pitch-based carbon fibers (`10%`, `20%`, `30%`, `40%`).
  3. `laminate_thickness_mm`: Specimen plaque thickness tested in an X-band rectangular waveguide (`1.0`, `1.5`, `2.0`, `2.5` mm).
  4. `annealing_temp_c`: Post-molding thermal annealing temperature (`200°C` vs. `260°C`) controlling MXene terminal group removal and PEEK crystallinity.
* **Target Variable:**
  * `emi_se_db`: Total Electromagnetic Interference Shielding Effectiveness ($\text{dB}$ at $10\text{ GHz}$ mid-X-band), representing the sum of reflection loss ($\text{SE}_R$) and absorption attenuation ($\text{SE}_A$).
* **Limitations:** Because physical twin-screw extrusion and Vector Network Analyzer (VNA) waveguide measurements are scheduled for subsequent laboratory work, this initial study uses a physics-informed dataset calibrated against published Simon's formalism and percolation conductivity scaling laws.
## 3. Data Cleaning Plan
1. **Schema & Missing Value Check:** Verify strict floating-point types across all 96 records and confirm zero missing or corrupted entries.
2. **Physical Range & Outlier Filtering:** Ensure all shielding values exceed $0\text{ dB}$ and clip extreme VNA measurement noise using the $1.5 \times \text{IQR}$ criterion per thickness group.
3. **Deduplication:** Verify unique specimen identifiers across all experimental combinations.
## 4. Feature Engineering Plan
1. **Percolation Conductivity Index (`percolation_network_index`):** Captures the sharp electrical percolation jump when MXene loading surpasses the $2.5\text{ wt\%}$ threshold: $\max(0, \text{MXene} - 2.0)^{1.35} \times (1 + 0.02 \times \text{CF\_vol})$.
2. **Skin-Depth Attenuation Factor (`thickness_x_conductivity_proxy`):** Models Simon's formalism ($\text{SE}_A \propto t \sqrt{\sigma}$) by multiplying `laminate_thickness_mm` with the square root of combined conductive filler content.
3. **Thermal Crystallinity Boost (`annealing_synergy`):** Interaction feature capturing enhanced inter-sheet contact after $260^\circ\text{C}$ annealing.
4. **Standardization:** Apply `StandardScaler` fitted strictly on the 80% training split to prevent data leakage.
## 5. Models Selected and Rationale
1. **Extra Trees Regressor (`ExtraTreesRegressor`):** Selected to capture sharp, nonlinear electrical percolation thresholds and multi-filler interactions on medium-sized tabular materials datasets with low variance.
2. **Ridge Regression (`Ridge`):** Selected as an L2-regularized linear baseline to quantify the performance gain achieved by nonlinear ensemble modeling over classical linear mixing approximations.
## 6. Evaluation Metrics and Rationale
Models are evaluated on a held-out 20% test split ($n = 20$) using:
* **Coefficient of Determination ($R^2$):** Measures the proportion of variance in $\text{EMI SE}$ explained by composite formulation and thickness.
* **Root Mean Squared Error ($\text{RMSE}$, in $\text{dB}$):** Penalizes large prediction errors in critical aerospace shielding specifications.
* **Mean Absolute Error ($\text{MAE}$, in $\text{dB}$):** Measures average attenuation error directly in decibels ($\text{dB}$).
