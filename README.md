# EdgeToAppExam
My solution in exam in edge(esp-32) to app(react native)
This repo is a collection off the reapos i created in my exam EdgeToApp
##### Table of Contents 
1. [Question](#question)
2. [Planning](#planning)
3. [Solution](#solution)
4. [Setup](#setup)
5. [repo Links](#links)



# Question <a name="question"></a>

Greenhouse system

The client was pleased with your Greenhouse prototype (Obligatory Submission 1), but they want a custom solution that is not limited by the cloud platform you used. Use your experience from the prototype to build a custom solution for the customer. They’ll expand the solution later, but for now they only need a Dashboard with a graph of the sensor data. The client has just started building greenhouses in 3 locations and want to use your new system to monitor all of them.

The requirements are:

- You must register and store Temperature, Humidity and Light from your ESP32 Edge device
- The client wants the solution to run on their own servers, so you must make a Node API to store the sensor data
- They want a Dashboard written in React that shows a graph of the last 24 hours of sensor data
- Deliver a video showing the final solution, all required code and written documentation. The reviewer should be able to replicate your solution.

You can build the Dashboard as either a web App or a mobile App written in Expo. If you are able to come up reasons to use other sensors in the Edge device, please do so. If you think the Dashboard needs more than just a graph, you are free to add features to the solution. Draw on your experience from the obligatory submissions and lectures to get ideas for good additions

# The planning <a name="planning"></a>


# The solution <a name="solution"></a>

### Implementation

This will have 3 projects, a server, client project(greenleaf) inside solution folder and edge project.

you can run it local or connect the client and edge device to my live solution on azure, for details on setup go to Setup and look at local, remote or private remote setup.

### MQTT and WS MQTT  Endpoints

you will find all topic for mqtt under the server routes folder in the wsmqtt.js file

device/devices are referencing  esp32 devices and not the client app.

 

- devices // can be used by any device even when not logged in
- devices/<deviceName>/profile to update device profile like location and password or FW to use and if auto update is on
- update : server will send an array whit latest FW for all builds, now its “dev” and “prod” this is for any device that are set to auto update
- locations/<location>/alarm : will send warning or alarm values on change it will send if alarm is on/off, type and alarm text.
- locations/<location>/live : used to send live humidity, temperature and calculated lux level(server will only store every 10 min)
- locations/<location>/light : will only be sent once pr day, it will send hours of sunlight and hours of lamp light per day
- locations/update: devices sends a location and gets back a city for that location, this is for when the client app updates the location of a device it needs to find what city the location is in so the weather API can get the correct temperatures

### HTTP Endpoint

Are bit many to name them all but the main ones are

- locations
- Devices
- Login

Each endpoint has its own file in the server project under routes named after the  main endpoints. 

### Server Project

Our server, built on Node.js and Express, incorporates several additional libraries and services used beyond the standard ones. These include:

- nodemailer
- aedes
- bcrypt (for generating hashed and salted user passwords)
- crypto (to encrypt cookies, thereby preventing plain text signing)
- expo-server-sdk (for push notifications)
- mongoose
- websocket-stream and ws (for running aedes over websocket)
- this project runs both local and on Azure cloud whit mongodb cluster

The server operates three services: MQTT over TCP, MQTT over WebSocket, and HTTP. MQTT caters to the edge device, whereas MQTT over WebSocket serves the client project as I haven't identified a reliable MQTT library for react native that supports MQTT over TCP.

Both MQTT servers have been configured to use the same event listeners. This design allows the client and edge device to communicate effectively, regardless of whether they operate on different ports or protocols. When an alarm event occurs or the edge device stops transmitting data (triggering a keepalive event after 30 seconds), the server sends push notifications and emails to the client app/email. Push notification only works on a real phone so test it you need expo go on your phone.

We use signed and encrypted cookies for persistent sign-ins, once a username and password have been provided. Passwords are hashed with unique salts, ensuring identical passwords do not result in the same hash. This is facilitated through the bcrypt and crypto libraries.

Certain routes require login, and some even necessitate admin privileges. MQTT also employs username and password authentication, controlling who can publish and subscribe to various topics.

Every 10 seconds, the server scans the firmware (FW) folder for files in the format `fw/dev/fw-dev-vX.bin` or `fw/prod/fw-prod-vX.bin` (where X is a version number). It then identifies the latest version for each build and publishes it on the "update" topic, allowing devices to identify the newest version.

I use dev for local testing and then prod for when the device should connect to the azure VM running.(still need to setup the edge device from the hotspot, see Edge device for details)

Push notifications can fail at two points, and if not properly handled, your app's push notification functionality may be disabled. To counter this, we remove a token associated with a user if it fails during execution. We also record the ticket ID and token for that ticket, checking 10 minutes later (or up to 30 minutes for production) to verify if the push notification was successful.

Considering each user may have multiple devices, we store tokens in an array within the userModel. Simultaneously, we store tickets and tokens for verification in the validTokenModel.

### Edge Device Project

The device will transmit the follow data:

- lux, temp and humidity will be transmitted live
- sun light data for each day will be sent once each day.
- there will be total of 17 alarms/warnings and status messages sent
- they will only be published on value changed.
- door alarm will also need too be open for a set amount of time before it get sent as an alarm

The edge device will transmit live data pertaining to light (lux), temperature, and humidity. It will issue alarms or warnings for high or low levels of these measurements, with specific thresholds detailed in the Monitoring section.

The device calculates lux based on the input from both light sensors. It will also interface with the Weather API to fetch external temperature data every 10 minutes.

The device uses an accelerometer to check the window's state, determining whether it's open or closed. It calculates the angle based on gravitational force, with an angle above 3 degrees indicating the window is open. If the window is open and the outside temperature is below 24 degrees, it sends an alarm. Similarly, if the temperature exceeds 24 degrees with clear skies and the window is closed, it sends an alarm. The device also triggers an alarm if the door remains open for an extended period.

The device broadcasts data over MQTT and uses HTTP for firmware downloads. If the device is set to auto-update, it will automatically update its firmware as soon as a new version is uploaded to the server. More information on this can be found in the Server Project section.

The edge device will fist start a hotspot called “ESP-WIFI-MANAGER “ on ip 192.168.4.1 where you setup your WIFI details and give the device a static IP for your network, then add inn the IP for where the Server is running.
 MQTT and http server should be the same IP.

The device will then connect too the given WIFI network and connect to the server, it will also start a new server on the static IP you gave where you can restart the hotspot if you need too change the settings.

If the device loses connection to the MQTT server over long period of time it will disconnect from the WIFI and restart the hotspot. 
When ever a device connects or reconnects due to a restart or a new device is connecting. the device will run thru all alarms and switch them on/off to test that all alarms are working and that the correct values get set. This will mean email and push notification will be triggered for each Alarm.

### Client Application Project

This project is built using React Native.

Key libraries utilized, in addition to standard React/React Native Expo libraries, include:

- react-native-picker/picker
- react-native-paho-mqtt
- expo-notifications
- react-native-chart-kit

Primary application screens include:

- Login: User authentication.
- Devices: Manage device settings, such as location or firmware.
- Locations: Create, modify, or delete locations.
- Main: Access main application features.
- Create User: Register new device credentials or create a new user (with potential for admin access after DB edit).
- Main Dashboard: Monitor statistics for a chosen location.
- Lux/Temp/Humidity Dashboard: View 1h, 6h, and 24h graphs for each dataset.
- Sun Dashboard: Display the past 3 days of sunlight hours versus lamp light for plants.
- Logs: View history and current status (ON/OFF) of all alarms and warnings that have been activated at least once.
- Error: Redirect to this page in case of errors like missing or incorrect cookies, or wrong password/username.

The application communicates with the server using HTTP and employs MQTT for receiving live data and alarm statuses from the edge device at a particular location.

When loaded onto a physical device via Expo Go, the application can also receive push notifications. If an alarm or warning occurs and the user clicks on the push notification, they will be directed to the dashboard of the location where the alarm was triggered to check the logs. Notifications function even when the app isn't actively being used.

# How to setup <a name="setup"></a>
## Perquisites

- Node and Npm
- MongoDB installed if using Local Setup
- Mail account if testing email alerts
- Edit Arduino_secrets.h in edge project
- Edit .env in server project and  make sure you change or comment out DB_URL if you are using [localhost](http://localhost) for db
- if you want too use your own username and password for client app and device you need to edit the start of server.js, we inject a admin and user at the start there if none are found.

## Local Setup

The device will take some time under startup, it needs to wait for light sensors to be all the way active then it will calibrate all sensors.

When you connect a device or restart a device the setup will turn on and off all the alarms so you will get a total off 11 emails and push notification if you have them active while doing so. you might get the window open alarm on/off all the time it means your device is just at the threshold for open window(3degrees) so just turn it a bit and it will stop. 

### Server

you need to go in to .env and change anything that has too do whit mail and DB to match your email and DB connection. also need to make sure the PORT is set too 3000

commands to run

```csharp
npm i  
```

to start the server whit hotreaload

```java
npm run dev
```

or run 

```csharp
node server.js
```

The server will create 2 user, one to use when logging in to the client(both will work)

and one to set on the edge device once all is up and running.

the Admin account is test/test for username/password

It will also add dummy data for the past 24h for live data and 1 week for the sunlight data.

### Edge

If you plan to connect the device later to the Azure server running you will need to select a unique name, but it cant be too long. reason I set the name is so i can easily see what device is doing what

You will need to build and upload the FW, then run “build filesystem image” and “upload filesystem image”  restart the device and and then connect to  ESP-WIFI-MANAGER hot spot and connect  to 192.168.4.1.
input your WIFI info and what IP you want the device to have(for resetting WIFIMANANGER)
Then add the Ip that you run the server on for http and MQTT server IP.
Next add the ports, 3000 for HTTP and 4000 for MQTT.

### Client/Greenleaf

You need expo Go for android or iphone to get full use of the app. or you can setup an android/iphone emulator.

You need to change some files

in appcontex.js you need too change the IP to your node server IP  and port to 3000
in app.js you need to change IP  to your node server IP, port is the same for both local and cloud.

commands to start:

```csharp
npm install
```

then

```csharp
npx expo install // not needed but i run it since it gives feedback on things 
```

then start it using

```csharp
npx expo start
```

now press a for android or scan QR code for your phone.

this mobile app has been design on an android pixel 3a phone running on a  emulator so If things look bad or dont work on your phone I would recommend setting up Android emulator by follow this guide. have tested it on a iphone 7 but dose not look as good there, but all thing work.

```csharp
https://docs.expo.dev/workflow/android-studio-emulator/
```

then start the client project again.

 

Users created by the server on startup if missing.

```csharp
admin:

//userName:”test”

//password:”test” 

//user:

//userName:”edge-01”

//password:”edge”
```

### Final setup

once your client, server and edge device is setup and connected to the server you can go in too you client app and log in as test/test then go too locations and add a location. 
Next go back to main menu and select devices, then the device you have connected and add the “edge-01” as username  and “edge” as password and select a location for the device. once you click save it will start sending data to that location and you can go locations again and then select dashboard for the location you send to too and see the live values.

You can create new locations and move the device or connect new devices

Once you are done testing locally you can move the device over to production, and start broadcasting to the live Azure server.  continue to “Remote setup” for setting this up. 

## Remote Setup

If you want to connect your device and app to a live environment you can use this option.

### Edge

You will need to go in too arduino_secrets.h and change NAME to a unique name other then “edge-01” 

if you all ready have local setup running for server, client and edge you can use the client to change from dev to prod build on the edge device and then select version 1. Then the edge device should be sending too remote server instead.

If you don't have  local  setup you will need to build and upload the fw, then run “build filesystem image” and “upload filesystem image”  restart the device and then connect to  ESP-WIFI-MANAGER hot spot and connect  to 192.168.4.1.
input your wifi info and add  `20.253.246.222`  as ip for mqtt/http server http port 80 and mqtt port 4000.

### Client/Greenleaf

You need expo Go for android or iphone.

go to https://expo.dev/@freebattie/greenleaf and go to advance, select “ESA update”, “Expo Pro”, set channel to “master” and sdk to “48.0.0” or go here :
https://expo.dev/%40freebattie/greenleaf?serviceType=eas&distribution=expo-go&scheme=exp%2Bgreenleaf&channel=master&sdkVersion=48.0.0
on IOS use cam, on android use expo app  and scan QR code.

Or you can build it from source code, then you need to make sure the correct port and IP is setup.

in appcontex.js you need too change the IP to`20.253.246.222`  and port to 80
in app.js you need to change IP  to `20.253.246.222` 

### Server

No need to do anything since you are connecting to the azure remote server.

## Remote Setup on your own  Azure VM

### VSC Packages

- Azure Tools
- Remote -SSH

### Setup

easiest way is to install azure tools in VSC and then from the server project create a VM (ubuntu)

if you installed azure tools you will have a tab for Azure form there you right click “Virtual Machine”

and select Create. 

Follow the steps, you might need to try different regions if you are using free tier

you need to then connect too the azure VM over SSH (should have a exaction icon for it)

right click the VM and select connect and enter password you made when setting up the VM

right now NODE and NPM have breaking issues and the official package is to old and on some ubuntu version so you need to download it from a *PPA* (personal package archive), and when I tested you needed to use node 16.x and not 18.x

first you need to install PPA

```bash
cd ~
curl -sL https://deb.nodesource.com/setup_16.x -o /tmp/nodesource_setup.sh
```

you can inspect the script if you like using nano or other editors like vsc

```bash
nano /tmp/nodesource_setup.sh
code /tmp/nodesource_setup.sh
```

next you need to install Node and NPM (used apt-get instead since apt didn't work for me)

```bash
sudo apt-get install nodejs npm
```

check that you got the correct version

```bash
node -v

v16.20
```

then you need to upload the code to a folder in your vm, if you are using VCS you can just drag the server folder over and it will upload it, I set it up whit git and Github where i created a SSH RSA public  and private key, then added the public key to Github and used SSH download  link from Github to clone the repo since my repo is private. and i followed this example

```powershell
https://linuxhint.com/clone-repo-with-ssh-key-in-git/
```

once you got the code uploaded you need to add your mongodb cloud connection string too the server.js file.

next you need to connect too Azure and find you VM and go to networking and open inbound and outbound ports for port 3000,4000 and 4001 if you don't change them.

also write down public IP of your VM, you will need to change your Client and Edge project files to use this IP. again you need to uncomment the code that creates the first user in server.js

### Run

to start the server go in too you VSC SSH  connection, from terminal go to the server folder and run

```bash
npm i

```

then start the server

```bash
npm run dev
```

then stop the server using 

```powershell
ctrl C
```

then install mp2 for running the server

```powershell
npm install pm2 -g
```

start the server whit watch so it will restart on file changes, —log will make console.log() in app write  to a file up one level in logs folder, this to stop watch form restarting when file gets written too:

```powershell
pm2 start server.js --name greenleaf  --log ../logs/file.log --merge-logs --watch
```

save changes

```powershell
pm2 save
```

you can now try and use MQTT explorer app to connect too the server and see if it works.

once you are happy with the setup and don't need too change the server app files any more you can turn off watch and or logging

```csharp
pm2 stop greenleaf
pm2 delete greenleaf
pm2 save
pm2 pm2 start server.js --name greenleaf  --log ../logs/file.log --merge-logs 
//or
pm2 start server.js --name greenleaf 
//then
pm2 save
```

if you want to run server on  port 80 your app needs higher privileges, to avoid doing that i redirect from incoming on port 80 to 8080 on the server so server runs on 8080 

```csharp
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080 
```

or you will need to give SUDO privileges when running server.
## repo Links <a name="links"></a>
