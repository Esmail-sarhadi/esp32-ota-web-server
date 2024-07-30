# 📶 ESP32 OTA Update with Web Server

This project demonstrates how to set up an ESP32 for Over-The-Air (OTA) updates using a web server. The ESP32 connects to a specified Wi-Fi network and hosts a web interface for uploading new firmware. The web interface includes a login page and an upload page, making it easy to update the firmware wirelessly.

## 📑 Table of Contents 
- [⚙️ Installation](#installation)
- [🚀 Usage](#usage)
- [📂 Project Structure](#project-structure)  
- [🔧 Code Explanation](#code-explanation) 
- [🐞 Troubleshooting](#troubleshooting)   
- [🤝 Contributing](#contributing)     
- [📄 License](#license)   
 
## ⚙️ Installation

To set up this project, you need to have the Arduino IDE installed along with the ESP32 board support. You can install the necessary libraries using the Library Manager in the Arduino IDE.

## 🚀 Usage

1. Clone this repository:
    ```bash
    git clone https://github.com/esmail-sarhadi/esp32-ota-web-server.git
    ```
2. Navigate to the project directory:
    ```bash
    cd esp32-ota-web-server
    ```
3. Open the `ota_web_server.ino` file in the Arduino IDE.
4. Modify the `ssid` and `password` variables with your Wi-Fi credentials.
5. Upload the code to your ESP32 board.
6. Once the ESP32 is connected to the Wi-Fi network, you can access the web interface using the IP address displayed in the Serial Monitor.

## 📂 Project Structure

- `ota_web_server.ino`: Main script containing the code for Wi-Fi connection, web server setup, and OTA update handling.

## 🔧 Code Explanation

### Wi-Fi Connection:
Connects the ESP32 to a specified Wi-Fi network.
```cpp
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
  delay(500);
  Serial.print(".");
}
Serial.println("");
Serial.print(">> Connected to ");
Serial.println(ssid);
Serial.print(">> IP address: ");
Serial.println(WiFi.localIP());
```

### mDNS Setup:
Sets up mDNS for easy access to the ESP32 using a hostname.
```cpp
if (!MDNS.begin(host)) {
  Serial.println(">> Error setting up MDNS responder!");
  while (1) {
    delay(1000);
  }
}
Serial.println(">> mDNS responder started.");
```

### Web Server:
Configures the web server to handle various requests.
```cpp
server.on("/", HTTP_GET, []() {
  server.sendHeader("Connection", "close");
  server.send(200, "text/html", loginIndex);
});
server.on("/serverIndex", HTTP_GET, []() {
  server.sendHeader("Connection", "close");
  server.send(200, "text/html", serverIndex);
});
server.on("/update", HTTP_POST, []() {
  server.sendHeader("Connection", "close");
  server.send(200, "text/plain", (Update.hasError()) ? "FAIL" : "OK");
  ESP.restart();
}, []() {
  HTTPUpload& upload = server.upload();
  if (upload.status == UPLOAD_FILE_START) {
    Serial.printf("Update: %s\n", upload.filename.c_str());
    if (!Update.begin(UPDATE_SIZE_UNKNOWN)) {
      Update.printError(Serial);
    }
  } else if (upload.status == UPLOAD_FILE_WRITE) {
    if (Update.write(upload.buf, upload.currentSize) != upload.currentSize) {
      Update.printError(Serial);
    }
  } else if (upload.status == UPLOAD_FILE_END) {
    if (Update.end(true)) {
      Serial.printf("Update Success: %u\nRebooting...\n", upload.totalSize);
    } else {
      Update.printError(Serial);
    }
  }
});
server.begin();
```

### Handling Client Requests:
Ensures the web server handles incoming client requests.
```cpp
void loop(void) {
  server.handleClient();
  delay(1);
}
```

## 🐞 Troubleshooting

- **Connection Issues:** If the ESP32 fails to connect to the Wi-Fi, ensure that the SSID and password are correct and that the Wi-Fi network is working.
- **OTA Failures:** If OTA updates fail, check the Serial Monitor for error messages and ensure that the ESP32 is connected to the same network as your computer.
- **Web Interface Access:** If you cannot access the web interface, verify the IP address of the ESP32 and ensure your computer is on the same network.

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request for any changes.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📝 Description

This project showcases the process of configuring an ESP32 to accept OTA updates via a web server. By hosting a web interface, the ESP32 allows for easy and convenient firmware updates, making it an ideal solution for remote or difficult-to-access devices.
