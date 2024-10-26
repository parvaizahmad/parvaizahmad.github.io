---
title: Virtual Joystick for Arduino Based Controller
description: We can use this web ui based joystick with arduino controller famously with esp8266/esp32.
categories: [demo, arduino, esp]
date: 2024-10-26 14:03:00 +500
tags: [arduino, esp8266, esp32, joystick, controller] # TAG names should always be lowercase
pin: false
math: false
mermaid: false
image:
  path: https://ik.imagekit.io/yhmzg9hf6/joystick.png?updatedAt=1729932857297
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: "Virtual Joystick"
---

# Joystick Controller

A joystick controller is an input device used to control video games, computer graphics, and other applications that require directional control. It typically consists of a stick that pivots on a base and reports its angle or direction to the device it is controlling. Joystick controllers can be found in various forms, including analog sticks, digital joysticks, and virtual joysticks.

## Purpose and Uses

### Gaming

Joystick controllers are widely used in gaming, providing an intuitive way to control characters, vehicles, and other elements within a game. They offer precise control over movement and actions, making them ideal for flight simulators, racing games, and other genres that require fine-tuned input.

### Robotics

In robotics, joystick controllers are used to manually control robots. This can be particularly useful in teleoperation scenarios where a human operator needs to control a robot remotely. Joysticks provide a straightforward way to translate human input into robotic movement.

### Industrial Applications

Joystick controllers are also used in various industrial applications, such as controlling cranes, forklifts, and other heavy machinery. They allow operators to perform complex maneuvers with ease and precision.

### Assistive Technology

For individuals with disabilities, joystick controllers can serve as an alternative input method for computers and other devices. They can be customized to suit the user's needs, providing greater accessibility and independence.

### Virtual Reality (VR) and Augmented Reality (AR)

In VR and AR applications, joystick controllers are used to navigate virtual environments and interact with virtual objects. They enhance the immersive experience by providing a natural and responsive way to control the virtual world.

## Virtual Joystick for Arduino

I designed a virtual joystick controller for use with ESP8266 and ESP32 microcontrollers. This innovative, web-based joystick leverages WebSockets to facilitate real-time communication with the server, ensuring responsive and seamless control. The virtual joystick has been rigorously tested with both ESP8266 and ESP32, demonstrating its versatility and reliability in various applications. By utilizing WebSockets, the controller provides instantaneous feedback and interaction, making it an ideal solution for projects requiring precise and real-time control. This advanced technology opens up new possibilities for remote operation, enhancing user experience and expanding the potential use cases for joystick controllers.

## Using Virtual Joystick

Using a virtual joystick is straightforward. It employs a server-client model, but instead of using HTTP, it uses WebSocket for communication. WebSocket is fast, full-duplex, and real-time.

### Adding Dependencies

To use this joystick, you need to install ESP32 or ESP8266 board support in the Arduino IDE. Install the support according to your needs and the availability of the board. Navigate to the Board Manager and install the appropriate board.

This project also depends on two external libraries: `WebSocketsServer` and `ArduinoJson`. You need to install both of these libraries to build and use the code.

Include the headers for the libraries as follows:

```cpp
  #include <WebSocketsServer.h>
  #include <ArduinoJson.h>
```

### Working of Joystick

The ESP32/8266 hosts the web page source code for the joystick. The ESP is configured in both station and AP modes, allowing you to connect directly to the ESP or use your Wi-Fi to connect to the controller. mDNS is also set up on the ESP with the hostname `joystick`. After the system is up and connected to Wi-Fi, you need to enter `http://joystick.local` in the browser of your mobile device or PC to access the joystick controller.

## Building and Uploading

Select the correct board and COM port in Arduino ide and select the upload button to build and upload the below code.
Before building and upload code you should consider changing the below mentioned setting in code.

- Select the correct board

```c
  // Define the board type
  #define ESP8266_BOARD 1
  #define ESP32_BOARD 2

  // Select the board type here
  #define BOARD_TYPE ESP32_BOARD // Change to ESP32_BOARD for ESP32
```

- WiFi SSID name and password

```cpp
  // Connect to the external WiFi network (STA mode)
  WiFiMulti.addAP("SSID", "PASSWORD");
```

- AP name and password

```cpp
  // Set up AP mode
  WiFi.softAP("Joystick", "password123"); // Set your AP SSID and password
```

- mDNS hostname if you want to use any other host name instead of `joystick.local`

```cpp
  // Start mDNS service
  String hostname = "joystick";
```

### Code of Web-based Virtual Joystick

Code is hosted on github but I will also provide the code in this blog post.

#### Github Repo

Click on the link to go to github repo:
[GitHub Repo Link](https://github.com/parvaizahmad/virtualJoystickESP)

#### Complete Code

Complete code for copy paste purpose is below.

```cpp

  // Define the board type
  #define ESP8266_BOARD 1
  #define ESP32_BOARD 2

  // Select the board type here
  #define BOARD_TYPE ESP32_BOARD // Change to ESP32_BOARD for ESP32

  #if BOARD_TYPE == ESP8266_BOARD
  #include <ESP8266WiFi.h>
  #include <ESP8266WebServer.h>
  #include <ESP8266WiFiMulti.h>
  #include <ESP8266mDNS.h>
  ESP8266WebServer server(80); // Define server for ESP8266
  ESP8266WiFiMulti WiFiMulti;  // Define WiFiMulti for ESP8266
  #elif BOARD_TYPE == ESP32_BOARD
  #include <WiFi.h>
  #include <WebServer.h>
  #include <WiFiMulti.h>
  #include <ESPmDNS.h>
  WebServer server(80); // Define server for ESP32
  WiFiMulti WiFiMulti;  // Define WiFiMulti for ESP32
  #endif

  #include <WebSocketsServer.h>
  #include <ArduinoJson.h>

  const char index_html[] PROGMEM = R"rawliteral(
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=8.0" />
      <title>ESP8266/32 Joystick Control</title>
      <style>
          body {
              font-family: 'Arial', sans-serif;
              display: flex;
              flex-direction: column;
              align-items: center;
              justify-content: top;
              height: 100vh;
              margin: 2px;
              padding: 5px;
              background-color: #f0f0f0;
              box-sizing: border-box;

          }

          .container {
              display: flex;
              flex-direction: column;
              align-items: center;
              gap: 5px;
              padding: 0px 15px 0px 15px;
              background: linear-gradient(135deg, #f0f0f0, #e0e0e0);
              border: 1px solid #ccc;
              border-radius: 15px;
              box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
              margin: 10px auto;
              max-width: 300px;
              user-select: none;
              height: auto;

          }

          .joystick-container {
              position: relative;
              width: 200px;
              height: 200px;
              background: radial-gradient(circle, #f0f0f0, #ccc);
              border-radius: 50%;
              box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
              border: 1px solid #918181;
              display: flex;
              justify-content: center;
              align-items: center;

          }

          .joystick-container::before,
          .joystick-container::after {
              content: '';
              position: absolute;
              background-color: #cfcfcf;
          }

          .joystick-container::before {
              width: 1px;
              height: 100%;
              top: 0;
              left: 50%;
              transform: translateX(-50%);
          }

          .joystick-container::after {
              width: 100%;
              height: 1px;
              top: 50%;
              left: 0;
              transform: translateY(-50%);
          }

          .joystick {
              position: absolute;
              width: 100px;
              height: 100px;
              background: radial-gradient(circle, #007bff, #0056b3);
              border-radius: 50%;
              top: 50%;
              left: 50%;
              transform: translate(-50%, -50%);
              box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
              touch-action: none;
              border: 5px solid #b9b8b8;
              z-index: 1;

          }

          /* .joystick::before {
              content: '';
              position: absolute;
              width: 30px;
              height: 30px;
              background-color: radial-gradient(circle, #008f99, #01d8f5);;
              border-radius: 50%;
              top: 50%;
              left: 50%;
              transform: translate(-50%, -50%);
              box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
          } */

          .sliders {
              display: flex;
              flex-direction: column;
              justify-content: center;
              align-items: center;
              padding: 10px;
              gap: 10px;
          }
          .slider-container {
              display: flex;
              flex-direction: column;
              align-items: center;
          }

          .slider {
              -webkit-appearance: none;
              appearance: none;
              width: 300px;
              height: 15px;
              background: linear-gradient(to right, green, red);
              outline: none;
              opacity: 0.7;
              transition: opacity .2s;
              border-radius: 10px;
          }
          .slider:hover {
              opacity: 1;
          }
          .slider::-webkit-slider-thumb {
              -webkit-appearance: none;
              appearance: none;
              width: 25px;
              height: 25px;
              background: #4CAF50;
              cursor: pointer;
              border-radius: 5px;
          }
          .slider::-moz-range-thumb {
              width: 25px;
              height: 30px;
              background: #4CAF50;
              cursor: pointer;
              border-radius: 5px;
          }

          .buttons {
              display: flex;
              justify-content: space-evenly;
              width: 100%;
              flex-wrap: wrap;
          }

          .button {
              width: 65px;
              height: 40px;
              background-color: #28a745;
              border: none;
              color: white;
              font-size: 24px;
              border-radius: 10px;
              box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
              cursor: pointer;
              transition: background-color 0.3s ease;
              margin: 5px;

          }

          .button.switch-on {
              background-color: #ff0707;
          }

          .button.push-active {
              background-color: #ff0707;
          }
          .common-label {
              font-size: 16px;
              font-weight: bold;
              margin-right: 1px;
              color: #333;
          }

          .toggle-checkbox {
              width: 20px;
              height: 20px;
              cursor: pointer;
              accent-color: #007bff;
          }
          .check-button-container {
              display: flex;
              justify-content: left;
              width: 100%;
              align-items: center;
              gap: 5px;
          }

          .nav-arrows {
              display: grid;
              grid-template-columns: repeat(3, 50px);
              grid-template-rows: repeat(3, 50px);
              gap: 10px;
              justify-content: center;
              align-items: center;
              margin: 2px auto;
              width: max-content;
          }

          .arrow {
              display: flex;
              justify-content: center;
              align-items: center;
              width: 50px;
              height: 50px;
              background-color: #28a745;
              color: white;
              font-size: 24px;
              border-radius: 5px;
              cursor: pointer;
              user-select: none;
              box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
              transition: background-color 0.2s;
              border: none;
          }

          .arrow:active{
              outline: none;
              background-color: #ff0707;
          }

          .arrow.up {
              grid-column: 2;
              grid-row: 1;
          }

          .arrow.left {
              grid-column: 1;
              grid-row: 2;
          }

          .arrow.stop {
              grid-column: 2;
              grid-row: 2;

          }

          .arrow.right {
              grid-column: 3;
              grid-row: 2;
          }

          .arrow.down {
              grid-column: 2;
              grid-row: 3;
          }

          .status-label {
              font-size: 14px;
              font-weight: bold;
              padding: 5px 10px;
              border-radius: 5px;
              color: #fff;
              user-select: none;
              grid-column: 3;
              grid-row: 1;
              justify-content: center;
          }

          .status-connected {
              background-color: #28a745;
          }

          .status-disconnected {
              background-color: #dc3545;
          }

          .data-container {
              display: flex;
              flex-direction: column;
              align-items: flex-start;
              margin: 5px 0;
              width: 100%;
              max-width: 400px;
          }

          .data-box {
              width: 100%;
              height: 20px;
              font-size: 12px;
              font-family: 'Arial', sans-serif;
              font-style: italic;
              color: #333;
              background-color: transparent;
              border: 1px solid transparent;
              border-radius: 5px;
              resize: none;
              box-shadow: none;
              overflow-y: auto;
          }

          .data-box:focus {
              outline: none;
              border-color: #007bff;
          }
      </style>
  </head>
  <body>
      <div class="container">
          <div id="status" class="status-label status-disconnected">Disconnected</div>
          <div class="data-container">
              <div id="dataBox" class="data-box"></div>
          </div>

          <div class="buttons">
              <button id="switch1" class="button" onclick="toggleSwitch(1)">&#8961</button>
              <button id="switch2" class="button" onclick="toggleSwitch(2)">&#8961</button>
              <button id="switch3" class="button" onclick="toggleSwitch(3)">&#8961</button>
              <button id="switch4" class="button" onclick="toggleSwitch(4)">&#8961</button>
              <button id="push1" class="button" onmousedown="sendPushData(1, 1)" onmouseup="sendPushData(1, 0)" ontouchstart="sendPushData(1, 1)" ontouchend="sendPushData(1, 0)">&#9085</button>
              <button id="push2" class="button" onmousedown="sendPushData(2, 1)" onmouseup="sendPushData(2, 0)" ontouchstart="sendPushData(2, 1)" ontouchend="sendPushData(2, 0)">&#9085</button>
              <button id="push3" class="button" onmousedown="sendPushData(3, 1)" onmouseup="sendPushData(3, 0)" ontouchstart="sendPushData(3, 1)" ontouchend="sendPushData(3, 0)">&#9085</button>
              <button id="push4" class="button" onmousedown="sendPushData(4, 1)" onmouseup="sendPushData(4, 0)" ontouchstart="sendPushData(4, 1)" ontouchend="sendPushData(4, 0)">&#9085</button>
          </div>

          <div class="nav-arrows">
              <button id="arrow-up" class="arrow up" onmousedown="sendPushData(5, 1)" onmouseup="sendPushData(5, 0)" ontouchstart="sendPushData(5, 1)" ontouchend="sendPushData(5, 0)">&#9651</button>
              <button id="arrow-left" class="arrow left" onmousedown="sendPushData(8, 1)" onmouseup="sendPushData(8, 0)" ontouchstart="sendPushData(8, 1)" ontouchend="sendPushData(8, 0)">&#9665</button>
              <button id="arrow-stop" class="arrow stop" onmousedown="sendPushData(6, 1)" onmouseup="sendPushData(6, 0)" ontouchstart="sendPushData(6, 1)" ontouchend="sendPushData(6, 0)">&#9711</button>
              <button id="arrow-right" class="arrow right" onmousedown="sendPushData(9, 1)" onmouseup="sendPushData(9, 0)" ontouchstart="sendPushData(9, 1)" ontouchend="sendPushData(9, 0)">&#9655</button>
              <button id="arrow-down" class="arrow down" onmousedown="sendPushData(7, 1)" onmouseup="sendPushData(7, 0)" ontouchstart="sendPushData(7, 1)" ontouchend="sendPushData(7, 0)">&#9661</button>
          </div>

          <div class="sliders">
              <div class="slider-container">
                  <label class="common-label" id="labelslider1" for="slider1">Slider 1: 0</label>
                  <input type="range" id="slider1" class="slider" min="0" max="100" value="0" oninput="sendSlider1Data()">
              </div>
              <div class="slider-container">
                  <label class="common-label" id="labelslider2" for="slider2">Slider 2: 0</label>
                  <input type="range" id="slider2" class="slider" min="0" max="100" value="0" oninput="sendSlider2Data()">
              </div>
          </div>

          <div id="controllerContainer">
              <div class="joystick-container" id="joystickContainer">
                  <div class="joystick" id="joystick"></div>
              </div>
          </div>
          <div class="check-button-container">
              <label class="common-label" for="toggle-return">RTH:</label>
              <input class="toggle-checkbox" type="checkbox" id="toggle-return" checked onchange="handleToggleChange()">
              <label class="common-label" for="toggle-sl">SL:</label>
              <input class="toggle-checkbox" type="checkbox" id="toggle-sl" unchecked onchange="handleSL()">
          </div>

      </div>

      <script>
          const joystick = document.getElementById('joystick');
          const joystickContainer = document.querySelector('.joystick-container');
          const slider1 = document.getElementById('slider1');
          const slider2 = document.getElementById('slider2');
          const toggleReturn = document.getElementById('toggle-return');
          const statusLabel = document.getElementById('status');

          let joystickOffset = { x: 0, y: 0 };
          let isDragging = false;
          let xPos = 0;
          let yPos = 0;

          let socket;

          // Initialize WebSocket
          function initWebSocket() {
              socket = new WebSocket('ws://' + location.hostname + ':81');

              socket.onopen = function() {
                  console.log("WebSocket connection opened");
                  updateStatus(true);
              };

              socket.onmessage = function(event) {
                  console.log("Message from server: ", event.data);
                  updateDataBox(event.data);
              };

              socket.onclose = function() {
                  console.log("WebSocket connection closed");
                  updateStatus(false);
              };

              socket.onerror = function(error) {
                  console.error("WebSocket error: ", error);
                  updateStatus(false);
              };
          }

          function updateDataBox(data) {
              const dataBox = document.getElementById('dataBox');
              dataBox.textContent = data;
          }
          function sendJoystickData(x, y) {
              const data = JSON.stringify({ type: 'joystick', x: Math.round(x*125), y: Math.round(y*(125)) });
              socket.send(data);
          }

          function sendSlider1Data() {
              const data = JSON.stringify({ slider1: slider1.value });
              document.getElementById('labelslider1').innerHTML = "Slider 1: " + slider1.value;
              socket.send(data);
          }

          function sendSlider2Data() {
              const data = JSON.stringify({ slider2: slider2.value });
              document.getElementById('labelslider2').innerHTML = "Slider 2: " + slider2.value;
              socket.send(data);
          }

          function initSliders() {
              document.getElementById('slider1').value = 0;
              document.getElementById('slider2').value = 0;
          }
          initSliders();

          function toggleSwitch(id) {
              const button = document.getElementById('switch' + id);
              const isOn = button.classList.toggle('switch-on');
              const value = isOn ? 1 : 0;
              const data = JSON.stringify({ type: 'switch', id: id, value: value });
              socket.send(data);
          }
          function sendPushData(id, value) {
              const button = document.getElementById('push' + id);
              const data = JSON.stringify({ type: 'push', id: id, value: value });
              socket.send(data);
              if (value === 1) {
                  button.classList.add('push-active');
              } else {
                  button.classList.remove('push-active');
              }
          }

          joystick.addEventListener('mousedown', startDrag);
          joystick.addEventListener('touchstart', startDrag, { passive: true });


          function startDrag(event) {
              event.preventDefault();
              isDragging = true;
              joystick.style.transition = 'left 0s, top 0s';

              document.addEventListener('mousemove', dragJoystick);
              document.addEventListener('mouseup', stopDrag);
              document.addEventListener('touchmove', dragJoystick, { passive: true });
              document.addEventListener('touchend', stopDrag);

              const rect = joystickContainer.getBoundingClientRect();

              if (event.touches) {
                  joystickOffset.x = event.touches[0].clientX - rect.left - xPos;
                  joystickOffset.y = event.touches[0].clientY - rect.top - yPos;
              } else {
                  joystickOffset.x = event.clientX - rect.left - xPos;
                  joystickOffset.y = event.clientY - rect.top - yPos;
              }
          }

          function dragJoystick(event) {
              if (!isDragging) return;

              const rect = joystickContainer.getBoundingClientRect();
              const limitRadius = rect.width / 2 - joystick.offsetWidth / 2;

              if (event.touches) {
                  xPos = event.touches[0].clientX - rect.left - joystickOffset.x;
                  yPos = event.touches[0].clientY - rect.top - joystickOffset.y;
              } else {
                  xPos = event.clientX - rect.left - joystickOffset.x;
                  yPos = event.clientY - rect.top - joystickOffset.y;
              }

              const distance = Math.sqrt(xPos * xPos + yPos * yPos);
              if (distance > limitRadius) {
                  const angle = Math.atan2(yPos, xPos);
                  xPos = limitRadius * Math.cos(angle);
                  yPos = limitRadius * Math.sin(angle);
              }

              joystick.style.left = `${xPos + rect.width / 2}px`;
              joystick.style.top = `${yPos + rect.height / 2}px`;

              sendJoystickData(xPos / limitRadius, yPos / limitRadius);
          }

          function stopDrag() {
              isDragging = false;
              document.removeEventListener('mousemove', dragJoystick);
              document.removeEventListener('mouseup', stopDrag);
              document.removeEventListener('touchmove', dragJoystick);
              document.removeEventListener('touchend', stopDrag);
              joystick.style.transition = 'left 0.4s, top 0.4s';
              if (toggleReturn.checked) {
                  sendJoystickData(0, 0);
                  xPos = 0;
                  yPos = 0;
                  joystick.style.left = '50%';
                  joystick.style.top = '50%';

              }
          }

          function updateStatus(isConnected) {
              if (isConnected) {
                  statusLabel.textContent = 'Connected';
                  statusLabel.classList.remove('status-disconnected');
                  statusLabel.classList.add('status-connected');
              } else {
                  statusLabel.textContent = 'Disconnected';
                  statusLabel.classList.remove('status-connected');
                  statusLabel.classList.add('status-disconnected');
              }
          }

          function handleToggleChange() {
              if (toggleReturn.checked) {
                  sendJoystickData(0, 0);
                  xPos = 0;
                  yPos = 0;
                  joystick.style.left = '50%';
                  joystick.style.top = '50%';
              } else {
                  console.log('Return to Home is disabled');
              }
          }

          function handleSL() {
              if (document.getElementById('toggle-sl').checked) {
                  document.querySelector('.container').style.touchAction = 'none';
              } else {
                  document.querySelector('.container').style.touchAction = 'auto';
              }
          }
          //alert("By defult, the return to home is enabled. If you want to disable it, uncheck the RTH checkbox. The SL checkbox is for disabling the touch action of the screen. If you want to enable the touch action, uncheck the SL checkbox.");
          window.onload = initWebSocket;
      </script>
  </body>
  </html>
  )rawliteral";

  #if BOARD_TYPE == ESP8266_BOARD
  unsigned long lastMDNSUpdate = 0;
  const unsigned long MDNS_UPDATE_INTERVAL = 100; // Update mDNS every 100 mili second
  #endif

  WebSocketsServer webSocket = WebSocketsServer(81);

  #define in1 23 // 14
  #define in2 22 // 12
  #define in3 19 // 13
  #define in4 18 // 15
  #define enA 17 // 17
  #define enB 16 // 16

  // setting PWM properties
  const int freq = 500;
  const int resolution = 8;
  int speedSet = 100;


  void moveControl(int x, int y, uint8_t num)
  {
      int analogWriteX = map(abs(x), 0, 127, 10, speedSet);
      int analogWriteY = map(abs(y), 0, 127, 10, speedSet);
      int buffer = 30;
      // Move Forward
      if (y <= -buffer)
      {
          webSocket.sendTXT(num, "Moving Forward");
          ledcWrite(enA, analogWriteY);
          ledcWrite(enB, analogWriteY);
          digitalWrite(in1, LOW);
          digitalWrite(in2, HIGH);
          digitalWrite(in3, LOW);
          digitalWrite(in4, HIGH);
      }

      // Move Forward Right
      if (x >= buffer && y <= -buffer)
      {
          webSocket.sendTXT(num, "Moving Forward Right");
          ledcWrite(enA, analogWriteX);
          ledcWrite(enB, analogWriteY);
          digitalWrite(in1, LOW);
          digitalWrite(in2, HIGH);
          digitalWrite(in3, LOW);
          digitalWrite(in4, HIGH);
      }

      // Move Forward Left
      if (x <= -buffer && y <= -buffer)
      {
          webSocket.sendTXT(num, "Moving Forward Left");
          ledcWrite(enA, analogWriteY);
          ledcWrite(enB, analogWriteX);
          digitalWrite(in1, LOW);
          digitalWrite(in2, HIGH);
          digitalWrite(in3, LOW);
          digitalWrite(in4, HIGH);
      }

      // No Move
      if (y == 0 && x == 0)
      {
          webSocket.sendTXT(num, "STOP");
          ledcWrite(enA, analogWriteY);
          ledcWrite(enB, analogWriteY);
          digitalWrite(in1, LOW);
          digitalWrite(in2, LOW);
          digitalWrite(in3, LOW);
          digitalWrite(in4, LOW);
      }

      // Move Backward
      if (y >= buffer && x <= buffer)
      {
          webSocket.sendTXT(num, "Moving Backward");
          ledcWrite(enA, analogWriteY);
          ledcWrite(enB, analogWriteY);
          digitalWrite(in1, HIGH);
          digitalWrite(in2, LOW);
          digitalWrite(in3, HIGH);
          digitalWrite(in4, LOW);
      }

      // Move Backward Right
      if (y >= buffer && x >= buffer)
      {
          webSocket.sendTXT(num, "Moving Backward Right");
          ledcWrite(enA, analogWriteX);
          ledcWrite(enB, analogWriteY);
          digitalWrite(in1, HIGH);
          digitalWrite(in2, LOW);
          digitalWrite(in3, HIGH);
          digitalWrite(in4, LOW);
      }

      // Move Backward Left
      if (y >= buffer && x <= -buffer)
      {
          webSocket.sendTXT(num, "Moving Backward Left");
          ledcWrite(enA, analogWriteY);
          ledcWrite(enB, analogWriteX);
          digitalWrite(in1, HIGH);
          digitalWrite(in2, LOW);
          digitalWrite(in3, HIGH);
          digitalWrite(in4, LOW);
      }
  }

  void handleRoot()
  {
      server.send(200, "text/html", index_html);
  }

  void handleWebSocketMessage(uint8_t *data, size_t len, uint8_t num)
  {
      // Allocate the JSON document
      StaticJsonDocument<200> doc;

      // Deserialize the JSON document
      DeserializationError error = deserializeJson(doc, data, len);

      // Test if parsing succeeds
      if (error)
      {
          Serial.print(F("deserializeJson() failed: "));
          Serial.println(error.f_str());
          return;
      }

      // Handle the received data based on type
      if (doc["type"] == "joystick")
      {
          int x = doc["x"].as<int>();
          int y = doc["y"].as<int>();
          moveControl(x, y, num);
      }
      else if (doc["slider1"].as<int>() >= 0)
      {
          int sliderValue = doc["slider1"].as<int>();
          speedSet = map(sliderValue, 0, 100, 100, 255);
          webSocket.sendTXT(num, "Speed set to: " + String(speedSet));
      }
      else if (doc["slider2"].as<int>() >= 0)
      {
          int sliderValue = doc["slider2"].as<int>();
          webSocket.sendTXT(num, "Slider 2 value: " + String(sliderValue));
      }
      else if (doc["type"] == "switch")
      {
          int id = doc["id"].as<int>();
          int value = doc["value"].as<int>();
          webSocket.sendTXT(num, "Switch data received: ID: " + String(id) + " Value: " + String(value));
      }
      else if (doc["type"] == "push")
      {
          int id = doc["id"].as<int>();
          int value = doc["value"].as<int>();
          webSocket.sendTXT(num, "Push button data received: ID: " + String(id) + " Value: " + String(value));
      }
      else
      {
          Serial.println("Unknown data received");
          webSocket.sendTXT(num, "Unknown data received");
      }
  }

  void onWebSocketEvent(uint8_t num, WStype_t type, uint8_t *payload, size_t length)
  {
      switch (type)
      {
      case WStype_DISCONNECTED:
          Serial.printf("[%u] Disconnected!\n", num);
          break;
      case WStype_CONNECTED:
      {
          IPAddress ip = webSocket.remoteIP(num);
          Serial.printf("[%u] Connection from ", num);
          Serial.println(ip.toString());
          break;
      }
      case WStype_TEXT:
          //webSocket.sendTXT(num, payload);
          handleWebSocketMessage(payload, length, num);
          break;
      case WStype_BIN:
          Serial.printf("[%u] get binary length: %u\n", num, length);
          break;
      case WStype_PING:
          Serial.printf("[%u] Received PING\n", num);
          break;
      case WStype_PONG:
          Serial.printf("[%u] Received PONG\n", num);
          break;
      case WStype_ERROR:
          Serial.printf("[%u] WebSocket Error!\n", num);
          break;
      }
  }

  void setup()
  {
      Serial.begin(115200);

      // configure LED PWM functionalitites
      //ledcSetup(ledChannel, freq, resolution);

      // attach the channel to the GPIO to be controlled
      ledcAttach(enA, freq, resolution);
      ledcAttach(enB, freq, resolution);

      pinMode(in1, OUTPUT);
      pinMode(in3, OUTPUT);
      pinMode(in2, OUTPUT);
      pinMode(in4, OUTPUT);

      digitalWrite(in1, LOW);
      digitalWrite(in3, LOW);
      digitalWrite(in2, LOW);
      digitalWrite(in4, LOW);

      // Connect to the external WiFi network (STA mode)
      WiFiMulti.addAP("SSID", "PASSWORD");

      // Wait for the ESP to connect to WiFi
      while (WiFiMulti.run() != WL_CONNECTED)
      {
          delay(100);
      }

      // Print the IP address
      Serial.print("Connected to ");
      Serial.println(WiFi.SSID());
      Serial.print("IP address: ");
      Serial.println(WiFi.localIP());

      // Start mDNS service
      String hostname = "joystick";
      if (MDNS.begin(hostname.c_str()))
      {
          Serial.printf("mDNS responder started with hostname: %s.local\n", hostname.c_str());
      }
      else
      {
          Serial.println("Error setting up mDNS responder!");
      }

      // Set up AP mode
      WiFi.softAP("Joystick", "password123"); // Set your AP SSID and password
      IPAddress myIP = WiFi.softAPIP();
      Serial.print("AP IP address: ");
      Serial.println(myIP);

      // Start the HTTP server
      server.on("/", handleRoot);
      server.begin();
      Serial.println("HTTP server started");

      // Start the WebSocket server
      webSocket.begin();
      webSocket.onEvent(onWebSocketEvent);
      Serial.println("WebSocket server started");
  }

  void loop()
  {
      server.handleClient();
      webSocket.loop();

  #if BOARD_TYPE == ESP8266_BOARD
      // Handle mDNS queries at a reduced frequency
      unsigned long currentMillis = millis();
      if (currentMillis - lastMDNSUpdate >= MDNS_UPDATE_INTERVAL)
      {
          lastMDNSUpdate = currentMillis;
          MDNS.update();
      }
  #endif
  }


```
