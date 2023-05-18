# 현재 코드

MainActivity.kt에서 `private fun measureCprScore(person: Person)` 코드만 확인하면 됩니다

위치 : tensorflow > examples > poseestimatin > MainActivity.kt

```kotlin
// person이 갖고 있는 관절 데이터들에서 어깨, 팔꿈치, 손목 데이터 추출 (현재 임시로 왼쪽 관절만 추출한 상태)
        person.keyPoints.forEach { point ->
            when (point.bodyPart) {
                BodyPart.LEFT_SHOULDER -> {
                    xShoulder = point.coordinate.x
                    yShoulder = point.coordinate.y
                }
                BodyPart.LEFT_ELBOW -> {
                    xElbow = point.coordinate.x
                    yElbow = point.coordinate.y
                }
                BodyPart.LEFT_WRIST -> {
                    xWrist = point.coordinate.x
                    yWrist = point.coordinate.y
                }
                else -> {}
            }
        }
```

```kotlin
// 일직선 판별
        var isCorrect = xShoulder - xElbow < 20 && xElbow - xWrist < 20
        if (isCorrect) {
            Log.i(TAG, "올바른 자세에요!")
            // TODO : 맞은 횟수 세기
            correctAngle++
        } else {
            Log.i(TAG, "팔을 90도로 유지하세요!")
            // TODO : 틀린 횟수 세기
            incorrectAngle++
        }
```

```kotlin
// 손목의 높이가 상승 곡선에서 꼭짓점을 찍고 하강하는 경우
        if (increased && beforeWrist > yWrist + 1) {
            increased = false
            maxHeight = yWrist
        }
        // 손목의 높이가 하강 곡선에서 꼭짓점을 찍고 상승하는 경우
        else if (!increased && beforeWrist < yWrist - 1) {
            increased = true
            minHeight = yWrist

            val num = if (maxHeight > minHeight) maxHeight - minHeight else minHeight - maxHeight
            wristList.add(num)
            Log.e(TAG, "${wristList.last()}")
        }
```

필요한 신체부위를 찾을때는 `BodyPart`에서 찾으면 됩니다

위치 : tensorflow > examples > poseestimatin > data > BodyPart.kt
```kotlin
enum class BodyPart(val position: Int) {
    NOSE(0),
    LEFT_EYE(1),
    RIGHT_EYE(2),
    LEFT_EAR(3),
    RIGHT_EAR(4),
    LEFT_SHOULDER(5),
    RIGHT_SHOULDER(6),
    LEFT_ELBOW(7),
    RIGHT_ELBOW(8),
    LEFT_WRIST(9),
    RIGHT_WRIST(10),
    LEFT_HIP(11),
    RIGHT_HIP(12),
    LEFT_KNEE(13),
    RIGHT_KNEE(14),
    LEFT_ANKLE(15),
    RIGHT_ANKLE(16);
    companion object{
        private val map = values().associateBy(BodyPart::position)
        fun fromInt(position: Int): BodyPart = map.getValue(position)
    }
}
```

# TensorFlow Lite Pose Estimation Android Demo

### Overview
This is an app that continuously detects the body parts in the frames seen by
your device's camera. These instructions walk you through building and running
the demo on an Android device. Camera captures are discarded immediately after
use, nothing is stored or saved.

The app demonstrates how to use 4 models:

* Single pose models: The model can estimate the pose of only one person in the
input image. If the input image contains multiple persons, the detection result
can be largely incorrect.
   * PoseNet
   * MoveNet Lightning
   * MoveNet Thunder
* Multi pose models: The model can estimate pose of multiple persons in the
input image.
   * MoveNet MultiPose: Support up to 6 persons.

See this [blog post](https://blog.tensorflow.org/2021/05/next-generation-pose-detection-with-movenet-and-tensorflowjs.html)
for a comparison between these models.

![Demo Image](posenetimage.png)

## Build the demo using Android Studio

### Prerequisites

* If you don't have it already, install **[Android Studio](
 https://developer.android.com/studio/index.html)** 4.2 or
 above, following the instructions on the website.

* Android device and Android development environment with minimum API 21.

### Building
* Open Android Studio, and from the `Welcome` screen, select
`Open an existing Android Studio project`.

* From the `Open File or Project` window that appears, navigate to and select
 the `lite/examples/pose_estimation/android` directory from wherever you
 cloned the `tensorflow/examples` GitHub repo. Click `OK`.

* If it asks you to do a `Gradle Sync`, click `OK`.

* You may also need to install various platforms and tools, if you get errors
 like `Failed to find target with hash string 'android-21'` and similar. Click
 the `Run` button (the green arrow) or select `Run` > `Run 'android'` from the
 top menu. You may need to rebuild the project using `Build` > `Rebuild Project`.

* If it asks you to use `Instant Run`, click `Proceed Without Instant Run`.

* Also, you need to have an Android device plugged in with developer options
 enabled at this point. See **[here](
 https://developer.android.com/studio/run/device)** for more details
 on setting up developer devices.


### Model used
Downloading, extraction and placement in assets folder has been managed
 automatically by `download.gradle`.

If you explicitly want to download the model, you can download it from here:

* [Posenet](https://storage.googleapis.com/download.tensorflow.org/models/tflite/posenet_mobilenet_v1_100_257x257_multi_kpt_stripped.tflite)
* [Movenet Lightning](https://tfhub.dev/google/movenet/singlepose/lightning/)
* [Movenet Thunder](https://tfhub.dev/google/movenet/singlepose/thunder/)
* [Movenet MultiPose](https://tfhub.dev/google/movenet/multipose/lightning/)

### Additional Note
_Please do not delete the assets folder content_. If you explicitly deleted the
 files, then please choose `Build` > `Rebuild` from menu to re-download the
 deleted model files into assets folder.
