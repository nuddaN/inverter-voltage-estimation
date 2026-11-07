# Inverter Black-Box Model: Phase Voltage Estimation

Data-driven model of a three-phase IGBT inverter that estimates the phase voltage from the quantities a
drive controller already measures: phase currents, PWM duty cycles, DC-link voltage and motor speed.
A model like this can be used to simulate an electric drive, or to compensate the inverter non-linearity
without a voltage sensor on each phase.

**Notebook on Kaggle (runnable):**
[Inverter Black-Box Model: Phase Voltage Estimation](https://www.kaggle.com/code/niccolnudda/inverter-black-box-model-phase-voltage-estimation)

## Data

[Inverter Data Set](https://www.kaggle.com/stender/inverter-data-set) by Stender et al., Paderborn
University: about 235,000 samples from a test bench with inverter, induction motor and control system.
Inputs are the speed, the DC-link voltage, the phase currents and the duty cycles at the current and
previous time steps (23 variables); the target is the phase A voltage. The data are not included in this
repository.

## Method

- **Physical references:** the ideal inverter (u = d · u_dc), a grey-box model that adds the sign of the
  phase current (dead time and switch voltage drop), and a linear regression on all the inputs.
- **Black-box models:** decision tree, pruned tree, random forest, k-nearest neighbours and fully connected
  neural networks, also tested with PCA-reduced inputs.
- **Validation:** 80 % of the data for development and 20 % held out as test set, split in blocks of 1000
  consecutive samples, because neighbouring rows share their previous-sample values. Hyperparameters are
  chosen with blocked 5-fold cross-validation; standardization and PCA are fitted on training data only.

## Results (test set, target MAE < 2 V)

| Model | Test MAE [V] |
|---|---|
| Physics (d · u_dc) | 13.46 |
| Grey box (2 inputs) | 2.97 |
| Linear regression (23 inputs) | 4.69 |
| Decision tree | **0.72** |
| Random forest | 1.10 |
| Neural network, 3 × 10 neurons (471 weights) | 1.12 |
| kNN | 2.86 |
| Neural network + PCA (10 components) | 4.29 |

- Two physically motivated inputs (grey box) beat a linear regression on all 23 inputs; its coefficients
  are about 0.99 on d · u_dc and −13 V on the sign of the current.
- The decision tree has the lowest error but about 375,000 nodes; the neural network reaches 1.12 V with
  471 weights, a better fit for an embedded implementation.
- PCA does not help here: it keeps the directions with the largest input variance, not those that matter
  for the voltage.

## Limits

The test bench ran only four constant speeds (500, 1500, 2500 and 3000 rpm). When the model has to predict
speeds missing from its training data, the error of the decision tree rises from 0.85 V to 12.53 V. The
results are therefore valid only at the tested operating points; other speeds and transients are not
validated.

## Running the notebook

The easiest way is to run it on Kaggle (link above). To run it locally, download the CSV from the dataset
page, change the path in the "Import dataset" cell, and install:

```
pip install numpy pandas scikit-learn tensorflow matplotlib seaborn joblib
```

Training results are cached in `cache/`, so a second run only reloads them.
