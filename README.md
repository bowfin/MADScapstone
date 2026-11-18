# Data-Driven Fault Detection and Diagnosis of HVAC Chiller Plants Using Multivariate Time-Series Data

## Research Objective

This project investigates the use of data-driven methods for **fault detection and diagnosis (FDD) of HVAC chiller plants** using multivariate time-series data. The study aims to determine whether operational measurements can distinguish normal from faulted chiller-plant operation and identify different fault types and, where applicable, fault severity.

## Research Questions

1.  Can classification models distinguish fault-free from faulted operation with high accuracy and a low false-alarm rate?
2.  Can models diagnose which fault category (sensor bias, valve bypass, fouling, cooling-tower PI, etc.) is present?
3.  Does model performance depend on fault severity? Severity was not evaluated as a separate prediction target, so this question is identified as future work.

## Data Source(s)

The primary data source is the **Lawrence Berkeley National Laboratory (LBNL) Fault Detection and Diagnostics (FDD) simulated chiller plant dataset.
* LBNL FDD Dataset: https://faultdetection.lbl.gov/dataset/simulated-chiller-plant/
* DOI: 10.25984/1881324
The dataset contains simulated one-minute chiller-plant operating data representing normal operation and multiple equipment and control fault scenarios.

## Repository Organization

├── README.md  
├── data/  
|   ├── raw/                    # LBNL CSV files (download; not tracked)  
├── notebooks/  
|   ├── 1_DataCleaning_Transformation.ipynb  
|   ├── 2_RawDataAssessment_.ipynb  
|   ├── 3_FeatureSelection_2.ipynb  
|   ├── 4_ExploratoryDataAnalysis.ipynb  
|   ├── 5_StatsVisualizations.ipynb  
|   ├── 6_Baseline2.ipynb  
|   ├── 7_Models.ipynb  
|   ├── 8_Hyperparameters.ipynb  
|   ├── 9_Test_Evaluation_Analysis.ipynb  

## Python Version

Python 3.13.5. Two conda environments were used:
*  ds: data preparation, exploratory analysis, and the Random Forest baseline
*  torch: the LSTM and 1D-CNN (GPU, CUDA 12.6)

## Required Packages

|Package	               |    Purpose                 |
|---|---|
pandas	                 |     data handling
numpy	                   |     numerical computing
pyarrow	parquet          |     read/write
scikit-learn	           |     Random Forest, metrics
PyTorch (CUDA 12.6 build)|	    LSTM and 1D-CNN
matplotlib / seaborn	   |     figures
jupyter	                 |     running notebooks

## Installation and Setup

# 1. Clone the repository
git clone https://github.com/bowfin/MADScapstone.git
cd MADScapstone

# 2. Environment for data preparation and the Random Forest baseline  
conda create -n ds python= 3.13.5  
conda activate ds  
pip install pandas numpy pyarrow scikit-learn matplotlib seaborn jupyter

# 3. Environment for the sequence models (NVIDIA GPU, CUDA 12.6)  
conda create -n torch python= 3.13.5  
conda activate torch  
pip install torch --index-url https://download.pytorch.org/whl/cu126  
pip install pandas numpy pyarrow scikit-learn matplotlib seaborn jupyter  

# 4. Download the LBNL dataset (see above) into data/raw/

Hardware used for the reported results: Windows laptop, NVIDIA GeForce RTX 3060 Laptop GPU (about 6 GB VRAM).  
Because of memory limits, the sequence-model notebooks read only selected parquet columns, use float32, subsample the training set, and build feature tensors per batch.  
A CPU-only run is possible but slow.

## How to Run the Analysis

Run the notebooks in order.  
1_DataCleaning_Transformation.ipynb: 	Loads the scenario CSV files, standardizes formats, attaches scenario and fault labels, and writes the cleaned data (parquet) used by the later notebooks.  
2_RawDataAssessment_.ipynb: Audits data quality: missing values, duplicates, constant columns, value ranges, and interquartile-range outlier screening. Shows that apparent outliers largely reflect operating regime (chiller staging), not faults.  
3_FeatureSelection_2.ipynb: sAudits data quality: missing values, duplicates, constant columns, value ranges, and interquartile-range outlier screening. Shows that apparent outliers largely reflect operating regime (chiller staging), not faults.  
4_ExploratoryDataAnalysis.ipynb: Explores the retained features: class distribution, fault-conditional summaries, staging-state behavior, and time-series patterns. Builds engineered features such as setpoint tracking error, evaporator delta-T, and the staging indicator.  
5_StatsVisualizations.ipynb: 	Statistical checks and visualizations: distribution shapes, normality and variance assumptions, and the tests that justify the modeling choices.  
6_Baseline2.ipynb]: Random Forest baseline on the chronological split. Compares the three feature-set variants (full, suspect columns removed, regime-normalized) and diagnoses feature importance.  
7_Models.ipynb: 	Builds the 240-minute windowed dataset and defines, trains, and compares the preliminary LSTM and 1D-CNN with class-weighted loss and early stopping.  
8_Hyperparameters.ipynb: Staged random-search hyperparameter tuning of the LSTM and 1D-CNN, selected on validation macro F1.  
9_Test_Evaluation_Analysis.ipynb: Final scoring of all models on the held-out test months: classification reports, confusion matrices, regime-conditioned results, and aggregate comparison across training, validation, and test.

## How to Reproduce the Major Results

Key settings are fixed so the experiments can be repeated:
*  Chronological partition: January to August (training), September to October (validation), November to December (test)
*  Random seed: 42
*  Class-weighted loss and early stopping (patience = 5 epochs on validation loss) for the sequence models
*  Sampling caps: 500,000 rows for the Random Forest; up to 15,000 windows per class for the sequence models
*  Windows: 240 minutes, 15-minute stride

##  How to Reproduce the Important Figures

|Figure	     |          Description	              |               Notebook
|---|---|---|
Figure 2.1	  | Analytical workflow diagram	   |  Page 18 of Manuscript  
Figure 5.4	  | Class distribution	           |  4_ExploratoryDataAnalysis.ipynb  
Figure 7.1-.2	| Confusion matrices (test set)	 |  9_Test_Evaluation_Analysis.ipynb  
Figure 8.1    | Macro F1 scores                |  9_Test_Evaluation_Analysis.ipynb  
Figure 8.2    | F1 score by fault category     |  9_Test_Evaluation_Analysis.ipynb  

## Citation Information

  author:       Oldham, Tim,  
  title:        Data-Driven Fault Detection and Diagnosis of {HVAC} Chiller Plants Using Multivariate Time-Series Data,  
  year:         2026,  
  howpublished: Data Science Capstone, Mississippi State University,  
  url:          https://github.com/bowfin/MADScapstone
