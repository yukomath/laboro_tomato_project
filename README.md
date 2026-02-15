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
<img width="6288" height="3992" alt="laboro_tomato_exp2" src="https://github.com/user-attachments/assets/ce91bf0f-e9c0-49a0-bb19-8366f62f5063" />
pictures by [Laboro Tomato Dataset](https://github.com/laboroai/LaboroTomato?tab=readme-ov-file)

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
**Notebook:**　<a href="https://colab.research.google.com/drive/1FauT_oj0B2qi6jVYM8_SDD9yq9yuB2Oc">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" />
</a>

This notebook evaluates the trained 6-class Mask R-CNN model on a held-out test dataset using COCO-style metrics.

### Test Dataset
- **161 test images**, separated from the training data
- COCO-style JSON annotations
- The same preprocessing as training/validation is applied to ensure consistency:
  - EXIF orientation correction
  - Image resizing to **800 × 800**
  - Bounding boxes and segmentation masks are scaled accordingly

### Test Dataset Definition
A custom `TomatoDataset` class is used for the test set, mirroring the training dataset implementation:
- Image loading and preprocessing
- Annotation parsing
- Bounding box and mask generation
- Image, box, and mask resizing to **800 × 800**
- Output format compatible with Mask R-CNN

### Model Loading
- Mask R-CNN with a ResNet-50 FPN backbone
- Classification and mask heads configured for **6 classes + background**
- The best-performing checkpoint selected during validation is loaded
- Model is evaluated in inference mode (`model.eval()`)

### Evaluation Metrics
The model is evaluated using **COCO-style metrics**, including:
- Bounding box Average Precision (bbox mAP)
- Segmentation mask Average Precision (mask mAP)
- Metrics averaged over IoU thresholds **0.50–0.95**
- Performance analyzed by **object size** (small / medium / large)

### Overall Test Results

**Bounding Box (bbox) mAP**
- AP @[IoU=0.50:0.95]: **0.623**
- AP @[IoU=0.50]: **0.816**
- AP @[IoU=0.75]: **0.703**

**Segmentation Mask mAP**
- AP @[IoU=0.50:0.95]: **0.637**
- AP @[IoU=0.50]: **0.807**
- AP @[IoU=0.75]: **0.720**

### Class-wise Average Precision (Test Set)

| Class              | AP     |
|--------------------|--------|
| b_fully_ripened    | 0.3584 |
| b_half_ripened     | 0.3169 |
| b_green            | 0.1504 |
| l_fully_ripened    | 0.1637 |
| l_half_ripened     | 0.2464 |
| l_green            | 0.0651 |


### Performance by Object Size (Bounding Box)

| Object Size | AP     | COCO Area Range (px²) | Approx. Area Ratio (800×800 image) |
|-------------|--------|------------------------|-------------------------------------|
| Small       | 0.0138 | < 1,024                | < 0.16%                             |
| Medium      | 0.1688 | 1,024 – 9,216          | 0.16% – 1.44%                       |
| Large       | 0.3250 | > 9,216                | > 1.44%                             |

*Object sizes follow the COCO definition　(small / medium / large).  
Area ratios are computed relative to the resized image resolution (800 × 800 = 640,000 px²).*


### Observations
- The model performs well on **medium and large tomatoes**.
- Performance on **small objects** is significantly lower, particularly for cherry tomatoes.
- This indicates that **small bounding box detection remains a key challenge** and an important area for future improvement.
  

## 3. Visual Check
**Notebook:** <a href="https://colab.research.google.com/drive/16I_pJ5Baph34w2O2cy7T77NDGboPzF7s">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" />
</a>

This notebook performs a qualitative inspection of the trained model by visualizing prediction results on test images.

Using the **best-performing model obtained during training**, 10 images are randomly sampled from the test dataset and inference results are visualized to assess model behavior beyond numerical metrics.

### Visualization Details
For each selected test image, the following prediction results are overlaid:
- Bounding boxes
- Segmentation mask contours
- Predicted class labels
- Confidence scores

All images are resized to **800 × 800**, consistent with the training and evaluation settings. Only predictions above a predefined confidence threshold are displayed.

<img width="300" alt="Unknown" src="https://github.com/user-attachments/assets/cc9825ea-afd7-401b-a3ac-de4b295dd448" />
<img width="300" alt="Unknown-1" src="https://github.com/user-attachments/assets/cf0db4ba-f9b2-4edc-8d29-65a371a0d8ed" />
<img width="300" alt="Unknown-2" src="https://github.com/user-attachments/assets/226ba178-aced-41b3-905d-d2de9af7a769" />
<img width="300" alt="Unknown-3" src="https://github.com/user-attachments/assets/c5c799aa-fc2d-4376-80bd-c46ca7083e13" />
<img width="300" alt="Unknown-4" src="https://github.com/user-attachments/assets/9dd7a709-154b-48b2-a645-a64815ca1541" />
<img width="300" alt="Unknown-5" src="https://github.com/user-attachments/assets/9dcdb930-78e7-409b-b7c0-34a71c38a70e" />
<img width="300" alt="Unknown-6" src="https://github.com/user-attachments/assets/0277a7a8-6572-4075-a73d-f3317952758b" />
<img width="300" alt="Unknown-7" src="https://github.com/user-attachments/assets/c47ae38e-08ff-412d-8e13-aafc3909098c" />
<img width="300" alt="Unknown-8" src="https://github.com/user-attachments/assets/a565509f-6d6c-422d-bd51-a28ada3046d9" />
<img width="300" alt="Unknown-9" src="https://github.com/user-attachments/assets/e2c073f2-11e1-4aa6-a9a6-b67cff6d82ae" />





Observations from Visual Inspection
- **Small tomatoes are sometimes missed**, consistent with the lower AP observed for small objects in the quantitative evaluation.  
- **Ripeness stages are generally predicted correctly**, but in some images both regular (b_*) and cherry (l_*) tomatoes appear together, making it difficult to clearly distinguish tomato type.  
- To build a model that reliably classifies **both ripeness and tomato type (regular vs cherry)** in mixed images, further improvements may be needed, such as handling small objects more accurately and incorporating type-specific features.


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

