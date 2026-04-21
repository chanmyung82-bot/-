<!DOCTYPE html>
<html>
<head>
  <title>Pose Test</title>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils"></script>
</head>

<body>
  <video id="video" autoplay playsinline style="display:none;"></video>
  <canvas id="canvas" width="640" height="480"></canvas>

<script>
const video = document.getElementById('video');
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

// 카메라 켜기
navigator.mediaDevices.getUserMedia({ video: true })
.then(stream => {
  video.srcObject = stream;
});

// 포즈 인식 설정
const pose = new Pose({
  locateFile: (file) => {
    return `https://cdn.jsdelivr.net/npm/@mediapipe/pose/${file}`;
  }
});

pose.setOptions({
  modelComplexity: 1,
  smoothLandmarks: true,
  enableSegmentation: false
});

// 결과 처리
pose.onResults(results => {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.drawImage(results.image, 0, 0, canvas.width, canvas.height);

  if (results.poseLandmarks) {
    const nose = results.poseLandmarks[0];      // 머리
    const leftHand = results.poseLandmarks[15]; // 왼손

    // 👉 손 들면 배경 빨간색
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
  },
  width: 640,
  height: 480
});
camera.start();
</script>

</body>
</html>
