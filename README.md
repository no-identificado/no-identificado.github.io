<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Acarreo de Minerales Dinámico</title>
  <style>
    #canvasContainer {
      position: relative;
    }
    #drawingCanvas {
      border: 2px solid black;
      cursor: crosshair;
    }
    #yellowSquare {
      position: absolute;
      width: 20px;
      height: 20px;
      background-color: yellow;
      pointer-events: none;
    }
    #buttonPanel {
      background-color: #34495e;
      padding: 10px;
      color: white;
      display: flex;
      align-items: center;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    }
    #buttonPanel button {
      background-color: #5d6d7e;
      color: white;
      border: none;
      padding: 10px;
      cursor: pointer;
      transition: background-color 0.3s;
      border-radius: 4px;
    }
    #buttonPanel button:hover {
      background-color: #85a5cc;
    }
    #buttonPanel label {
      margin-right: 5px;
    }
    #buttonPanel input[type="range"] {
      margin-right: 10px;
    }
    #buttonPanel select {
      margin-right: 10px;
    }
    #title {
      margin-top: 20px;
      font-size: 32px;
      font-weight: bold;
      text-align: center;
      font-family: 'Georgia', serif;
      color: #1c2833;
      text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
    }
    #truckTable {
      margin-top: 20px;
      border-collapse: collapse;
      width: 80%;
      margin-left: auto;
      margin-right: auto;
      box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
    }
    #truckTable th {
      background-color: #f39c12;
      color: white;
      padding: 15px;
      text-align: center;
      border: 1px solid #ddd;
    }
    #truckTable td {
      background-color: #fce5cd;
      border: 1px solid #ddd;
      text-align: center;
      padding: 10px;
    }
    #truckTable tr:nth-child(even) {
      background-color: #f8c471;
    }
  </style>
</head>
<body>
  <div id="title">Acarreo de Minerales Dinámico</div>
  <div>
    <h2>Algoritmo de Acarreo de Minas y Visualización</h2>
    <h3>Entradas de datos</h3>
    <form id="haulageForm">
      <label for="distance1">Distancia Tramo 1 (km):</label>
      <input type="number" id="distance1" name="distance1"><br>

      <label for="emptySpeed1">Velocidad Camion Vacío Tramo 1 (km/h):</label>
      <input type="number" id="emptySpeed1" name="emptySpeed1"><br>

      <label for="loadedSpeed1">Velocidad Camion Cargado Tramo 1 (km/h):</label>
      <input type="number" id="loadedSpeed1" name="loadedSpeed1"><br>

      <label for="distance2">Distancia Tramo 2 (km):</label>
      <input type="number" id="distance2" name="distance2"><br>

      <label for="emptySpeed2">Velocidad Camion Vacío Tramo 2 (km/h):</label>
      <input type="number" id="emptySpeed2" name="emptySpeed2"><br>

      <label for="loadedSpeed2">Velocidad Camion Cargado Tramo 2 (km/h):</label>
      <input type="number" id="loadedSpeed2" name="loadedSpeed2"><br>

      <label for="truckCapacity">Capacidad de Camion 1 (toneladas):</label>
      <input type="number" id="truckCapacity" name="truckCapacity"><br>

      <button type="button" onclick="calculateHaulage()">Calcular</button>
    </form>
    <h3>Resultados</h3>
    <div id="result"></div>
  </div>
  <div id="canvasContainer">
    <canvas id="drawingCanvas" width="800" height="600"></canvas>
    <div id="yellowSquare"></div>
  </div>
  <div id="buttonPanel">
    <button onclick="startMovement()">Empezar Movimiento</button>
    <button onclick="stopMovement()">Detener Movimiento</button>
    <button onclick="clearCanvas()">Limpiar</button>
    <label for="speedControl">Velocidad:</label>
    <input type="range" id="speedControl" min="0" max="20" value="5">
    <label for="colorControl">Color:</label>
    <select id="colorControl">
      <option value="red">Rojo</option>
      <option value="blue">Azul</option>
      <option value="green">Verde</option>
      <option value="black">Negro</option>
    </select>
  </div>
  <table id="truckTable">
    <thead>
      <tr>
        <th>Característica</th>
        <th>Valor Actual</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Combustible (galones)</td>
        <td id="fuel">50</td>
      </tr>
      <tr>
        <td>Velocidad (m/s)</td>
        <td id="speed">30</td>
      </tr>
      <tr>
        <td>Tonelaje (toneladas)</td>
        <td id="tonnage">35</td>
      </tr>
    </tbody>
  </table>
  <script>
    const canvas = document.getElementById('drawingCanvas');
    const ctx = canvas.getContext('2d');
    const yellowSquare = document.getElementById('yellowSquare');
    const speedControl = document.getElementById('speedControl');
    const colorControl = document.getElementById('colorControl');
    let drawing = false;
    let path = [];
    let animationFrameId;
    let index = 0;
    let movementActive = false;

    // Variables for dynamic data
    let fuel = 50;
    let speed = 30;
    let tonnage = 35;
    let updateInterval;

    canvas.addEventListener('mousedown', (e) => {
      drawing = true;
      ctx.strokeStyle = colorControl.value;
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(e.offsetX, e.offsetY);
      path.push({ x: e.offsetX, y: e.offsetY, color: colorControl.value });
    });

    canvas.addEventListener('mousemove', (e) => {
      if (!drawing) return;
      ctx.lineTo(e.offsetX, e.offsetY);
      ctx.stroke();
      path.push({ x: e.offsetX, y: e.offsetY, color: colorControl.value });
    });

    canvas.addEventListener('mouseup', () => {
      drawing = false;
    });

    function startMovement() {
      if (path.length === 0 || movementActive) return;
      movementActive = true;
      index = 0;
      moveSquare();
      updateInterval = setInterval(updateTable, 1000);
    }

    function moveSquare() {
      if (!movementActive) return;
      if (index >= path.length) {
        index = 0;
      }
      if (path[index]) {
        yellowSquare.style.left = path[index].x + 'px';
        yellowSquare.style.top = path[index].y + 'px';
        index += parseInt(speedControl.value);
        animationFrameId = requestAnimationFrame(moveSquare);
      }
    }

    function stopMovement() {
      movementActive = false;
      clearInterval(updateInterval);
      cancelAnimationFrame(animationFrameId);
    }

    function clearCanvas() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      path = [];
      index = 0;
      movementActive = false;
      clearInterval(updateInterval);
      cancelAnimationFrame(animationFrameId);
    }

    function updateTable() {
      fuel = Math.max(30, Math.min(70, fuel + (Math.random() * 2 - 1)));
      speed = Math.max(20, Math.min(50, speed + (Math.random() * 4 - 2)) * speedControl.value / 5);
      tonnage = Math.max(30, Math.min(40, tonnage + (Math.random() * 2 - 1)));

      document.getElementById('fuel').innerText = fuel.toFixed(1);
      document.getElementById('speed').innerText = speed.toFixed(1);
      document.getElementById('tonnage').innerText = tonnage.toFixed(1);
    }

    function calculateHaulage() {
      const distance1 = parseFloat(document.getElementById('distance1').value);
      const emptySpeed1 = parseFloat(document.getElementById('emptySpeed1').value);
      const loadedSpeed1 = parseFloat(document.getElementById('loadedSpeed1').value);
      const distance2 = parseFloat(document.getElementById('distance2').value);
      const emptySpeed2 = parseFloat(document.getElementById('emptySpeed2').value);
      const loadedSpeed2 = parseFloat(document.getElementById('loadedSpeed2').value);
      const truckCapacity = parseFloat(document.getElementById('truckCapacity').value);

      const totalDistance = distance1 + distance2;
      const avgSpeed = (emptySpeed1 + loadedSpeed1 + emptySpeed2 + loadedSpeed2) / 4;
      const haulageTime = totalDistance / avgSpeed;
      const cycleHaulage = haulageTime * truckCapacity;

      document.getElementById('result').innerText = 
        `Tiempo de acarreo: ${haulageTime.toFixed(2)} horas\nCapacidad del ciclo: ${cycleHaulage.toFixed(2)} toneladas`;
    }
  </script>
</body>
</html>
