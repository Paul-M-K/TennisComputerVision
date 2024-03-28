# Tennis Match Analysis Project

## Overview

This project aims to bring computer vision technologies into the realm of tennis sports analytics. By utilizing advanced machine learning models, the system is designed to detect and track players and tennis balls throughout video footage of matches. This projects solution extends beyond simple tracking; it comprehensively understands the spatial dynamics of the game by detecting key points of the tennis court. This allows for an in-depth analysis of player positioning, ball trajectories, distances covered, and the precision of shots. 

## Features

- **Player and Ball Detection**: Utilizes YOLO, an object detection model, to accurately detect and track fast-moving tennis balls and players in real-time.
- **Court Key Points Detection**: Employs a Convolutional Neural Network (CNN) trained to identify the court's key points, assisting in strategic analysis.
- **Gameplay Analysis**:
  - Determines whether shots are in or out.
  - Counts the number of shots hit by each player.
  - Measures the velocity of each shot.
- **Player Movement Analysis**:
  - Calculates the speed of the players.
  - Evaluates the total distance covered by each player during a match.

## Technology Stack

- **PyTorch**: The project uses PyTorch to train both the YOLO and CNN models, leveraging its dynamic computational graph and efficient memory usage for high-speed analysis.
- **Object Detection Model (YOLO)**: Fine-tuned to enhance the detection accuracy of the default out-of-the-box models for the high-speed context of tennis.
- **Convolutional Neural Network**: Custom-built and trained to estimate the tennis court's key points, enabling a nuanced understanding of player positions and game dynamics.

## Getting Started

### Analyzing a Single Frame

To begin our analysis, we start with a static image of a tennis match. This initial step helps us understand the basics of image detection using the YOLO model.

Original Image:
![Original Tennis Match Image](input_videos/image.png)

After running this image through the YOLO model, we observe the following results:

Predicted Image:
![YOLO Predicted Image](runs/detect/predict/image.png)

In the predicted image, we can see that YOLO successfully detects various objects, including people, tennis rackets, and a clock. Each detected object is surrounded by a bounding box, accompanied by a class name and a confidence score, indicating the model's certainty about the object's identity.

### Analyzing a Video Sequence

Next, we apply YOLO to a video of a tennis match. In this scenario, YOLO analyzes each frame of the video and assigns bounding boxes, class names, and confidence scores to all detected objects.

Video Analysis:
![YOLO Video Analysis](runs/detect/predict3/input_video.gif)

However, we notice that the detection of the tennis ball (classified as "sports ball") is inconsistent across frames. To improve our analysis and gather more detailed information, we will need to fine-tune a detector model specifically for the tennis ball.

## Refining the YOLO Model for Tennis Ball Detection

To enhance the accuracy of our YOLO model in detecting tennis balls, we will utilize a specialized training dataset tailored for this purpose. This dataset is available on Roboflow and can be accessed via the following link:

[Tennis Ball Detection Dataset on Roboflow](https://universe.roboflow.com/viren-dhanwani/tennis-ball-detection)
![Sample Image from Dataset](readme_folder/fed3_jpg.rf.09d2a51b2725b6734cf7c512f9eab272.jpg)


The dataset utilized for refining the YOLO model is a collection of images capturing tennis court scenes, closely resembling the initial image used in our analysis. It includes aerial photographs of tennis courts with the tennis ball in motion, often appearing blurry due to its high speed. Each image in the dataset is annotated with bounding boxes that precisely outline the tennis ball, providing essential training data for improving the model's detection accuracy (Bounding Box [0 0.58359375 0.4861111111111111 0.0125 0.025] the first number is the class the last for are the bounding box).

### Dataset Composition:

- **Total Images**: 578
- **Training Set**: 428 images
- **Validation Set**: 100 images
- **Test Set**: 50 images

This dataset is instrumental in training our YOLO model to better detect and track tennis balls, particularly in challenging conditions where the ball is moving rapidly. By fine-tuning our model with this dataset, we aim to enhance its performance in real-time tennis match analysis.

### Implementing Dual-Pass Detection for Comprehensive Analysis

In our refined YOLOv5 model, we have significantly improved the detection of tennis balls, achieving more consistent results than before. However, this specialization has led to a limitation: the model now primarily detects tennis balls and overlooks other crucial elements in the scene, such as players and objects. This is a result of training the model exclusively on a dataset comprising tennis balls.

![Improved Ball Detection](runs/detect/predict4/input_video.gif)

To overcome this challenge and ensure a comprehensive analysis of tennis matches, we will adopt a dual-pass detection approach:

1. **First Pass - Player and Object Detection**: Utilize a general object detection model, such as YOLOv8, to detect players, chairs, and other objects in the frame. This pass will provide us with bounding boxes around these objects, capturing their presence and locations.

2. **Second Pass - Tennis Ball Detection**: Apply our refined YOLOv5 model specifically trained for tennis ball detection. This pass will focus on accurately tracking the tennis ball throughout the match.

By combining the results of these two passes, we can achieve a holistic view of the match, tracking both the players and the tennis ball simultaneously.

### Object Tracking for Consistent Analysis

A key aspect of our analysis is object tracking, which involves maintaining the identity of objects across frames. While our current approach provides bounding boxes around players, these boxes are not consistent between frames due to the movement of objects. To address this, we will implement object tracking techniques that match bounding boxes across frames, ensuring that we can track the same player throughout the video. This matching process, known as object tracking, will allow us to analyze the movements and positions of players more accurately.

By integrating these enhancements into our system, we aim to provide a more detailed and consistent analysis of tennis matches, capturing both the dynamics of the ball and the players' movements.


## Tracking Players with YOLOv8

After successfully detecting the tennis ball with our refined YOLOv5 model, we now focus on tracking the players in the video using YOLOv8. YOLOv8 is known for its high accuracy and speed in object detection, making it an ideal choice for tracking fast-moving objects like players in a tennis match.

![Object trcking](runs/detect/track2/input_video.gif)

### Implementation

To track players, we utilize the tracking feature of YOLOv8, which provides us with bounding boxes around detected players across frames. This allows us to monitor their movements and positions throughout the match. Here's how we implement it:

```python
from ultralytics import YOLO

# Initialize the YOLOv8 model
model = YOLO('yolov8x')

# Track players in the video
result = model.track('input_videos/input_video.mp4', conf=0.2, save=True)

# Print results and bounding boxes
print(result)
print("boxes:")
for box in result[0].boxes:
    print(box)
```

In this implementation, we set a confidence threshold of 0.2 to ensure that the model only tracks objects it is reasonably sure about. The `track` method returns results containing the bounding boxes for detected players in each frame.

### Analysis

With the bounding boxes obtained from YOLOv8, we can analyze various aspects of player movement, such as:

- **Speed**: By calculating the distance covered by a player between frames, we can estimate their speed.
- **Positioning**: Analyzing the positions of players at different points in the match can provide insights into their strategies and court coverage.
- **Engagement**: By tracking how often and how quickly players move, we can gauge their level of engagement and fatigue.

By combining the tracking information from YOLOv8 with the ball detection from YOLOv5, we can create a comprehensive analysis of the tennis match, covering both the players' movements and the trajectory of the ball.


## Video Analysis Output

In the video output generated by our tracking implementation, we can observe the following:

- **Player Tracking**: The players are being tracked consistently across frames, as indicated by the unique IDs assigned to each player (e.g., id1 and id2). These IDs remain constant throughout the video, demonstrating the model's accuracy in tracking the players' movements.

- **Ignoring Irrelevant Objects**: Besides the players, there may be other individuals or objects in the scene that are also assigned IDs. However, for the purpose of our analysis, we will focus solely on the IDs corresponding to the two main players. This allows us to isolate and track their movements without being distracted by other elements in the video.

- **Ease of Isolation**: With the unique IDs assigned to each player, we can easily isolate their movements for further analysis. This enables us to study their positioning, speed, and overall engagement in the match without interference from other tracked objects.

The video output provides a clear visual representation of how well our model is able to track the players, making it a valuable tool for analyzing their performance and strategies during the match.







## Contributing

(Include guidelines on how others can contribute to the project.)

## License

(Include information about the project's license here, if applicable.)

## Contact

(Your contact information or that of the project maintainer.)


# Tennis Match Analyzer with YOLOv8x

This project utilizes the YOLOv8x (You Only Look Once, version 8x) object detection model to analyze and identify various elements in a live tennis match. The image demonstrates the model's ability to detect and classify objects with bounding boxes and confidence scores in real-time.
![Initial Image of tennis match](/input_videos/image.png)


## Features

- **Real-Time Object Detection**: Using the YOLOv8x model, the application identifies and labels different objects in a tennis match scene.
- **Confidence Scoring**: Each object is recognized with a confidence score, reflecting the model's certainty about the object's identification.
- **Multi-Object Classification**: The model simultaneously classifies multiple objects, including tennis players, audience members, chairs, and sports equipment.
- **Sports Analytics**: This application can be utilized for sports analytics, providing insights into player positions, game statistics, and audience engagement.

## Application in Action

The provided image from a tennis match shows the YOLOv8x model's output. The model highlights:

- Bounding box.
- class name.
- Confidence score.
- Players on the court.
- Spectators around the court.
- The chair umpire.
- Tennis equipment, including the ball and racket.
![inital prediction](/runs/detect/predict/image.png)
Each object is enclosed in a bounding box with a label and a confidence score (e.g., 'person 0.83', 'clock 0.36', 'sports ball 0.50').

## Potential Use Cases

- **Player Performance Tracking**: Analyzing player movements and positions to aid in performance improvement strategies.
- **Audience Analysis**: Monitoring audience size and engagement levels.
- **Equipment Detection**: Keeping track of the ball and equipment for automated scoring systems.

## Future Enhancements

- Enhancing the model for even higher accuracy and speed.
- Expanding the number of recognizable objects specific to various sports.
- Integrating with video feeds for live match analysis.
