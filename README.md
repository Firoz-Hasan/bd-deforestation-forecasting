bd-deforestation-forecasting
==========================

Repository purpose
------------------
This repository contains the code and notebooks used for:
1) Tile-based forest / non-forest segmentation from Landsat imagery (U-Net variants).
2) Annual forest-cover time-series extraction and short-term deforestation forecasting using
   ARIMA, LSTM, and ConvLSTM models.
3) Google Earth Engine (GEE) preprocessing scripts for cloud-free annual composites,
   Landsat 7 SLC-off gap filling, and Landsat 8 → Landsat 7 normalization (PIF regression).

It is intended to support full reproducibility of the experiments reported in the paper:
“Cross-region transfer evaluation of tile-based forest segmentation and annual deforestation
forecasting using Landsat time series in Bangladesh.”


Repository contents (uploaded files)
------------------------------------
Notebooks (Python):
- Split_Images_Into_Tiles_rgb.ipynb
  Creates 256×256 tiles from imagery and aligns masks for training/evaluation.
- Calculate_Forest_Percentage.ipynb
  Computes annual forest cover (%) from predicted masks and/or ground-truth masks.
- train_vgg16_model.ipynb, vgg16_unet.ipynb, VGG16_Unet (1).ipynb, Vgg16_Unet_Model_6_Bands.ipynb
  VGG16-based U-Net training/evaluation (RGB and 6-band variants).
- densenet_unet.ipynb
  DenseNet-based segmentation model experiments.
- Kfold_cross_validation.ipynb, Cross_Validation_Check.ipynb
  K-fold cross-validation utilities and checks.
- Benchmark_Inference.ipynb
  Inference benchmarking (sec/tile, sec/km², throughput).
- gradcam_interpritibility.ipynb, Custom_Unet_gradcam_segmentation.ipynb
  Interpretability (Grad-CAM) and error-focused Grad-CAM variants.
- interpolation_check.ipynb
  Checks / experiments related to temporal interpolation steps.
- Environment_report.ipynb
  Captures the Python / CUDA / driver environment for reproducibility.
- LSTM_Model.ipynb, ConvLstm_Model.ipynb
  Forecasting models using LSTM and ConvLSTM.
- Arima_Model.ipynb
  Statistical forecasting baseline (ARIMA).

Google Earth Engine (GEE) scripts:
- annualmediancomposite.js
  Cloud-free annual median composites from Landsat 7 Collection 2 Level-2 SR, using QA masks
  and a scene-level cloud cover filter (≤20%). Optical scaling uses the C2 L2 scale/offset:
  SR * 0.0000275 + (-0.2).
- SLC_OFF.js
  Hybrid Landsat 7 SLC-off gap filling: spatial fill (reduced kernel) + temporal mean fill,
  using ≤20% cloud cover and QA masking.
- PIF based Normalization.txt
  Full GEE code for PIF-based RGB normalization Landsat-8 (OLI) → Landsat-7 (ETM+),
  including before/after bias/RMSE/variance and Q–Q percentile export.

Other:
  Notes on how Grad-CAM outputs were generated and how to use them in the paper.


Data assumptions and folder layout
----------------------------------
The notebooks assume a tile dataset layout like:

datasets_temp/
  images/     (GeoTIFF or arrays; either RGB or multi-band)
  masks/      (binary masks aligned to images)
  ... (optional outputs)

Common settings used in the notebooks:
- Resolution: 30 m (Landsat).
- Tile size: 256 × 256.
- Typical band order for 6-band experiments: [R, G, B, NIR, SWIR1, SWIR2]
  (confirm in your own data loader if you stored a different order).


Installation
------------
1) Create and activate a clean environment (recommended):
   conda create -n bddeforest python=3.10 -y
   conda activate bddeforest

2) Install dependencies:
   pip install -r requirements.txt

Notes for Windows users:
- Some packages (notably rasterio) can be easier to install via conda-forge:
  conda install -c conda-forge rasterio


Quickstart (end-to-end)
-----------------------
A) Preprocess Landsat in GEE
1) Use annualmediancomposite.js to generate cloud-free annual composites.
2) Use SLC_OFF.js to fill Landsat 7 SLC-off gaps where needed.
3) Use PIF based Normalization.txt to normalize Landsat 8 RGB to the Landsat 7 reference.

Export the resulting GeoTIFFs (30 m, cloud-optimized is fine).

B) Create tiles + masks
Run:
- Split_Images_Into_Tiles_rgb.ipynb
This will create / load 256×256 tiles and aligned masks for model training.

C) Train segmentation model
Run one of:
- Vgg16_Unet_Model_6_Bands.ipynb (6-band U-Net)
- train_vgg16_model.ipynb or vgg16_unet.ipynb (RGB experiments)
- densenet_unet.ipynb (DenseNet variant)

D) Evaluate + interpretability (Grad-CAM)
Run:
- Custom_Unet_gradcam_segmentation.ipynb or gradcam_interpritibility.ipynb
These produce Grad-CAM overlays and error-focused Grad-CAM panels.

E) Annual forest-cover time series + forecasting
1) Compute yearly forest cover (%):
   - Calculate_Forest_Percentage.ipynb
2) Forecast forest cover:
   - Arima_Model.ipynb
   - LSTM_Model.ipynb
   - ConvLstm_Model.ipynb


Reproducibility notes (seeds and environment)
---------------------------------------------
Set seeds at the beginning of your notebooks/scripts to reduce run-to-run variance:

import os, random, numpy as np, tensorflow as tf
seed = 42
os.environ["PYTHONHASHSEED"] = str(seed)
random.seed(seed)
np.random.seed(seed)
tf.random.set_seed(seed)

GPU operations may still be non-deterministic depending on CUDA/cuDNN kernels.

Environment details should be recorded in:
- Environment_report.ipynb

Example environment used in the paper experiments (workstation):
- OS: Windows 11
- Python: 3.10 (Anaconda)
- TensorFlow: 2.13.0
- GPU: NVIDIA GTX 1650 (4 GB VRAM), system RAM: 48 GB
- NVIDIA driver: 591.44
- CUDA runtime reported by nvidia-smi: 13.1


How to cite
-----------
If you use this repository, please cite the associated paper (add final citation here once accepted),
and include the repository URL and release tag (e.g., v1.0.0) used for your experiments.
