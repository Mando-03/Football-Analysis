
# Football Analysis

A comprehensive tool for processing and analyzing video footage, producing detailed insights into gameplay and player performance enhancing game understanding and performance evaluation.


## ⚽ Features
1. [Comprehensive Object Detection and Tracking](#comprehensive-object-detection-and-tracking)
2. [Field Keypoint Detection](#field-keypoint-detection)
3. [Player Club Assignment](#player-club-assignment)
4. [Real-World Position Mapping](#real-world-position-mapping)
5. [Dynamic Voronoi Diagram](#dynamic-voronoi-diagram)
6. [Ball Possession Calculation](#ball-possession-calculation)
7. [Speed Estimation](#speed-estimation)
8. [Live Video Preview](#live-video-preview)
9. [Tracking Data Storage](#tracking-data-storage)


## ❓ How to Run


### Clone the Repository:
Clone the project repository to your local machine:
```bash
git clone https://github.com/mradovic38/football_analysis.git
cd football_analysis
```

### Prerequisites
- Ensure you have **Python 3.6 or later** installed.
- Install the required packages using pip:
```bash
pip install -r requirements.txt
```

### Add Roboflow API Key:
1. Create a file in the root directory named: `config.py`.
2. Add the folowing in `config.py`: 
```python
ROBOFLOW_API_KEY = 'your_roboflow_api_key_here'
```
Replace the placeholder value with your actual Roboflow API key.

### Train the Models:
Before running the analysis, you need to train the models. The training notebooks are located in the [models/train](models/train) subfolder. You will find two notebooks: [`object_detection_train.ipynb`](models/train/object_detection/object_detection_train.ipynb) and [`keypoints_detection_train.ipynb`](models/train/keypoints_detection/keypoints_detection_train.ipynb).

Open each notebook and modify the `RESULTS_DIR` parameter to ensure it points to the latest training run (e.g., `trainx`, where x is the index of the latest run).

#### Configure Input and Output Video Paths:
Edit the [`main.py`](main.py) file to configure the input and output video paths according to your requirements. Make sure the paths are set correctly to ensure the analysis can access the videos.

#### Install Required Dependencies:
Ensure you have all the necessary dependencies installed. You can use a package manager like pip:
```bash
pip install -r requirements.txt
```

#### Configure Club Colors:
Edit the [`main.py`](main.py) file to define the jersey colors of the clubs you want to analyze. Modify the RGB values in the Club object creation section.

#### Run the Script:
Open a terminal and navigate to the project directory. Execute the [`main.py`](main.py) script to start the analysis:
```bash
python main.py
```
## 📜 Feature Explanations

### [Comprehensive Object Detection and Tracking](tracking/object_tracker.py)
Detect and track players, goalkeepers, referees, and footballs using advanced computer vision techniques.


#### Detection: 
The system processes multiple frames from a video feed, resizing them to 1280x1280 pixels for optimal performance. The **YOLO11s detection** model performs batch inference, identifying objects such as players, goalkeepers, referees, and the ball, returning bounding boxes and associated confidence scores.
#### Tracking:
Using the **Byte Tracker**, detected objects are tracked across frames. The tracking maintains the identities of objects as they move through the scene.


### [Field Keypoint Detection](tracking/keypoints_tracker.py)
Identify keypoints on the football field to facilitate accurate spatial analysis and improve the quality of data collected.


The KeypointsTracker utilizes the **YOLO11n pose** model to detect keypoints in video frames. Each frame undergoes contrast adjustment using histogram equalization to improve the visibility of keypoints, which enhances the model's detection capabilities. After preprocessing, frames are resized to 1280x1280 pixels to ensure compatibility with the model.

### [Player Club Assignment](club_assignment/club_assigner.py)
Automatically assign clubs to players based on jersey colors, streamlining the analysis process.


#### Color Masking: 
Applies a mask to images based on green color detection in the HSV color space. This masking helps to isolate players from the background by filtering out green areas (such as the field) and focusing on player jerseys. If the green coverage exceeds a specified threshold, the image is masked accordingly to improve jersey color detection.

#### Color Clustering: 
The dominant jersey color is identified through **K-Means clustering**. The jersey color is then extracted based on cluster analysis, distinguishing between player and background colors.

#### Club Prediction: 
Predicting which club a player belongs to based on the extracted jersey color. Calculating the closest color centroid associated with each club and assigning the player to the club with the nearest match. This functionality extends to both players and goalkeepers, differentiating between their respective color schemes.

### [Real-World Position Mapping](position_mappers)
Map real-world object positions to a 2D surface to enhance the understanding of gameplay dynamics.\
Real-world position mapping involves transforming detected object positions from a perspective view into a top-down representation. This process begins by identifying keypoints within the original perspective, which serve as reference points for the transformation. By computing a **homography matrix** based on these keypoints, the system establishes a relationship between the original perspective and the desired top-down view.
Once the homography matrix is obtained, the positions of detected objects, such as players or other entities, are projected into this new perspective. The result is a set of coordinates that accurately reflect the positions of these objects in the top-down view, facilitating analysis and visualization of their movements on the field. This mapping process is crucial for applications in sports analysis, enabling a clearer understanding of player positioning and strategies during a game.

### [Dynamic Voronoi Diagram](annotation/projection_annotator.py)
The Voronoi diagram represents regions of influence around a set of points, with each region containing all locations closest to one particular point compared to any other. In this context, each point represents a player's position on the field, and the resulting Voronoi regions give a visual indication of each player's area of control.


### [Ball Possession Calculation](ball_to_player_asignment)
Calculate ball possession effectively and assign possession to players, providing valuable insights into gameplay strategies.


#### Nearest Player:
Ball possession is tracked by determining which player or team controls the ball at any given time. This is done by calculating the distance between the ball and players, and assigning possession to the nearest player if they are within a reasonable range. Once a player is identified as possessing the ball, their team is given credit for possession.

#### Determining ball validity:
To ensure the ball is valid, its movement is checked for consistency with the previous frames. If the ball moves too quickly or erratically (exceeding a defined speed threshold), it is considered invalid, and its position is disregarded. Since the penalty dot gets mistaken for a ball often, these positions are being invalidated.

#### Grace Periods:
The tracking system also includes a grace period, allowing a player or team to retain possession for a short time even if the ball briefly leaves their control. If no valid possession is detected for a while, possession is lost, and the system resets. There is also a ball grace period, which allows the system to briefly retain the last known possession if the ball temporarily disappears or its position becomes uncertain. This ensures possession isn't lost immediately when the ball goes out of view or experiences brief tracking issues.

#### Calculating Possession
Over time, these possession instances are accumulated to calculate the overall possession percentages for each team. These percentages give an indication of how much time each team has controlled the ball during the game.

### [Speed Estimation](speed_estimation/speed_estimator.py)
The speed estimation process in this context aims to calculate how fast players are moving on the football field using video footage.

#### Real-World Scaling:
The football field in the video is measured in pixels, but real-world distances are needed to estimate speed. By knowing the actual dimensions of the field (in meters) and the dimensions in pixels, a scaling factor is created for both the x-axis and y-axis. This allows conversion from pixel distances to real-world meters.

#### Speed Calculation:
The Euclidean distance between the player’s current position and their previous position is calculated. This gives the distance the player has moved between frames, in meters.

The time difference between the two frames is computed based on the frame rate (frames per second, or FPS) of the video. This provides the duration over which the player moved the calculated distance.\
Speed is computed by dividing the distance traveled by the time difference. The result is in meters per second, which is then converted to kilometers per hour (km/h) by multiplying by 3.6.
A maximum speed limit is applied (e.g. 40 km/h) to ensure that the calculated speeds remain realistic.

#### Smoothing:
To reduce fluctuations in speed due to sudden changes or noise in the tracking data, a smoothing mechanism is applied. This uses a moving average over a defined number of frames (smoothing window). The speeds from previous frames are averaged with the current speed, giving a more stable speed estimate.

### Live Video Preview
Monitor the video processing in real-time with a live video preview, enhancing user interaction and analysis.

### [Tracking Data Storage](file_writing/tracks_json_writer.py)
Save tracking information to JSON files for further analysis and record-keeping, ensuring a comprehensive review of gameplay data.
## License

This project is licensed under the [MIT License](LICENSE). However, it uses the YOLO11 models, which are licensed under the [AGPL-3.0 and Enterprise Licenses](https://www.ultralytics.com/license).


## 📖 Resources

 - [Code In a Jiffy - Build an AI/ML Football Analysis system with YOLO, OpenCV, and Python](https://youtu.be/neBZ6huolkg?si=ZEgoreaHGE6YniXN)
 - [Roboflow - Football AI Tutorial: From Basics to Advanced Stats with Python](https://youtu.be/aBVGKoNZQUw?si=DEb1BbARQBrd87Ez)

