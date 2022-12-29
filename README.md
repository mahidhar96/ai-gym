# AI GYM
This project is a prototype for making sure that a person using is has the correct posture while performing an exercise, while intelligently tracking the number of reps.
The project contains 2 parts:
- An extensible framework for configuring exercises, for any number of cameras.
- A rep_counter for for running these cofigurations.

## Rep Counter
Rep counter currently leverages [mediapipe](https://google.github.io/mediapipe/) and expects mediapipe landmarks in the exercise configurations, but it is designed in such a way that this can be swapped with other pose estimation libraries such as [openpose](https://github.com/CMU-Perceptual-Computing-Lab/openpose), [deepstream](https://developer.nvidia.com/deepstream-sdk), [fastpose](https://drnoodle.github.io/fastpose_html/), etc.

## Exercises
This folder contains a bunch of predefined exercises in the format exercise_cameras. as of now most exercises are configures for 1 or 2 cameras, but this can be expanded.

## Example Run
![Example Run](readme.gif)

Full Demo Here: [Demo](https://www.youtube.com/watch?v=e9oyCnkOCok)

### Example exercise definition:

```
{
    "name": "LegPress", // Name of the exercise
    "version": "1.0", // Version for book keeping
    "measurements": [ // A list of measurements, the rep counter needs to calculate, these mesaurements will be used for counting reps in sequence and guiding the user in indicators. and plotting.
        {
            "name": "foot_length", // name of the measurement, this name can be used for referencing in indicators and sequnces for rep counting.
            "type": "euclidean", // type of the measurement
            "camera": 0, // the camera from which this should be calculated, by default it is zero
            "points": [
                "right_foot_index", // the points which are included for the type of calculation. 
                "left_foot_index"
            ]
        },
        {
            "name": "foot_length_1.1",
            "initial": "foot_length",
            "type": "multiply",
            "value": 1.1
        },
        {
            "name": "foot_length_1.3",
            "initial": "foot_length",
            "type": "multiply",
            "value": 1.3
        },
        {
            "name": "foot_length_0.8",
            "initial": "foot_length",
            "type": "multiply",
            "value": 0.8
        },
        {
            "name": "knee_length",
            "type": "euclidean",
            "camera": 0,
            "points": [
                "left_knee",
                "right_knee"
            ]
        },
        {
            "name": "shoulder_length",
            "type": "euclidean",
            "camera": 0,
            "points": [
                "left_shoulder",
                "right_shoulder"
            ]
        },
        {
            "name": "shoulder_length_1.3",
            "initial": "shoulder_length",
            "type": "multiply",
            "value": 1.3
        }
    ],
    "indicators": [
        {
            "name": "angle_back", // name of the indicators
            "type": "angle", // type of the indicators
            "camera": 1,
            "points": [
                "left_shoulder", //along with the predefined points fromo mediapipe, measurements can also be used here.
                "left_hip",
                "left_knee"
            ],
            "plot": true, // if set to true, this will be plotted.
            "good": { // what is the a good range for this indicator
                "min": 40,
                "max": 90,
                "warning": "Great !, good back posture" // description on the plot
            },
            "intermediate": {
                "min": 30,
                "max": 120,
                "warning": "Acceptable , Try adjusting your back alittle."
            },
            "bad": {
                "min": 10,
                "max": 180,
                "warning": "Bad back posture, try adjusting your back and Legs"
            }
        },
        {
            "name": "knee_length",
            "type": "relative",
            "camera": 0,
            "good": {
                "min": "foot_length_0.8",
                "warning": "Good Job !, maintiane this"
            },
            "intermediate": {
                "max": "foot_length_1.3",
                "warning": "Acceptable , Try adjusting your Knees alittle."
            },
            "bad": {
                "warning": "Knees are inward, try keeping them staright"
            }
        },
        {
            "name": "foot_length",
            "type": "relative",
            "camera": 0,
            "good": {
                "min": "shoulder_length",
                "warning": "Awesome !, good posture"
            },
            "intermediate": {
                "max": "shoulder_length_1.3",
                "warning": "Acceptable , Try adjusting your Feet to shouder length."
            },
            "bad": {
                "warning": "Feet and Shoulders should be parallel, try adjusting you feet"
            }
        }
    ],
    "sequence": [ //steps in sequence, when the user finishes these, a rep will be counted.
        {
            "name": "knee_angle",
            "type": "angle",
            "camera": 1,
            "points": [
                "left_hip",
                "left_knee",
                "left_ankle"
            ],
            "max": 210 // if this angle goes below 210, count step 1
        },
        {
            "name": "knee_angle",
            "type": "angle",
            "camera": 1,
            "points": [
                "left_hip",
                "left_knee",
                "left_ankle"
            ],
            "min": 270 // if this angle goes above 270 count step 2 and add 1 to reps
        }
    ],
    "plot": [ //additional plotting for debugging and tracking.
        {
            "name": "knee_angle",
            "type": "angle",
            "camera": 1,
            "points": [
                "left_hip",
                "left_knee",
                "left_ankle"
            ],
            "indicator": -1
        },
        {
            "name": "foot_length",
            "type": "line",
            "camera": 0,
            "points": [
                "right_foot_index",
                "left_foot_index"
            ],
            "indicator": 2
        },
        {
            "name": "knee_length",
            "type": "line",
            "camera": 0,
            "points": [
                "left_knee",
                "right_knee"
            ],
            "indicator": 1
        },
        {
            "name": "shoulder_length",
            "type": "line",
            "camera": 0,
            "points": [
                "left_shoulder",
                "right_shoulder"
            ],
            "indicator": 2
        }
    ]
}
```

## Future work
Track user experience using facial features usinf [dlib](http://dlib.net/) and [mediapipe face mesh](https://google.github.io/mediapipe/solutions/face_mesh.html)