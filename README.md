<!DOCTYPE html>
<html>
<head>
  <title>Pose Control</title>

  <!-- MediaPipe -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose/pose.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js"></script>
</head>

<body style="margin:0;">

<video id="video" autoplay playsinline style="position:absolute;"></video>
<canvas id="canvas"></canvas>

<script>
const video = document.getElementById("video");
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

// 카메라 켜기
navigator.mediaDevices.getUserMedia({ video: true })
.then(stream => {
  video.srcObject = stream;
});

// MediaPipe Pose 설정
const pose = new Pose({
  locateFile: (file) => {
    return `https://cdn.jsdelivr.net/npm/@mediapipe/pose/${file}`;
  }
});

pose.setOptions({
  modelComplexity: 0,
  smoothLandmarks: true,
  enableSegmentation: false
});

// 결과 처리
pose.onResults((results) => {
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;

  ctx.drawImage(results.image, 0, 0, canvas.width, canvas.height);

  if (results.poseLandmarks) {
    const nose = results.poseLandmarks[0];
    const leftHand = results.poseLandmarks[15];

    // 손 들면 배경 변경
    if (leftHand.y < nose.y) {
      document.body.style.background = "red";
    } else {
      document.body.style.background = "white";
    }
  }
});

// 카메라 연결
const camera = new Camera(video, {
  onFrame: async () => {
    await pose.send({ image: video });
  }
});
camera.start();
</script>

</body>
</html>
