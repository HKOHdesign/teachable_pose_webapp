<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Teachable Machine Pose WebApp</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background: #f0f4f8;
      text-align: center;
      padding: 30px;
    }

    .title {
      font-size: 2rem;
      font-weight: bold;
      margin-bottom: 20px;
      color: #333;
    }

    .start-button {
      background: #4f46e5;
      color: #fff;
      border: none;
      padding: 12px 24px;
      font-size: 1rem;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    .start-button:hover {
      background: #3730a3;
    }

    .canvas-container {
      margin: 20px auto;
      width: 220px;
      height: 220px;
      background: #fff;
      border-radius: 12px;
      padding: 10px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }

    #canvas {
      border-radius: 8px;
      width: 200px;
      height: 200px;
    }

    #label-container {
      margin-top: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;
    }

    #label-container div {
      background: #fff;
      padding: 8px 16px;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      font-size: 0.95rem;
      color: #333;
      width: fit-content;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/pose@0.8/dist/teachablemachine-pose.min.js"></script>
</head>
<body>
  <div class="title">Teachable Machine Pose Model</div>
  <button type="button" class="start-button" onclick="init()">Start</button>
  <div class="canvas-container">
    <canvas id="canvas"></canvas>
  </div>
  <div id="label-container"></div>

  <script>
    const URL = "https://teachablemachine.withgoogle.com/models/Kim9WXS0Q/";
    let model, webcam, ctx, labelContainer, maxPredictions;

    async function init() {
      const modelURL = URL + "model.json";
      const metadataURL = URL + "metadata.json";

      model = await tmPose.load(modelURL, metadataURL);
      maxPredictions = model.getTotalClasses();

      const size = 200;
      const flip = true;
      webcam = new tmPose.Webcam(size, size, flip);
      await webcam.setup();
      await webcam.play();
      window.requestAnimationFrame(loop);

      const canvas = document.getElementById("canvas");
      canvas.width = size;
      canvas.height = size;
      ctx = canvas.getContext("2d");
      labelContainer = document.getElementById("label-container");
      for (let i = 0; i < maxPredictions; i++) {
        labelContainer.appendChild(document.createElement("div"));
      }
    }

    async function loop() {
      webcam.update();
      await predict();
      window.requestAnimationFrame(loop);
    }

    async function predict() {
      const { pose, posenetOutput } = await model.estimatePose(webcam.canvas);
      const prediction = await model.predict(posenetOutput);

      for (let i = 0; i < maxPredictions; i++) {
        const classPrediction =
          prediction[i].className + ": " + prediction[i].probability.toFixed(2);
        labelContainer.childNodes[i].innerHTML = classPrediction;
      }

      drawPose(pose);
    }

    function drawPose(pose) {
      if (webcam.canvas) {
        ctx.drawImage(webcam.canvas, 0, 0);
        if (pose) {
          const minPartConfidence = 0.5;
          tmPose.drawKeypoints(pose.keypoints, minPartConfidence, ctx);
          tmPose.drawSkeleton(pose.keypoints, minPartConfidence, ctx);
        }
      }
    }
  </script>
</body>
</html>
