# GestureX — Real-Time Hand Gesture Recognition

A real-time hand gesture recognition application built with **Python, OpenCV, MediaPipe, and scikit-learn**. It detects hand landmarks from a live camera feed, classifies five predefined gestures using a trained Random Forest model, and triggers a corresponding Windows desktop action.

## Features

- Real-time hand tracking using the MediaPipe Hand Landmarker task.
- Classification of five gestures using a Random Forest model.
- Landmark smoothing and confidence/hold-time checks to reduce accidental triggers.
- Desktop actions mapped to recognized gestures.
- Scripts for collecting landmark samples, preprocessing data, creating session-based train/test splits, and training classifiers.
- Evaluation results for both Random Forest and RBF SVM models.

## Gesture controls

| Gesture | Desktop action |
|---|---|
| Open palm | Open YouTube in the default browser |
| Fist | Minimize other visible windows (show desktop) |
| Thumb | Lock the computer |
| Index finger | Open Windows Settings |
| Peace sign | Open File Explorer |

> **Important:** This project is designed for **Windows**. Some actions use Windows-specific APIs and commands. The Thumb gesture locks the computer, so be aware of the action mapping before running the application.

## Tech stack

- Python
- OpenCV (`opencv-python`) for camera input and display
- MediaPipe Tasks for 21-point hand landmark detection
- NumPy and Pandas for data processing
- scikit-learn for Random Forest and RBF SVM classification
- Joblib for saving and loading trained models

## Project structure

```text
Project - Hand Gesture Detection/
├── dataset/
│   └── hand_gesture_master_dataset.csv
├── models/
│   ├── gesture_random_forest.joblib
│   └── hand_landmarker.task
├── results/
│   └── metrics.json
├── src/
│   ├── collect_landmarks.py
│   ├── preprocess_landmarks.py
│   ├── realtime_recognition.py
│   ├── train_classifier.py
│   └── train_test_split.py
├── requirements.txt
├── .gitignore
└── README.md
```

The `dataset/` file contains the collected landmark dataset. The trained Random Forest and MediaPipe task model in `models/` are used by the real-time application. Train/test split files and prediction exports are generated artifacts and are not required to be committed.

## Model evaluation

The supplied dataset contains **6,000 samples**. Evaluation used a session-level split so that samples from a capture session are kept together rather than being randomly distributed across training and testing rows.

| Model | Test accuracy |
|---|---:|
| Random Forest (deployed model) | **98.58%** |
| RBF SVM (comparison model) | **98.42%** |

The test set contains 1,200 samples; the training set contains 4,800 samples. These figures describe the recorded test split and are not a guarantee of accuracy in every camera, lighting, or user environment. Full evaluation details are in `results/metrics.json`.

## Requirements

- Windows 10 or Windows 11
- Python (the project was developed and tested in a Python 3.14.7 virtual environment)
- A working webcam or camera device
- The camera index configured in `src/realtime_recognition.py`

The default camera index in the scripts is `1`, originally configured for a phone camera connected through DroidCam. If you use a built-in webcam, you may need to change `CAMERA_INDEX` to `0` (or another index supported by your system) in `src/realtime_recognition.py`. The collection script has its own `CAMERA_INDEX` setting in `src/collect_landmarks.py`.

## Installation

Open PowerShell in the project root directory.

1. Create a virtual environment:

   ```powershell
   py -m venv .venv
   ```

2. Activate it:

   ```powershell
   .\.venv\Scripts\Activate.ps1
   ```

   If PowerShell blocks activation, you can run the commands below using `.venv\Scripts\python.exe` without activating the environment.

3. Upgrade pip and install dependencies:

   ```powershell
   python -m pip install --upgrade pip
   python -m pip install -r requirements.txt
   ```

## Run the application

From the project root, run:

```powershell
python src/realtime_recognition.py
```

The application opens a camera window and displays the detected gesture and its confidence/status. Hold a recognized gesture steadily until the on-screen countdown completes to trigger its mapped action. Press **Q** in the recognition window to quit.

## Training and data workflow

The following commands are for reproducing the data and model workflow. Run them from the project root.

1. **Collect additional landmarks** (optional; uses the camera and writes to `dataset/custom_landmarks/`):

   ```powershell
   python src/collect_landmarks.py
   ```

2. **Preprocess the master dataset** (optional; writes generated files under `features/`):

   ```powershell
   python src/preprocess_landmarks.py
   ```

3. **Create session-level train/test splits:**

   ```powershell
   python src/train_test_split.py
   ```

4. **Train and evaluate the classifiers:**

   ```powershell
   python src/train_classifier.py
   ```

Training writes model files under `models/` and evaluation outputs under `results/`. The real-time application loads `models/gesture_random_forest.joblib` and `models/hand_landmarker.task`.

## Configuration notes

- Change `CAMERA_INDEX` in the relevant source file if the intended camera is not detected.
- Gesture confidence and hold-time behavior can be adjusted near the settings at the top of `src/realtime_recognition.py` (`CONFIDENCE_THRESHOLD`, `GESTURE_HOLD_TIME`, and related settings).
- The desktop actions are implemented in `src/realtime_recognition.py` and are Windows-specific.

## Limitations

- Recognition performance can vary with camera quality, lighting, hand position, and gestures that differ from the training examples.
- The application currently supports the five gestures listed above.
- Desktop automation is Windows-specific and should not be assumed to work on macOS or Linux.
