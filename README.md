# Automatic Modulation Classification for Low-Power IoT

A lightweight **Automatic Modulation Classification (AMC)** system for identifying digital and analog modulation schemes from wireless IQ signal samples using statistical signal features and a compact neural network.

The project is based on the methodology presented in the paper **“Automatic Modulation Classification for Low-Power IoT Applications”** by Yasmín R. Mondino-Llermanos and Graciela Corral-Briones, published in *IEEE Latin America Transactions*, Vol. 22, No. 3, March 2024. This repository implements and evaluates the core feature-engineering and lightweight neural-network approach described in the paper.

---

## Overview

Wireless communication systems use different modulation schemes to encode information onto carrier signals. **Automatic Modulation Classification** determines the modulation type of an incoming signal without requiring prior knowledge of the transmitter configuration.

This project focuses on a resource-efficient approach suitable for **low-power and resource-constrained IoT environments**.

Instead of directly processing raw IQ samples with a large deep-learning model, the system:

1. Processes IQ signal samples.
2. Extracts statistical and signal-processing features.
3. Normalizes the resulting feature vectors.
4. Classifies the modulation using a lightweight MLP neural network.
5. Evaluates classification performance across different SNR conditions.

---

## Key Features

* Automatic classification of **11 modulation schemes**
* IQ signal processing using **I/Q components**
* Statistical and signal-processing based feature extraction
* Time-domain, amplitude, phase, spectral and higher-order features
* Feature normalization using `StandardScaler`
* Lightweight **Multi-Layer Perceptron (MLP)** classifier
* Stratified train/test split
* Classification evaluation using accuracy and F1-score
* Confusion matrix analysis
* SNR-based performance analysis
* Feature importance analysis using permutation-based evaluation
* Designed around low-complexity inference for IoT-oriented applications

---

## Modulation Classes

The implementation works with the following 11 modulation classes:

| Modulation | Type    |
| ---------- | ------- |
| BPSK       | Digital |
| QPSK       | Digital |
| 8PSK       | Digital |
| QAM16      | Digital |
| QAM64      | Digital |
| CPFSK      | Digital |
| GFSK       | Digital |
| PAM4       | Digital |
| AM-DSB     | Analog  |
| AM-SSB     | Analog  |
| WBFM       | Analog  |

The dataset configuration contains SNR levels ranging from **-20 dB to +18 dB in 2 dB increments**.

---

## Dataset

The project is structured around the **RadioML 2016.10A** dataset.

### Dataset characteristics

* **11 modulation classes**
* **20 SNR levels**
* SNR range: **-20 dB to +18 dB**
* **1,000 examples** per modulation/SNR combination
* **220,000 total signal examples**
* Each signal contains **128 IQ samples**
* I and Q are represented as two separate channels

Each sample therefore contains:

```text
2 × 128 = 256 raw IQ values
```

The notebook attempts to load the original `RML2016.10a_dict.pkl` dataset. If the dataset is unavailable, it generates a synthetic dataset with the same overall structural configuration for demonstration and experimentation.

> **Important:** Results obtained using the synthetic fallback should not be interpreted as results on the original RadioML dataset.

---

## System Architecture

```text
                IQ Signal
                    │
                    ▼
        ┌───────────────────────┐
        │ Signal Preprocessing  │
        │ I / Q Components      │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │ Feature Extraction    │
        │                       │
        │ • Time-domain         │
        │ • Amplitude           │
        │ • Phase               │
        │ • Spectral            │
        │ • Higher-order stats  │
        └───────────┬───────────┘
                    │
                    ▼
             25 Features
                    │
                    ▼
        ┌───────────────────────┐
        │ StandardScaler        │
        │ Feature Normalization │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │ Lightweight MLP       │
        │                       │
        │ 25 → 25 → 12 → 11     │
        │ ReLU + Softmax        │
        └───────────┬───────────┘
                    │
                    ▼
          Predicted Modulation
```

---

## Feature Engineering

A central part of the project is replacing direct processing of raw IQ samples with a compact statistical representation.

The implementation extracts **25 features** from the signal.

### Feature categories

| Category                | Examples                                                   |
| ----------------------- | ---------------------------------------------------------- |
| Time Domain             | Mean, variance, skewness, kurtosis                         |
| Amplitude               | Envelope mean, envelope variance                           |
| Power                   | Mean power, PAPR                                           |
| Phase                   | Mean and variance of instantaneous phase                   |
| Frequency               | Mean frequency, frequency variance                         |
| Spectral                | PSD, spectral centroid, spectral spread, spectral flatness |
| Higher-Order Statistics | Cumulant-based features                                    |

This reduces the model input from:

```text
256 raw IQ values
        ↓
25 engineered features
```

The feature vector therefore contains approximately **9.8% of the original number of input values** while retaining signal characteristics useful for modulation classification.

---

## Machine Learning Model

The primary classifier is a compact **Multi-Layer Perceptron (MLP)**.

### Architecture

```text
Input Layer
25 features
     │
     ▼
Hidden Layer
25 neurons
ReLU
     │
     ▼
Hidden Layer
12 neurons
ReLU
     │
     ▼
Output Layer
11 neurons
Softmax
     │
     ▼
Modulation Class
```

### Model configuration

| Parameter            | Value          |
| -------------------- | -------------- |
| Model                | MLP Classifier |
| Input Features       | 25             |
| Hidden Layers        | 25, 12         |
| Hidden Activation    | ReLU           |
| Output               | 11 classes     |
| Output Activation    | Softmax        |
| Optimizer            | Adam           |
| Learning Rate        | 0.001          |
| Trainable Parameters | 1,105          |

The compact architecture is intended to reduce computational and memory requirements compared with large CNN-based modulation classifiers.

---

## Training Pipeline

The training workflow consists of the following stages:

```text
RadioML / Synthetic Dataset
          │
          ▼
     IQ Samples
          │
          ▼
 Feature Extraction
          │
          ▼
   25-D Feature Vector
          │
          ▼
    Train/Test Split
       80 / 20
          │
          ▼
    StandardScaler
          │
          ▼
     MLP Training
          │
          ▼
   Model Evaluation
```

The dataset is split using stratification to maintain class distribution between training and testing sets. Feature scaling is fitted only on the training data and subsequently applied to the test data to avoid data leakage.

---

## Evaluation

The implementation evaluates the classifier using:

* Accuracy
* Weighted F1-score
* Classification report
* Confusion matrix
* SNR-dependent accuracy
* Feature importance analysis

### Experimental Results

For the reported replication experiment:

| Metric                  |  Result |
| ----------------------- | ------: |
| Overall Accuracy        |  51.68% |
| Weighted F1             |  0.4990 |
| Accuracy at SNR ≥ 10 dB |  89.38% |
| Train Samples           | 176,000 |
| Test Samples            |  44,000 |
| Features                |      25 |
| Parameters              |   1,105 |

A second single-hidden-layer model was also evaluated, producing **51.88% accuracy** and a **0.5106 weighted F1-score** in the reported experiment.

> These figures are experimental results from the implementation and should not be presented as the original paper's results.

---

## SNR Analysis

Signal-to-Noise Ratio has a significant effect on modulation classification.

At lower SNR values, noise makes the characteristics of different modulation schemes harder to distinguish. The project therefore evaluates classification performance across different SNR conditions rather than reporting only a single aggregate metric.

The reported experiment achieved **89.38% accuracy for SNR ≥ 10 dB**, while overall accuracy across the complete SNR range was **51.68%**.

---

## Feature Importance

The project also performs permutation-based feature importance analysis.

The basic procedure is:

```text
Original Feature
      │
      ▼
Shuffle Feature
      │
      ▼
Evaluate Model
      │
      ▼
Measure Accuracy Drop
      │
      ▼
Estimate Feature Importance
```

A larger reduction in classification accuracy after shuffling a feature indicates that the model relies more heavily on that feature.

This provides an interpretable view of which signal characteristics contribute most to modulation classification.

---

## Visualization

The project generates visualizations for understanding the signal and model behavior, including:

* IQ signal plots
* Constellation diagrams
* Feature distributions
* Feature scatter plots
* Confusion matrices
* Feature importance plots
* SNR versus classification accuracy

For example, constellation diagrams help visualize how modulation schemes occupy different regions of the I-Q plane.

---

## Technologies Used

### Programming

* Python

### Numerical & Signal Processing

* NumPy
* SciPy

### Machine Learning

* Scikit-learn
* MLPClassifier
* StandardScaler
* Train/Test Split
* Classification Metrics

### Visualization

* Matplotlib
* Seaborn

### Data Handling

* Pickle
* Python file handling

---

## Installation

Clone the repository:

```bash
git clone https://github.com/suke2004/<repository-name>.git
cd <repository-name>
```

Create a virtual environment:

```bash
python -m venv venv
```

Activate it on Windows:

```bash
venv\Scripts\activate
```

Activate it on Linux/macOS:

```bash
source venv/bin/activate
```

Install dependencies:

```bash
pip install numpy scipy scikit-learn matplotlib seaborn requests
```

---

## Dataset Setup

Place the RadioML dataset file in the project directory:

```text
RML2016.10a_dict.pkl
```

Expected structure:

```text
project/
│
├── RML2016.10a_dict.pkl
├── notebook.ipynb
├── README.md
└── ...
```

If the dataset file is not available, the notebook can generate synthetic signal data for experimentation.

---

## Running the Project

Launch Jupyter Notebook:

```bash
jupyter notebook
```

Open the project notebook and execute the cells sequentially.

The notebook performs:

```text
Dataset Loading
      ↓
Signal Visualization
      ↓
Feature Extraction
      ↓
Feature Analysis
      ↓
Model Training
      ↓
Model Evaluation
      ↓
SNR Analysis
      ↓
Feature Importance
```

---

## Project Structure

A recommended repository structure is:

```text
automatic-modulation-classification/
│
├── README.md
├── notebooks/
│   └── amc.ipynb
│
├── data/
│   └── RML2016.10a_dict.pkl
│
├── models/
│   └── ...
│
├── figures/
│   ├── signal_gallery.png
│   ├── feature_scatter.png
│   ├── feature_importance.png
│   └── confusion_matrix.png
│
└── requirements.txt
```

---

## Research Reference

This implementation is based on:

**Yasmín R. Mondino-Llermanos and Graciela Corral-Briones**

> *Automatic Modulation Classification for Low-Power IoT Applications*

**IEEE Latin America Transactions**, Volume 22, Issue 3, March 2024.

DOI:

```text
10.1109/TLA.2024.10431424
```

The project uses the paper as the methodological reference while implementing the feature extraction and lightweight neural-network classification workflow in Python.

---

## Limitations

* Classification performance decreases under challenging low-SNR conditions.
* The synthetic-data fallback is intended for demonstration and does not replace evaluation on the original RadioML dataset.
* The implementation focuses on a lightweight feature-based approach rather than large end-to-end deep-learning architectures.
* Reported experimental metrics are specific to the implementation configuration and should not be interpreted as universal performance guarantees.

---

## Future Improvements

Potential extensions include:

* Evaluation exclusively on the original RadioML 2016.10A dataset
* Comparison with CNN and CNN-LSTM based AMC models
* Additional feature-selection techniques
* Model quantization for embedded deployment
* ONNX/TFLite deployment
* Real-time IQ stream classification
* Evaluation on additional wireless datasets
* Deployment on resource-constrained IoT hardware

---

## License

This project is intended for educational and research purposes.

If you use the methodology or datasets associated with the referenced work, follow their respective licensing and attribution requirements.

---

## Author

**Mahalaxmi Karnti**

Electronics and Communication Engineering
NIT Trichy

GitHub: `https://github.com/KarnatiMahalaxmi`

---

## Acknowledgements

* The authors of the referenced IEEE research paper
* The creators of the RadioML 2016.10A dataset
* The open-source Python scientific computing and machine-learning ecosystem
