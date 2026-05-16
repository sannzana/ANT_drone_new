<h2><strong>Task-01: Dataset Understanding & Preprocessing</strong></h2>

<h3><strong>Dataset Overview</strong></h3>

<p>
This project uses the <strong>VisDrone Dataset</strong>, a drone-based aerial image dataset for object detection and tracking. The dataset contains high-resolution drone images captured in urban and suburban environments. It includes objects such as pedestrians, people, cars, vans, trucks, bicycles, buses, motorcycles, and other traffic-related objects.
</p>

<p>
The dataset used in this project is already converted into <strong>YOLO format</strong>, so each image has a corresponding <code>.txt</code> label file.
</p>

<p>Each label line follows this YOLO format:</p>

<pre><code>class_id x_center y_center width height</code></pre>

<h3><strong>Original Classes</strong></h3>

<pre><code>0: pedestrian
1: people
2: bicycle
3: car
4: van
5: truck
6: tricycle
7: awning-tricycle
8: bus
9: motor</code></pre>

<p>
The assessment requires detecting <strong>humans</strong> and <strong>cars</strong>. Therefore, only the required classes were selected.
</p>

<p>The class mapping used in this project is:</p>

<pre><code>Original class 0: pedestrian → New class 0: human
Original class 1: people     → New class 0: human
Original class 3: car        → New class 1: car</code></pre>

<p>
All other classes were removed from the training and validation labels.
</p>

<h3><strong>Final Classes</strong></h3>

<pre><code>0: human
1: car</code></pre>

<h2><strong>Task-02: Model Training & Evaluation</strong></h2>

<h3><strong>1. Environment Setup and Dataset Configuration</strong></h3>

<p>
A YOLO-compatible <code>data.yaml</code> file was created after preprocessing the dataset. This file defines the training and validation image paths and the two final classes used in this project.
</p>

<pre><code>path: /kaggle/working/drone_human_car_dataset
train: images/train
val: images/val

names:
  0: human
  1: car</code></pre>

<p>
This configuration is required so that YOLO can locate the dataset and correctly map class IDs to class names.
</p>

<h3><strong>2. Model Selection and Training</strong></h3>

<p>
A pre-trained <strong>YOLO11m</strong> model was used and fine-tuned on the processed VisDrone dataset. YOLO was selected because it is fast, suitable for real-time detection, and effective for object detection tasks in drone imagery.
</p>

<pre><code>Model: YOLO11m
Image size: 960
Epochs: 50
Batch size: 8
Classes: human, car
Training type: Transfer learning / fine-tuning</code></pre>

<p>
The model was trained using filtered training images and labels. Validation was done using filtered validation images and labels.
</p>

<h3><strong>3. Training Outputs</strong></h3>

<p>
After training, the following main files were generated:
</p>

<pre><code>best.pt
last.pt
results.png
confusion_matrix.png
PR_curve.png
P_curve.png
R_curve.png
F1_curve.png</code></pre>

<p>
The <code>best.pt</code> file was used for final prediction, human counting, tiled detection, heatmap visualization, and test evaluation.
</p>

<h3><strong>4. Validation Results</strong></h3>

<p>
The trained model was evaluated on the validation dataset. The validation result showed that car detection performed better than human detection because cars are larger and easier to detect in aerial images.
</p>

<pre><code>Overall Precision: 0.8197
Overall Recall: 0.7182
Overall mAP50: 0.7124
Overall mAP50-95: 0.4505

Human mAP50: 0.602
Car mAP50: 0.822</code></pre>


<h2><strong>Task-03: Human & Car Detection with Human Counting</strong></h2>

<h3><strong>Detection Approach</strong></h3>

<p>
After training, the saved YOLO model <code>best.pt</code> was loaded and used for inference on test images. The system detects two classes:
</p>

<pre><code>0: human
1: car</code></pre>

<p>
For each test image, the model predicts bounding boxes, confidence scores, and class IDs. The bounding boxes are drawn on the image, and the detected class name and confidence score are displayed beside each box.
</p>

<h3><strong>Human and Car Counting</strong></h3>

<p>
The counting logic is simple. After detection, the system checks the class ID of every predicted bounding box.
</p>

<pre><code>If class_id == 0:
    human_count += 1

If class_id == 1:
    car_count += 1</code></pre>

<p>
The final output image displays both the bounding boxes and the total count.
</p>

<pre><code>Humans: X | Cars: Y</code></pre>

<h3><strong>Generated Detection Outputs</strong></h3>

<p>
The following output folders were generated during testing:
</p>

<pre><code>test_output_1_normal_yolo
test_output_2_class_aware_hybrid
test_output_3_always_tiled_yolo
test_output_4_density_heatmap</code></pre>

<p>
The normal YOLO output shows standard full-image detection. The class-aware hybrid output is used as the main final method. The always-tiled YOLO output is included for comparison.
</p>

<h3><strong>Saved Count Files</strong></h3>

<p>
The human and car counts were saved in CSV files:
</p>

<pre><code>test_normal_counts.csv
test_hybrid_counts.csv
test_tiled_counts.csv
test_density_counts.csv
test_detection_comparison.csv</code></pre>

<p>
The <code>test_detection_comparison.csv</code> file compares normal YOLO, class-aware hybrid detection, and always tiled YOLO using human count, car count, and inference time.
</p>
<h2><strong>Task-04: Optional Creative Improvement</strong></h2>

<h3><strong>Class-Aware Hybrid Detection</strong></h3>

<p>
Instead of using only normal YOLO detection, a class-aware hybrid detection method was implemented as a creative improvement. The validation result showed that car detection was stronger than human detection. Therefore, the system handles cars and humans differently.
</p>

<pre><code>Cars:
    detected mainly using normal YOLO

Humans:
    detected using normal YOLO + tiled YOLO

Final step:
    merge boxes using Non-Maximum Suppression</code></pre>

<p>
This approach was used because cars are larger and easier to detect in aerial images, while humans are often very small. Tiled detection helps improve the chance of detecting small humans.
</p>

<h3><strong>Tiled YOLO Detection</strong></h3>

<p>
Tiled detection splits the input image into smaller overlapping sections. YOLO is then applied to each tile separately. The detected boxes are mapped back to the original image and merged using Non-Maximum Suppression.
</p>

<pre><code>Input image
→ Split into overlapping tiles
→ Run YOLO on each tile
→ Convert tile boxes back to original image coordinates
→ Apply NMS
→ Show final boxes and counts</code></pre>

<p>
The tiled YOLO output was saved separately to compare its behavior with normal YOLO detection.
</p>

<h3><strong>Crowd Density Heatmap</strong></h3>

<p>
A crowd density heatmap was also added as an extra visualization feature. It uses the center point of detected human bounding boxes and creates a heatmap to show crowded areas.
</p>

<pre><code>Detected human boxes
→ Extract human center points
→ Apply heatmap generation
→ Overlay heatmap on original image</code></pre>

<p>
This feature is useful for drone-based crowd monitoring, public safety analysis, rescue operations, and surveillance. It does not replace object detection, but it gives an additional visual understanding of human concentration in the scene.
</p>

<h3><strong>Object Tracking Note</strong></h3>

<p>
Full object tracking using ByteTrack, DeepSORT, BoT-SORT, or OC-SORT was not included in the final submitted test pipeline. Instead, the project focuses on detection, human counting, tiled small-object detection, class-aware hybrid detection, and crowd density visualization.
</p>
</p>
<h3><strong>Optional ByteTrack Tracking</strong></h3>

<p>
As an additional bonus feature, ByteTrack tracking was added at the end of the pipeline. Since object tracking works better on video or continuous frames, a short video was created from sequential test images. The trained YOLO model was then used with ByteTrack to assign a unique track ID to the same human or car across frames. This helps identify whether an object appearing in multiple frames is the same object, which can reduce repeated counting in video-based monitoring.
</p>

<pre><code>Sequential test images
→ Create short video
→ Run YOLO + ByteTrack
→ Assign track IDs
→ Save tracking output</code></pre>

<p>
The tracking result is saved in:
</p>

<pre><code>test_output_5_bytetrack_tracking</code></pre>


