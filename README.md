# Project_Electronics_WiFi-ESP32-LED-Control
One of the most interesting project we can build while learning to work with a WiFi Based board is a web Server. As we continue the journey to learn how to build projects with the ESP32, we will examine how to build a simple web Server with the ESP32.

![Screenshot 2021-08-01 044614](https://user-images.githubusercontent.com/56201060/127756553-cc2bc5c1-26cf-4482-a11d-4efe926df609.jpg)

![Screenshot 2021-08-01 051302](https://user-images.githubusercontent.com/56201060/127756915-52f3a0c2-5282-4f2e-8e0d-13803dbaea2a.jpg)


 Web server is essentially a device which stores, processes and delivers web pages to web clients, which can range from browsers on our laptops to Mobile apps on our smartphones. The communication between client and server takes place using a special protocol called Hypertext Transfer Protocol (HTTP). The client uses an URL to make a request for a particular page and the server responds with the content of that web page or an error message if the page is not available.In our case, rather than responding with a specific webpage, the URL will be used to control the state of LEDs connected to the ESP and the changes made will be updated on the webpage. For instance, if a URL like “http://192.168.1.9”  is entered in the browser, the web server will understand its time to turn the LED “ON” and then update the status of the LED to “ON” on the webpage. Same thing will happen when it’s required to go “OFF”.

# REQUIRED COMPONENTS:

The following components are required for this project:

DOIT ESP32 DEVKIT V1

220 ohms Resistor (2)

LED (2)

Breadboard

Jumper Wire

# SCHEMATICS:

 Connect the components as shown below.
 
 ![Screenshot 2021-08-01 040450](https://user-images.githubusercontent.com/56201060/127755931-86b39c9a-700f-43f0-8a30-011092492b0f.jpg)
 
 
The positive legs of the green and red LEDs are connected t0 GPIO pins 26 and 27 on the ESP32 (respectively),while their negative legs are connected to ground via a 220 ohms resistor to limit the amount of current flowing through the LEDs.With the schematics done, we can now move to the code for the project.

# CODE:

To reduce the amount of code we need to write, we will use the ESP32 WiFi Library. The library contains functions that make setting up the ESP32 as a web server easy. The library, along with several others, is packaged together with the ESP32 board files and are automatically installed when the ESP32 board files are installed on the Arduino IDE.

            #include <WiFi.h>

Next, add the credentials of the WiFi access point to which the ESP32 will be connected. Ensure the username and password are in-between the double quotes. We also specify the port through which the system will communicate and create a variable to hold requests.

              // Load Wi-Fi library
              // Replace with your network credentials
              const char* ssid = "smartmethods"; 
              const char* password = "123456789"; 

              // Set web server port number to 80
              WiFiServer server(80);  //The server responds to clients (web browsers) on port 80 (standard port)

              // Variable to store the HTTP request
              String header;

Next, we declare the pins of the ESP32 to which the red and green LED are connected and create variables to hold the state of the LEDs.

              // Auxiliar variables to store the current output state
              String output26State = "off";
              String output27State = "off";

              // Assign output variables to GPIO pins
              const int output26 = 26;
              const int output27 = 27;

With this done, we move to the void setup() function.We start by initializing the serial monitor (it will be used for debugging later) and set the pinModes of the pins to which the LEDs are connected as outputs. We then set the pins “LOW” to ensure the system starts at a neutral state.

             void setup() {
              Serial.begin(115200);
                // Initialize the output variables as outputs
              pinMode(output26, OUTPUT); //--> LED port Direction output
              pinMode(output27, OUTPUT);
                // Set outputs to LOW
              digitalWrite(output26, LOW); // Turn off Led  
              digitalWrite(output27, LOW); 
  
  Next, we connect to the access point using the credentials as arguments to the WiFi.begin() function and the WiFi.status() function to check if connection was successful.
  
               // Connect to Wi-Fi network with SSID and password
              Serial.print("Connecting to ");
              Serial.println(ssid);
              WiFi.begin(ssid, password);
              while (WiFi.status() != WL_CONNECTED) {
              delay(500);
              Serial.print(".");
              }

If the connection is successful, a text is printed on the serial monitor to indicate that, and the IP address of the web server is also displayed. This IP address becomes the web address for the server and it is what we need to enter on any web browser on the same network to access the Server.

              // Print local IP address and start web server
              Serial.println("");
              Serial.println("WiFi connected.");
              Serial.println("IP address: "); 
              Serial.println(WiFi.localIP());  // this will display the Ip address of the Pi which should be entered into your browser 
  
With that done, we start the server using the server.begin() function and proceed to the void loop( ) function.  

            server.begin();

The void loop() function is where the majority of the work is done. We start by using the server.available() function to listen for incoming connection by clients. When a client is available and connected, we read the client request and send a header as a response.

             void loop(){
              WiFiClient client = server.available();   // Listen for incoming clients
              if (client) {         // If a new client connects,
              Serial.println("New Client.");  // print a message out in the serial port
              String currentLine = "";       // make a String to hold incoming data from the client
              while (client.connected()) {     // loop while the client's connected
              if (client.available()) {        // if there's bytes to read from the client,
              char c = client.read();          // read a byte, then
              Serial.write(c);             // print it out the serial monitor
              header += c;
              if (c == '\n') {
     // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:     
              if (currentLine.length() == 0) {
     // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
              client.println("HTTP/1.1 200 OK");
              client.println("Content-type:text/html");
              client.println("Connection: close");
              client.println();
  
              // turns the GPIOs on and off
              if (header.indexOf("GET /26/on") >= 0) {
              Serial.println("GPIO 26 on");
              output26State = "on";
              digitalWrite(output26, HIGH);
              } else if (header.indexOf("GET /26/off") >= 0) {
              Serial.println("GPIO 26 off");
              output26State = "off";
              digitalWrite(output26, LOW);
              } else if (header.indexOf("GET /27/on") >= 0) {
              Serial.println("GPIO 27 on");
              output27State = "on";
              digitalWrite(output27, HIGH);
              } else if (header.indexOf("GET /27/off") >= 0) {
              Serial.println("GPIO 27 off");
              output27State = "off";
              digitalWrite(output27, LOW);
              }
 
 Next, we create the webpage that will be displayed and updated by the NodeMCU as the user interacts with it. The key function for this is the Client.println() function which is used to send HTML scripts line by line to the client (browser).We start by using the “doctype” to indicate that the next few texts to be printed are HTML lines.
 
             client.println("<!DOCTYPE html><html>");
  
  Next, we add the lines below to make webpage responsive irrespective of the browser being used.
  
              client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
  
  We also throw in some bits of CSS to the client to make the page user-friendly. You can edit this to add your own color, font style, etc.
  
              // CSS to style the on/off buttons 
            // Feel free to change the background-color and font-size attributes to fit your preferences
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");
  
  Next, the webpage header is sent alongside the buttons which are set to display ON or OFF  based on the current state of the LEDs. It will display OFF if the current state is ON and vice versa.
 
  
               // Web Page Heading
            client.println("<body><h1>ESP32 Web Server</h1>");
            
            // Display current state, and ON/OFF buttons for GPIO 26  
            client.println("<p>GPIO 26 - State " + output26State + "</p>");
            // If the output26State is off, it displays the ON button       
            if (output26State=="off") {
              client.println("<p><a href=\"/26/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/26/off\"><button class=\"button button2\">OFF</button></a></p>");
            } 
               
            // Display current state, and ON/OFF buttons for GPIO 27  
            client.println("<p>GPIO 27 - State " + output27State + "</p>");
            // If the output27State is off, it displays the ON button       
            if (output27State=="off") {
              client.println("<p><a href=\"/27/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/27/off\"><button class=\"button button2\">OFF</button></a></p>");
            }
            client.println("</body></html>");
            
            
           Next, we close the connection and the loop goes over again.
           
                // Clear the header variable
              header = "";
               // Close the connection
              client.stop();
              Serial.println("Client disconnected.");
              Serial.println("");
              }
              }
  
  
  
  The complete code for the project is available below and under the download section of this tutorial.
  
                
            /* WiFi-ESP32-LED-Control.
              By Dejan Eng.Odai Farhat.
              Time:1-Aug-2021 
              Note:To enter the control panel 192.168.1.9 */

              // Load Wi-Fi library
  
              #include <WiFi.h>

              // Load Wi-Fi library
              // Replace with your network credentials
              const char* ssid = "smartmethods"; 
              const char* password = "123456789"; 

              // Set web server port number to 80
              WiFiServer server(80);  //The server responds to clients (web browsers) on port 80 (standard port)

              // Variable to store the HTTP request
              String header;
  
              // Auxiliar variables to store the current output state
              String output26State = "off";
              String output27State = "off";

              // Assign output variables to GPIO pins
              const int output26 = 26;
              const int output27 = 27;
  
              void setup() {
              Serial.begin(115200);
    // Initialize the output variables as outputs
               pinMode(output26, OUTPUT); //--> LED port Direction output
               pinMode(output27, OUTPUT);
   
    // Set outputs to LOW
              digitalWrite(output26, LOW); //--> Turn off Led  
              digitalWrite(output27, LOW); //--> Turn off Led 
   
    // Connect to Wi-Fi network with SSID and password
              Serial.print("Connecting to ");
              Serial.println(ssid);
              WiFi.begin(ssid, password);
              while (WiFi.status() != WL_CONNECTED) {
              delay(500);
              Serial.print(".");
              }
                // Print local IP address and start web server
              Serial.println("");
              Serial.println("WiFi connected.");
              Serial.println("IP address: "); 
              Serial.println(WiFi.localIP());// this will display the Ip address of the Pi which should be entered into your browser 
              server.begin();
              }
  
              void loop(){
              WiFiClient client = server.available();   // Listen for incoming clients
              if (client) {         // If a new client connects,
              Serial.println("New Client.");  // print a message out in the serial port
              String currentLine = "";       // make a String to hold incoming data from the client
              while (client.connected()) {     // loop while the client's connected
              if (client.available()) {        // if there's bytes to read from the client,
              char c = client.read();          // read a byte, then
              Serial.write(c);             // print it out the serial monitor
              header += c;
              if (c == '\n') {
                 // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:     
              if (currentLine.length() == 0) {
                 // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
              client.println("HTTP/1.1 200 OK");
              client.println("Content-type:text/html");
              client.println("Connection: close");
              client.println();
  
              // turns the GPIOs on and off
              if (header.indexOf("GET /26/on") >= 0) {
              Serial.println("GPIO 26 on");
              output26State = "on";
              digitalWrite(output26, HIGH);
              } else if (header.indexOf("GET /26/off") >= 0) {
              Serial.println("GPIO 26 off");
              output26State = "off";
              digitalWrite(output26, LOW);
              } else if (header.indexOf("GET /27/on") >= 0) {
              Serial.println("GPIO 27 on");
              output27State = "on";
              digitalWrite(output27, HIGH);
              } else if (header.indexOf("GET /27/off") >= 0) {
              Serial.println("GPIO 27 off");
              output27State = "off";
              digitalWrite(output27, LOW);
              }
            // Display the HTML web page
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<link rel=\"icon\" href=\"data:,\">");
            // CSS to style the on/off buttons 
            // Feel free to change the background-color and font-size attributes to fit your preferences
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");
            
            // Web Page Heading
            client.println("<body><h1>ESP32 Web Server</h1>");
            
            // Display current state, and ON/OFF buttons for GPIO 26  
            client.println("<p>GPIO 26 - State " + output26State + "</p>");
            // If the output26State is off, it displays the ON button       
            if (output26State=="off") {
              client.println("<p><a href=\"/26/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/26/off\"><button class=\"button button2\">OFF</button></a></p>");
            } 
               
            // Display current state, and ON/OFF buttons for GPIO 27  
            client.println("<p>GPIO 27 - State " + output27State + "</p>");
            // If the output27State is off, it displays the ON button       
            if (output27State=="off") {
              client.println("<p><a href=\"/27/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/27/off\"><button class=\"button button2\">OFF</button></a></p>");
            }
            client.println("</body></html>");
  
               // The HTTP response ends with another blank line
            client.println();

               // Break out of the while loop
              break;
              } else { // if you got a newline, then clear currentLine
                currentLine = "";  }
              } else if (c != '\r') {   // if you got anything else but a carriage return character,
                currentLine += c;  // add it to the end of the currentLine
    
                }
              }
  
              }
                 // Clear the header variable
              header = "";
               // Close the connection
              client.stop();
              Serial.println("Client disconnected.");
              Serial.println("");
              }
              }
  
                                                /* END */

  
  
            

 
  
