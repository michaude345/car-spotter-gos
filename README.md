<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Car Spotter GO</title>
  <meta name="theme-color" content="#333333" />
  <link rel="manifest" href="manifest.json" />
  <link rel="icon" href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACwAAAAuCAYAAACw+VtbAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAGXRFWHRTb2Z0d2FyZQB3d3cuaW5rc2NhcGUub3Jnm+48GgAAAQZJREFUWIXt1rENg0AURNEPK6gBcQGcQEcQEcQOcQPcAPyg+Pgpy+ihSHKzkJLyG3Mb43xxECmxtvMW0JcZ1RbQExgRJhYQkwlI0sx3G/4fY5E4AbAhwDMAGcBbwjXKqO3lPqQqMJfwU9oym0RfKgTcmIfgUtobZXc5t2UYaRyrLcoGNYU7GuC2UOJgDETVFzV6HV70Auc02C1tCXK4kksSqOyHJHZt0K6cUwrzIsKqZcvlUos7n6AYd5Vce0oR4QAAAAAElFTkSuQmCC" />
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      background: #f4f4f4;
      color: #333;
      margin: 0;
      padding: 0 10px 40px 10px;
    }
    h1 {
      margin-top: 20px;
      font-size: 2em;
    }
    video {
      width: 90%;
      max-width: 500px;
      border: 2px solid #333;
      margin-top: 10px;
      border-radius: 8px;
    }
    #inputSection {
      margin-top: 15px;
    }
    input {
      padding: 10px;
      width: 60%;
      max-width: 300px;
      font-size: 1em;
      border-radius: 4px;
      border: 1px solid #ccc;
    }
    button {
      padding: 10px 20px;
      font-size: 1em;
      margin-left: 10px;
      cursor: pointer;
      border-radius: 4px;
      border: none;
      background-color: #333;
      color: white;
      transition: background-color 0.3s ease;
    }
    button:hover {
      background-color: #555;
    }
    #results {
      margin-top: 20px;
      max-width: 600px;
      margin-left: auto;
      margin-right: auto;
      text-align: left;
    }
    ul {
      list-style-type: none;
      padding: 0;
      max-height: 300px;
      overflow-y: auto;
      border: 1px solid #ddd;
      background: white;
      border-radius: 6px;
    }
    ul li {
      padding: 8px 12px;
      border-bottom: 1px solid #eee;
    }
    ul li:last-child {
      border-bottom: none;
    }
  </style>
</head>
<body>
  <h1>🚗 Car Spotter GO</h1>
  <video id="camera" autoplay playsinline></video>

  <div id="inputSection">
    <input type="text" id="carInput" placeholder="Ex: Toyota Supra 1998" />
    <button onclick="scanCar()">Scanner</button>
    <button onclick="scanCarAuto()">Scan IA</button>
  </div>

  <div id="results">
    <h2>📋 Voitures repérées :</h2>
    <ul id="carList"></ul>
  </div>

  <!-- TensorFlow.js + MobileNet -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.18.0"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/mobilenet"></script>

  <script>
    const camera = document.getElementById("camera");
    const carList = document.getElementById("carList");
    let model;
    let spottedCars = [];

    const rareCars = ["Bugatti", "Ferrari", "McLaren", "Lamborghini", "DeLorean", "Supra", "Skyline"];

    async function startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        camera.srcObject = stream;
      } catch (error) {
        alert("Caméra non disponible.");
      }
    }

    async function loadModel() {
      model = await mobilenet.load();
      console.log("✅ Modèle MobileNet chargé");
    }

    function scanCar() {
      const input = document.getElementById("carInput").value.trim();
      if (!input) {
        alert("Entre un nom de voiture !");
        return;
      }

      let isRare = rareCars.some((brand) =>
        input.toLowerCase().includes(brand.toLowerCase())
      );
      let points = isRare ? 100 : 10;

      spottedCars.push({ name: input, points });
      updateCarList();
      document.getElementById("carInput").value = "";
    }

    async function scanCarAuto() {
      if (!model) {
        alert("L'IA n'est pas encore prête, attends quelques secondes...");
        return;
      }

      const prediction = await model.classify(camera);
      if (prediction.length > 0) {
        const name = prediction[0].className;
        const isRare = rareCars.some((brand) =>
          name.toLowerCase().includes(brand.toLowerCase())
        );
        const points = isRare ? 100 : 10;

        spottedCars.push({ name, points });
        updateCarList();
      } else {
        alert("Aucune voiture détectée.");
      }
    }

    function updateCarList() {
      carList.innerHTML = "";
      spottedCars.forEach((car) => {
        const li = document.createElement("li");
        li.textContent = `${car.name} → ${car.points} pts`;
        carList.appendChild(li);
      });
    }

    startCamera();
    loadModel();
  </script>
</body>
</html>
