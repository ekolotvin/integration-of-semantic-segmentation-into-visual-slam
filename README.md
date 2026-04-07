# Integration of Semantic Segmentation into Visual SLAM
![11](https://github.com/user-attachments/assets/2cf86d77-cca1-4169-b44c-926d316dd18f)
![Снимок экрана 2026-02-28 010250](https://github.com/user-attachments/assets/2525095e-fda9-4f1d-9fc8-0d6ff1241c7a)
## In this project a semantic segmentation model is integrated into a visual SLAM pipeline to build a three-dimensional semantically annotated scene.
For each input frame:<br>
1. A semantic mask is predicted;<br>
2. The mask is integrated into the SLAM pipeline;<br>
3. A 3D map of the scene with semantic labels is constructed.

## Project Overview
The project is divided into two main parts:

<ins>Part 1 — Semantic Segmentation:</ins><br> 
Training and comparison of DeepLabV3+ models on various road datasets.<br>
Datasets used:<br>
1) [The KITTI Vision Benchmark Suite (KITTI: Semantic Segmentation Evaluation)](https://www.cvlibs.net/datasets/kitti/eval_semseg.php?benchmark=semantics2015);<br>
2) [Berkeley Deep Drive (BDD100K: 10K images + Segmentation)](http://bdd-data.berkeley.edu/download.html);<br>
3) Combined dataset (KITTI + BDD100K).<br>

<ins>Part 2 — Integration into Visual SLAM:</ins><br>
Application of the final segmentation model to build a 3D scene with semantic annotations;<br>
For 3D scene reconstruction, the dataset used is<br>
[The KITTI Vision Benchmark Suite (Visual Odometry / SLAM Evaluation 2012)](https://www.cvlibs.net/datasets/kitti/eval_odometry.php).

## 1. Semantic Segmentation
**KITTI Dataset.**<br> 
200 annotated images were used;<br>
10 unified classes were defined:<br>
*road, sidewalk, building, vegetation, sky, car, truck, bus, person,pole, traffic sign, terrain*;<br>
Class remapping was performed to ensure consistency across datasets.

<ins>Data Split:</ins><br>
160 — training;<br>
40 — validation.<br>

<ins>kitti_model training</ins>.<br> 
Architecture: DeepLabV3+;<br> 
Fine-tuning of the classification layer;<br> 
50 training epochs;<br> 
The best *kitti_model* was saved based on the mIoU validation metric;<br> 
After training inference was performed on KITTI test images.

**BDD100K Dataset (10K images).**<br>
10,000 annotated images were used;<br>
After filtering and remapping the same 10 classes as in the KITTI dataset were retained.<br>

<ins>Data Split:</ins><br>
5500 — training;<br>
1000 — validation;<br>
1500 — test.

<ins>Training of bdd_model</ins>.<br>
Initialization of DeepLabV3+;<br>
Fine-tuning of the classification layer;<br>
50 training epochs;<br>
The best *bdd_model* was saved based on the mIoU validation metric.<br>

<ins>Cross-Domain Evaluation</ins>.<br>
To analyze generalization capability *kitti_model* was tested on the BDD100K test split;<br>
*bdd_model* was evaluated on its own test set;<br>
The experiment demonstrates the impact of domain shift between datasets.

**Combined Dataset.**<br>
KITTI and BDD100K were merged into a single dataset.<br>

<ins>Data Split:</ins><br>
6560 — training;<br>
820 — validation;<br>
820 — test.

<ins>Training of general_model</ins>.<br>
Initialization of DeepLabV3+;<br> 
Fine-tuning of the classification layer;<br>
50 training epochs;<br>
The best *general_* was saved based on the mIoU validation metric.

**Model comparison and final model selection.**<br>
The following models were evaluated on the test split of the combined dataset:<br>
*kitti_model*(mIoU = 0,468), 
*bdd_model*(mIoU = 0,776), 
*general_model*(mIoU = 0,673).<br>
The selection criteria for the *final_model* were a high mIoU metric, as well as the quality of visual performance on image sequences from the KITTI Odometry Dataset.<br>
*general_model* was selected as the final model, as its visual quality during inference is noticeably higher, while its mIoU value is only slightly lower than that of *bdd_model*.<br>

*bdd_model* (inference):
![bdd_model](https://github.com/user-attachments/assets/ccc3534b-1b7d-48d2-8453-8ec2f54acbfd)

*general_model* (inference):
![general_model](https://github.com/user-attachments/assets/504eb55c-bdcc-4d05-8b6e-fbea62057e07)
## 2. Integration into Visual SLAM
**3D Semantic Map Construction.**<br>
For each frame:<br>
<ins>A semantic mask is predicted.</ins><br>
The RGB image is fed into the trained *final_model*;<br>
The output is a mask where each pixel is assigned a class.<br>

<ins>Transformation of LiDAR points into the camera coordinate system.</ins><br>
LiDAR measures 3D points in its own coordinate system;<br>
To align them with the image, the points are transformed into the camera coordinate system using the calibration parameters *calib*;<br>

<ins>The LiDAR point cloud is projected onto the image.</ins><br>
For each 3D point its spatial position and the corresponding class from the semantic mask can be determined.

<ins>Assigning semantics to 3D points.</ins><br>
After projection each 3D point takes the class of the pixel it falls onto;<br>
The point is assigned a color according to this class.<br>

The standard geometric reconstruction of the scene is augmented with semantic information.

**Global 3D map construction.**<br>
<ins>To build a complete map of the scene:</ins>

  1. The camera pose estimated by SLAM is used;<br>
  2. All local point clouds are transformed into a unified global coordinate system;<br>
  3. The point clouds are sequentially merged.<br>

<ins>As a result a global 3D map of the scene surface with semantic annotations is obtained:</ins>

![1](https://github.com/user-attachments/assets/7b0e49f2-b7bd-4c24-b12e-a97eb3bb9766)<br>




