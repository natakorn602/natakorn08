# natakorn08
<!DOCTYPE html>
<html>
<head>
  <title>Rice & Servo Control</title>

  <!-- สำคัญมากสำหรับมือถือ -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>

  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #74ebd5, #ACB6E5);
      min-height: 100vh;
      margin: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 15px;
    }

    .card {
      background: white;
      padding: 25px;
      border-radius: 20px;
      box-shadow: 0 10px 25px rgba(0,0,0,0.2);
      text-align: center;
      width: 100%;
      max-width: 400px;
    }

    h2 {
      margin-bottom: 10px;
      font-size: clamp(18px, 4vw, 22px);
    }

    #timer {
      font-size: clamp(28px, 7vw, 40px);
      font-weight: bold;
      margin: 15px 0;
      color: #333;
    }

    .button-group {
      display: flex;
      justify-content: center;
      gap: 10px;
      flex-wrap: wrap;
    }

    button {
      padding: 12px 20px;
      border: none;
      border-radius: 10px;
      font-size: 16px;
      cursor: pointer;
      transition: 0.2s;
      flex: 1;
      min-width: 120px;
    }

    .onBtn {
      background-color: #4CAF50;
      color: white;
    }

    .offBtn {
      background-color: #f44336;
      color: white;
    }

    /* ---------------- BATTERY ---------------- */

    .battery {
      width: clamp(70px, 20vw, 90px);
      height: clamp(140px, 40vw, 180px);
      border: 4px solid #333;
      border-radius: 12px;
      margin: 20px auto;
      padding: 6px;
      display: flex;
      flex-direction: column-reverse;
      justify-content: space-between;
      background: #eee;
      position: relative;
    }

    .battery::after {
      content: "";
      width: 35%;
      height: 10px;
      background: #333;
      position: absolute;
      top: -14px;
      left: 50%;
      transform: translateX(-50%);
      border-radius: 4px;
    }

    .level {
      height: 30%;
      border-radius: 6px;
      background-color: red;
      transition: 0.4s;
    }

    .active {
      background-color: #4CAF50;
      box-shadow: 0 0 12px #4CAF50;
    }

    .label {
      margin-top: 5px;
      font-size: clamp(14px, 3.5vw, 16px);
      font-weight: bold;
    }

    hr {
      margin: 20px 0;
    }

    /* ---------- Mobile Extra ---------- */
    @media (max-width: 480px) {
      .card {
        padding: 20px;
        border-radius: 15px;
      }

      button {
        font-size: 15px;
      }
    }

  </style>
</head>

<body>

<div class="card">
  <h2>Rice Level Monitor</h2>

  <div class="battery">
    <div id="level3" class="level"></div>
    <div id="level2" class="level"></div>
    <div id="level1" class="level"></div>
  </div>

  <div class="label">ระดับข้าวสาร</div>

  <hr>

  <h2>Servo Control</h2>
  <div id="timer">30:00</div>

  <div class="button-group">
    <button class="onBtn" onclick="sendOn()">ON</button>
    <button class="offBtn" onclick="sendOff()">OFF</button>
  </div>
</div>

<script>
const client = mqtt.connect("wss://test.mosquitto.org:8081");

client.on("connect", function () {
  console.log("Connected MQTT");

  client.subscribe("visawa/rice/level1");
  client.subscribe("visawa/rice/level2");
  client.subscribe("visawa/rice/level3");
});

client.on("message", function(topic, message){
  let msg = message.toString();

  if(topic === "visawa/rice/level1"){
    updateLevel("level1", msg);
  }

  if(topic === "visawa/rice/level2"){
    updateLevel("level2", msg);
  }

  if(topic === "visawa/rice/level3"){
    updateLevel("level3", msg);
  }
});

function updateLevel(id, status){
  const element = document.getElementById(id);
  if(status === "ON"){
    element.classList.add("active");
  } else {
    element.classList.remove("active");
  }
}

/* ---------------- TIMER ---------------- */

let countdown;
let totalSeconds = 1800;

function updateDisplay() {
  let minutes = Math.floor(totalSeconds / 60);
  let seconds = totalSeconds % 60;

  document.getElementById("timer").innerText =
    String(minutes).padStart(2,'0') + ":" +
    String(seconds).padStart(2,'0');
}

function startTimer() {
  clearInterval(countdown);
  totalSeconds = 1800;
  updateDisplay();

  countdown = setInterval(() => {
    if (totalSeconds > 0) {
      totalSeconds--;
      updateDisplay();
    } else {
      clearInterval(countdown);
    }
  }, 1000);
}

function resetTimer() {
  clearInterval(countdown);
  totalSeconds = 1800;
  updateDisplay();
}

function sendOn(){
  client.publish("visawa/servo/control","ON");
  startTimer();
}

function sendOff(){
  client.publish("visawa/servo/control","OFF");
  resetTimer();
}

updateDisplay();
</script>

</body>
</html>
