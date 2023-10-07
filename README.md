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
