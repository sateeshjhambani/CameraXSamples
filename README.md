# CameraXSamples

A sample app that outlines the capture image and capture video use cases of the Jetpack's CameraX API. The app presents a simplified UI with the CameraX preview, a couple of icons to take photos and videos and a bottom sheet to present the captured media content.

We use the CameraX's LifecycleCameraController class (which provides most of the API's functionality) to capture photos and videos.

## Usage

The method responsible for capturing a still image.

```kotlin
private fun takePhoto(
        controller: LifecycleCameraController,
        onPhotoTaken: (Bitmap) -> Unit
    ) {
        if (!hasRequiredPermissions()) return

        controller.takePicture(
            ContextCompat.getMainExecutor(applicationContext),
            object: OnImageCapturedCallback() {
                override fun onCaptureSuccess(image: ImageProxy) {
                    super.onCaptureSuccess(image)

                    // image transformation
                    val matrix = Matrix().apply {
                        postRotate(image.imageInfo.rotationDegrees.toFloat())
                    }
                    val rotatedBitmap = Bitmap.createBitmap(
                        image.toBitmap(),
                        0,
                        0,
                        image.width,
                        image.height,
                        matrix,
                        true
                    )

                    onPhotoTaken(rotatedBitmap)
                }

                override fun onError(exception: ImageCaptureException) {
                    super.onError(exception)

                    Log.e("Camera", "Unable to take a photo: ", exception)
                }
            }
        )
    }
```

The method responsible for recording a video. 

```kotlin
private fun recordVideo(
        controller: LifecycleCameraController,
        onStartRecording: () -> Unit,
        onStopRecording: () -> Unit,
    ) {
        if (recording != null) {
            recording?.stop()
            recording = null

            onStopRecording()

            return
        }

        if (!hasRequiredPermissions()) return

        onStartRecording()

        val outputFile = File(filesDir, "my-recording.mp4")
        recording = controller.startRecording(
            FileOutputOptions.Builder(outputFile).build(),
            AudioConfig.create(true),
            ContextCompat.getMainExecutor(applicationContext),
        ) { event ->
            when (event) {
                is VideoRecordEvent.Finalize -> {
                    if (event.hasError()) {
                        recording?.close()
                        recording = null

                        onStopRecording()

                        Toast.makeText(applicationContext, "Video capture failed", Toast.LENGTH_LONG).show()
                    } else {
                        Toast.makeText(applicationContext, "Video capture succeeded", Toast.LENGTH_LONG).show()
                    }
                }
            }
        }
    }
```

### References

[CameraX Documentation] (https://developer.android.com/media/camera/camerax)
