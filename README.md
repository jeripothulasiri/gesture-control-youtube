Here’s the humanized and enriched version with more images and a conversational tone.

---

# Controlling YouTube Player with Hand Gestures

## Index
1. [Introduction](#introduction)
2. [How It Works](#how-it-works)  
    * [Hand Gestures](#hand-gestures)  
    * [Sleepiness Detection](#sleepiness-detection)    
    * [Absence Detection](#absence-detection)  
3. [Project Structure](#project-structure)
4. [How to Use It](#how-to-use-it)
    * [Installing Libraries](#installing-libraries)
    * [Saving Gesture Data](#saving-gesture-data)
    * [Training the Model](#training-the-model)
    * [Running the Web App](#running-the-web-app)
5. [Limitations](#limitations)
6. [References](#references)

---

## Introduction

Imagine controlling your YouTube player without touching the keyboard or mouse—just using your hands! Gesture-based interfaces are becoming increasingly popular because they allow for natural and intuitive interactions. They’re being used in a variety of fields, from home automation to healthcare and even virtual reality.

The goal of this project is to recognize a set of hand gestures using an Artificial Neural Network and use those gestures to control a YouTube player. Not only that, but the system can also detect if you’ve fallen asleep or left the room, pausing the video automatically. How cool is that? Check out this [demo](https://www.youtube.com/watch?v=gHVrGI3632s)!

![Gesture Control Demo](https://user-images.githubusercontent.com/100664869/194749265-5bd27c59-a248-440e-8702-5a442b83472b.gif)

Why YouTube? Well, it's widely used, free (ads included), and you can find almost any kind of visual content. Plus, this technique can be adapted to control any other media player that supports keyboard shortcuts or has an API.

---

## How It Works

### Hand Gestures

For detecting hand gestures, the approach I used was inspired by [Nikita Kiselov’s project](https://github.com/kinivi/tello-gesture-control). One major advantage of this method is that it doesn't require tons of images for training. Instead, it uses landmarks as input for the model.

Here’s how it works:

1. **Extracting Landmarks:** I used [MediaPipe's hand detector](https://google.github.io/mediapipe/solutions/hands.html) to extract 2D coordinates of 21 hand landmarks. The image below shows these keypoints. For this project, I focused on wrist and fingertip coordinates.

    ![Hand Keypoints](https://user-images.githubusercontent.com/100664869/194749666-20208ade-89d6-4062-b177-f36e514c0b1e.png)

2. **Preprocessing:** The wrist coordinates were subtracted from the rest of the points, flattened, and normalized by the maximum absolute value. I also calculated distances between specific points and normalized those too.

3. **Training the Model:** With the preprocessed data (13 features), I trained a simple artificial neural network to recognize 13 classes of gestures.

    ![Gesture Detection Workflow](https://user-images.githubusercontent.com/100664869/194754857-cb9520dd-9ea5-4c6c-bc81-aed97e0e3195.png)

### Sleepiness Detection

Inspired by [Adrian Rosebrock's blog](https://pyimagesearch.com/2017/05/08/drowsiness-detection-opencv/), this feature detects if you’re about to fall asleep while watching a video.

Here’s the process:

1. **Face Detection:** I used Dlib's frontal face detector. For installation instructions, see [Ubuntu/macOS](https://pyimagesearch.com/2017/03/27/how-to-install-dlib/) or [Windows 10](https://www.geeksforgeeks.org/how-to-install-dlib-library-for-python-in-windows-10/).

2. **Eye Aspect Ratio (EAR):** By analyzing eye landmarks, the system calculates the EAR to determine if you’re dozing off. If the EAR drops below a certain threshold and stays there, it pauses the video.

    ![Sleep Detection Workflow](https://user-images.githubusercontent.com/100664869/194755008-74282aa3-01a1-4ea3-8563-a1777278e752.png)

### Absence Detection

Absence detection is pretty straightforward:

1. **Face Detection:** Using [MediaPipe's face detector](https://google.github.io/mediapipe/solutions/face_detection.html), it checks if your face is visible.

2. **Absence Detection:** If no face is detected for a set number of frames, the system assumes you’ve left and pauses the video.

---

## Project Structure

```bash
 ┣━ 📂data
 ┃ ┣━ 📜check_data.ipynb
 ┃ ┣━ 📜gestures.csv
 ┃ ┣━ 📜label.csv
 ┃ ┗━ 📜player_state.json
 ┣━ 📂flask_app
 ┃ ┣━ 📂static
 ┃ ┃ ┣━ 📂css
 ┃ ┃ ┃ ┗━ 📜styles.css
 ┃ ┃ ┗━ 📂icons
 ┃ ┃ ┃ ┗━ 📜favicon-32x32.png
 ┃ ┣━ 📂templates
 ┃ ┃ ┗━ 📜demo.html
 ┃ ┣━ 📜app.py
 ┃ ┗━ 📜video_feed.py
 ┣━ 📂models
 ┃ ┣━ 📜model.pth
 ┃ ┣━ 📜model_architecture.py
 ┃ ┗━ 📜shape_predictor_68_face_landmarks.dat
 ┣━ 📜main.py
 ┣━ 📜requirements.txt
 ┣━ 📜train.ipynb
 ┗━ 📜utils.py
```

- **main.py:** Saves data and checks model output.
- **utils.py:** Contains helper functions used in `main.py`.
- **train.ipynb:** Used for training and validating the neural network.
- **data/:** Contains saved data (`gestures.csv`), and a notebook to check data (`check_data.ipynb`). `player_state.json` stores the player's state (playing or paused).
- **models/:** Contains the trained neural network (`model.pth`) and its architecture (`model_architecture.py`). It also includes the facial landmarks predictor.
- **flask_app/:** Contains files for running the web app.

---

## How to Use It

### Installing Libraries

First, create a virtual environment and install the required libraries:

```bash
cd project_folder_name
python -m venv your_virtual_env_name
your_virtual_env_name\Scripts\activate.bat
pip install -r requirements.txt  
```

### Saving Gesture Data

Run `main.py`. Once the webcam video loads, press 'r' to start logging gestures. Press keys '0' to '9' (or 'A' to 'Z' for more classes) to save gestures in `gestures.csv`. Each row in the CSV represents a gesture, with the first column being the label and the others being the normalized keypoints and distances.

![Data Logging Example](https://user-images.githubusercontent.com/100664869/194744094-7ee8244c-a750-4339-bdd5-1f57f8226564.png)

### Training the Model

Open and run `train.ipynb` to train your model. Adjust the model architecture or data if needed. The model architecture looks like this:

![Model Architecture](https://user-images.githubusercontent.com/100664869/194754632-14b9589f-7689-41b5-897f-0ed5339bbbf5.png)

### Running the Web App

To run the web app, follow these steps:

```bash
cd flask_app
python app.py
```

You'll get a link, like http://127.0.0.1:5000, where the app is running.

Paste a YouTube video link into the input field and click "Start." The video will load along with your webcam feed. Hand gestures work when your hand is within the red box in the webcam feed. Initially, left-click on the player to focus, then control the video using hand gestures.

![Web App](https://user-images.githubusercontent.com/100664869/194744362-67e00d66-0f01-49b2-b253-e4e3bd055003.png)

Here’s a quick reference for gestures:

| Gesture | Function |
|---|---|
| Move mouse | Move mouse cursor |
| Left click | Left-click mouse |
| Right click | Right-click mouse |
| Play/Pause | Toggle play/pause |
| Backward | Seek backward 5 seconds |
| Forward | Seek forward 5 seconds |
| Vol.down.gen | Decrease computer's volume |
| Vol.up.gen | Increase computer's volume |
| Vol.down.ytb | Decrease YouTube volume |
| Vol.up.ytb | Increase YouTube volume |
| Full screen | Toggle full-screen mode |
| Subtitle | Toggle subtitles/closed captions |
| Neutral | No action |
| Sleepiness | Pause video if sleeping |
| Absence | Pause video if absent |

---

## Limitations

- **Low Light:** Hand landmarks might not be stable in low light, affecting gesture detection quality.
- **Frontal Face Required:** Sleepiness detection works best when your face is directly facing the camera.
- **Distance:** The system