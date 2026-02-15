# Tomato Ripeness Classification

This project implements a tomato ripeness classification system using Mask R-CNN. The model detects tomatoes in images and classifies them into six ripeness categories. The dataset used is from [Laboro Tomato Dataset](https://github.com/laboroai/LaboroTomato), and the project covers data preparation, model training, evaluation, and a web application for inference.


## 1. Data Prepataion and training
Link of Notebook
<a href="https://colab.research.google.com/drive/1T378e23B6nI3bIUCm63hTady4dFLfbjg">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" />
</a>

The notebook prepares the dataset and trains the Mask R-CNN model:

Dataset: Uses images and COCO-style JSON annotations from LaboroTomato, stored on Google Drive.

Classes: 6 ripeness classes for regular (b) and cherry (l) tomatoes:
b_fully_ripened
b_half_ripened
b_green
l_fully_ripened
l_half_ripened
l_green

Data Exploration: Checks number of images, annotation structure, image resolutions, and class distribution.

Dataset Split: 80% training / 20% validation with fixed random seed for reproducibility.

Custom Dataset: TomatoDataset handles image loading, annotation processing, mask generation, resizing, and safe augmentation (horizontal flip).

Model: Pretrained Mask R-CNN (ResNet-50 FPN backbone) with modified classification and mask heads for 6 classes.

Training:
Optimizer: SGD with momentum and weight decay

Learning rate scheduler: StepLR

Checkpointing system to resume training and save best model

Evaluation: Uses COCO-style metrics (bounding box mAP and mask mAP) during validation.

Output: Plots training loss and validation mAP over epochs.

The notebook saves the best-performing model for further evaluation.


## 2. Test Evaluation
<a href="https://colab.research.google.com/drive/1FauT_oj0B2qi6jVYM8_SDD9yq9yuB2Oc">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" />
</a>

This notebook evaluates the trained model on the held-out test dataset:
Loads test images, COCO-style annotations, and the best trained checkpoint.
Applies consistent preprocessing and dataset handling as in training.
Computes COCO-style metrics:
Bounding box mAP
Segmentation mask mAP
Provides class-wise and object-size-wise performance analysis.

## 3. Visual Check
<a href="https://colab.research.google.com/drive/16I_pJ5Baph34w2O2cy7T77NDGboPzF7s">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" />
</a>

This notebook provides a visual inspection of the model predictions:
Displays sample images from test set.
Overlays predicted bounding boxes and masks with confidence scores.
Allows quick qualitative verification of model performance.

## 4. Web App
A FastAPI-based web application can serve the trained model for real-time tomato ripeness classification:
Users upload images.
The app returns predicted ripeness for detected tomatoes with bounding boxes and masks.
Useful for farm monitoring, sorting, or quality control workflows.

This section explains how to run the Tomato Ripeness Classification model via a simple web application.

### How to Use

1. **Prepare the code and model**  
   Download the `tomato_app` folder from [Google Drive](https://drive.google.com/drive/folders/1M5ch-lMnK7btZoiY79pVhrRTZzBJu8jl?usp=share_link).  
   This folder contains all the necessary files for running the web application.

2. **Navigate to the app folder in terminal**
   ```bash
   cd tomato_app

3. Start the web application
Run the following command:
```bash
uvicorn app:app --reload
```
4. After starting, the terminal will display the URL to access the app,:
```bash
http://127.0.0.1:8000
```
Open the displayed URL in your browser.

5. Upload tomato images to see the predicted ripeness classes.

### Notes
Make sure the trained model file (best_model_6class.pth) is present in the app folder.
Ensure your Python environment has all required libraries installed (torch, torchvision, Pillow, FastAPI, etc.).

### Additional Notes 
All notebooks are designed to run in Google Colab with mounted Google Drive for data storage.
Training is checkpoint-resumable for robust experimentation.
The pipeline supports scalable evaluation and deployment-ready model export.

### Future Work 
The model shows lower accuracy for small bounding boxes, indicating potential for improvement.

