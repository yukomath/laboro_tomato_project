# Tomato Ripeness Classification

This project implements a tomato ripeness classification system using Mask R-CNN. The model detects tomatoes in images and classifies them into six ripeness categories. The dataset used is from [Laboro Tomato Dataset](https://github.com/laboroai/LaboroTomato), and the project covers data preparation, model training, evaluation, and a web application for inference.


## 1. Data Preparation and Training

**Notebook:** <a href="https://colab.research.google.com/drive/1T378e23B6nI3bIUCm63hTady4dFLfbjg">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" />
</a>

This notebook prepares the dataset and trains a Mask R-CNN model for tomato ripeness classification.


### Dataset
- Images and COCO-style JSON annotations from the  
  [Laboro Tomato Dataset](https://github.com/laboroai/LaboroTomato), stored on Google Drive.
- The dataset consists of:
  - **643 images** for training
  - **161 images** for testing
- Images in the **training folder** are further split into **training and validation sets**.
- **EXIF orientation correction** is applied during image loading to ensure correct image alignment (critical for stable training).



### Classes
Six ripeness classes for regular (`b`) and cherry (`l`) tomatoes:
- `b_fully_ripened`
- `b_half_ripened`
- `b_green`
- `l_fully_ripened`
- `l_half_ripened`
- `l_green`

### Data Exploration
- Number of images and annotations
- Annotation structure
- Image resolutions
- Class distribution

### Dataset Split
- 80% training / 20% validation
- Fixed random seed for reproducibility

### Custom Dataset
A custom `TomatoDataset` class handles:
- Image loading and preprocessing
- Annotation parsing
- Mask generation
- Image resizing　to **800 × 800**
- Optional data augmentation (flip and rotation)

### Data Augmentation
- Horizontal flip and rotation were initially evaluated.
- Rotation augmentation degraded mAP performance.
- Horizontal flip showed minimal impact on performance.
- **As a result, no data augmentation is used in the final model.**

### Model
- Pretrained **Mask R-CNN** with a **ResNet-50 FPN** backbone
- Classification and mask heads modified for 6 ripeness classes

### Training
- Training is performed for **10 epochs**.
- Optimizer: SGD with momentum and weight decay
- Learning rate scheduler: StepLR
- Checkpointing system to resume training and save the best-performing model

### Evaluation
- COCO-style metrics during validation:
  - Bounding box mAP
  - Segmentation mask mAP

### Results
- Final (10th) epoch performance:
  - **Training loss:** `0.3673`
  - **Validation bounding box mAP:** `0.6710`
  - **Validation segmentation mask mAP:** `0.6832`

*Note: The loss is computed on the training set, while mAP metrics are evaluated on the validation set.*

- Training loss and validation mAP are plotted over epochs.
  <img width="989" height="390" alt="image" src="https://github.com/user-attachments/assets/119f55da-0292-42bc-a93f-72eb86def7c3" />


### Output
- The best-performing model checkpoint is saved and used for test evaluation and deployment.
- **Best model:**  
  [Download link to best model](https://drive.google.com/file/d/188HODfp_RYNznEYvKf0b48vskUmnbdR5/view?usp=share_link)




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
4. After starting, the terminal will display the URL to access the app, e.g.:
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

