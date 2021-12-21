## **Default Credentials**
Below is the default credentials for the database and the application. The admin credential for the application is automatically generated if it doesn't exist.
#### 1. MongoDB
- **Admin**
   - username: `admin`
   - password: `developer`
- **Regular**
   - username: `smartdispenser`
   - password: `developer`

#### 2. Application
- username: `admin`
- password: `developer`

&nbsp;
## **Installation**
#### 1. Software
Download all files below and install them on your windows machine. You may want to choose `.msi` file as it provide easy installation step.

1. Download [NodeJS](https://nodejs.org/en/download/)
2. Download [MongoDB](https://www.mongodb.com/try/download/community)
3. Download [MongoDB Compass](https://www.mongodb.com/try/download/compass)
4. Download [MongoDB Shell](https://www.mongodb.com/try/download/shell)
5. Download [Zadig](https://zadig.akeo.ie/)

After software installation complete, you may need to add the paths below into your `system path`. To do so, open __Windows Search__ and type `Environment` in the search field. Choose __Edit the system Environment Variables__ from the search result. The _System Properties_ window will appear, click _Environment Variables_ in the bottom right corner of the window. Inside _Environtment Variables_ window, select variable `Path` and click _Edit_. When the _Edit environtment variables_ window appears, click _new_ to add the following path:

1. `C:\Program Files\nodejs\`
2. `C:\Users\{USERNAME}\AppData\Local\Programs\mongosh\`
3. `C:\Program Files\MongoDB\Server\5.0\bin\`

You may need to restart your machine before your terminal recognize the new _system path_. The next step is to install _LIBUSB_ driver using _Zadig_. You need to be careful here as **installing driver into the wrong device may result an unexpected result**. You should make sure that you are selecting the _RFID Reader_ and not another device.
1. Open `Zadig` as an Administrator.
2. Go to `Options`, check `List All Devices`.
3. Select your _RFID Reader_. The device may be listed as `USB Standard Keyboard (Interface 0)`. You need to make it sure by looking into the _USB ID_ (it's the same as _ProductID_), in my case the _USB ID_ was `FFFF 0035 00` (the _ProductID_ is `35`). You should check yours from the _Device Properties_.
4. On the _Driver_ section, select `WinUSB (v6.X.XXXX.XXXXX)`.
5. When you are sure that this is the device, the last step is to _click_ `Replace Driver`.
6. After the driver is successfully installed, yo can't use the device as you regularly did. This is important for the sake of the server to be able to access the device. Otherwise, the server will tell you that the device is not supported by the _Operating System_.

#### 2. Database
To setup MongoDB Security, you need to open _Windows PowerShell_ and run the following command.
1. Start MongoDB Shell.
   - command: `mongosh`
2. Create new database named _smartdispenser_.
   - command: `use smartdispenser`
3. Check if the database was created successfully. You would see _smartdispenser_ in the list.
   - command: `show dbs`
4. Use admin database to create new user.
   - command: `use admin`
5. Create Super Admin user.
   - command: `db.createUser({user: "admin", pwd: "developer", roles: ["root", "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase"]})`
6. Create a user which will be used by the server to connect to the database.
   - command: `db.createUser({user: "smartdispenser", pwd: "developer", roles: [{role: "readWrite", db: "smartdispenser"}]})`
7. Check if users are successfully created. You would see two users in the list.
   - command: `db.system.users.find()`
8. Exit MongoDB Shell.
   - command: `exit`

After the shell command successfully executed, you need to go to `C:\Program Files\MongoDB\Server\5.0\bin` (MongoDB installation directory) and open `mongod.cfg` file using text editor. Add the following configuration command after `#security:` line.
````
security:
  authorization: "enabled";
````
You may need to restart MongoDB Service to enable authorization feature. To do so, open _Windows Task Manager_ by pressing `CTRL + SHIFT + ESC` and select _Services_ tab. Find process with name `MongoDB`, right click it and select _restart_.
You can also restart the service using terminal by running the following commands.
1. Start MongoDB Shell.
   - command: `mongosh`
2. Use admin database.
   - command: `use admin`
3. Shutdown the server.
   - command: `db.shutdownServer()`
4. Exit from MongoDB Shell.
   - command: `exit`
5. Start MongoDB service.
   - command: `net start MongoDB`

#### 3. Server
The insctructions below will guide you to install the NodeJS application as a windows service. The service will run as daemon and will automatically restart it self when the application crashed. The service will automatically started after system boot, so you don't need to start it manually.
1. Copy everything inside `Production` folder into `C:\Smart Dispenser\`.
2. Go to the project directory (using _File Explorer_) and open file `.env` using text editor.
3. Modify the `PORT`, `USB_VENDOR_ID`, and `USB_PRODUCT_ID` value according to your need. The `USB_XXX` is the device descriptor of the _RFID Reader_. You can find the _VendorID_ and _ProductID_ in the device properties.
4. Open _Windows PowerShell_ as Administrator.
5. Go to project directory.
   - command: `cd "C:\Smart Dispenser"`
6. Install all dependencies.
   - command: `npm install --only=prod`
7. Run application for the first time.
   - command: `node index.js`
8. Stop the NodeJS application by pressing `CTRL + C`. If this is not working, you can close _Windows PowerShell_ and open it again.
9. Start the Application Service.
   - command: `net start smartdispenser`.

**Important!**
The `PORT`, `USB_VENDOR_ID`, `USB_PRODUCT_ID` and all of it's kind inside `.env` file are treated as _Environtment Variable_. These variables are created only the _first time when the service created_. Whenever you are modifying these variables, you need to reinstall the service. To do so, follow the steps below.
1. Open _Windows PowerShell_ as Administrator.
2. Stop current service.
   - command: `net stop smartdispenser.exe`
3. Go to project directory.
   - command: `cd "C:\Smart Dispenser"`
4. Uninstall the service.
   - command: `npm run uninstall`
5. Make sure that the `daemon` folder inside the project directory has been deleted. If not, you can run `npm run uninstall` once again or just deleting it manually.
6. After the service has been successfully uninstalled, you can install it again.
   - command: `node index.js`
7. Stop the server by pressing `CTRL + C` or just closing the _Windows PowerShell_ and open a new one.
8. Start the service.
   - command: `net start smartdispenser.exe`

#### 4. **Microcontroller**
The following instructions will guide you to setup the ESP32 microcontroller using Arduino IDE.

##### 4.1. IO Pins
The pins used in the microcontroller are as shown below. You can change it according to your PCB Schema.

1. `PIN_BUTTON_STOP` is used by the _Stop Button_. When the button is pressed, current registered user will be invalidated and the _Relay_ will stop responding to the _Proximity_ sensor.
2. `PIN_LED` is used by the _WS2812B LED_. There is only one LED is used. The meaning of the first LED colors are described as the following:
   - `RED` means the device is offline or cannot connect to the server.
   - `YELLOW` means the device is online (connected to the server) but there is no user is registered (it is on idle or ready state). The _Relay_ will not respond to _Proximity_ state, unless _Temporary Open_ is set.
   - `GREEN` means there is a user registered and the device is ready to go. The _Relay_ will respon to _Proximity_ state, and will override _Temporary Open_ setting. It's mean that if the users use their card, their quota will be used instead of the _Temporary Open_ quota.
   - `BLUE` means the _Relay_ is ON.
3. `PIN_RELAY` is used by the _Relay_.
4. `PIN_BUZZER` is used by the _Buzzer_. The beep sound of the buzzer is vary according to the device state. The meaning of the buzzer beeps are described as the following:
   - `Beep Once` means the user quota has been reached.
   - `Beep Twice` means the user has pressed the _Stop Button_ and it's data has been invalidated.
5. `PIN_WATERFLOW` is used by the _Water Flow Sensor_. The water flow sensor needs to be calibrated. The calibration constant (_flowConstant_) can be set through the web application as an _Admin_ or an _Operator_. The _flowConstant_ is assigned as double with the default value of 10. Please note that the water output will increase as the _flowConstant_ decrease. It means that the _flowConstant_ of 5 will be outputting more water than the _flowConstant_ of 6.
6. `PIN_PROXIMITY` is used by the _Proximity Sensor_.
7. `PIN_ETHER_XXX` is used by the _Ethernet Module_. Please note that you need to change the `MODE` to `TCPClient::ETHERNET` to be able to use _Ethernet_ as the connection carrier.

##### 4.2. IO Setup
Input can be configured as _Pulled Up_ which mean the default state when not pressed is `HIGH`. Output can be configured as _Active Low_ which should be used if the device is _Active Low_ as well. For example, most of relay modules are _Active Low_, so it's important to configure `PIN_RELAY` as _Active LOW_. This setting can be modified through `void setup()` by modifying the `begin(bool)` parameter. `relay.begin(true)` means the `PIN_RELAY` is set to _Active Low_ and `buttonStop.begin(true)` means the `PIN_BUTTON_STOP` is set to be _Pulled Up_. The IO pins are described as the following.
1. `PIN_RELAY` as _Output_.
2. `PIN_BUZZER` as _Output_.
3. `PIN_LED` as _Output_.
4. `PIN_BUTTON_STOP` as _Input_.
5. `PIN_WATERFLOW` as _Input_ (Interrupt).
6. `PIN_PROXIMITY` as _Input_.

##### 4.3. Constant Variables
The variables used by the microcontroller are described as the following. Please note that these variables are constant and cannot be modified at runtime.
1. `MODE` is the connection carrier type. There are two possible mode which is `TCPClient::WIFI` and `TCPClient::ETHERNET`.
2. `MAC_ADDRESS` is the MAC address of the _Ethernet Module_. You can set the address arbitrarily as long as it's not being used by other device in the same network.
3. `IP_XXX` and `DNS_XXX` are used by _WiFi_ and _Ethernet_ module to connect to the network server using Static Configuration.
4. `WIFI_XXX` is the _WiFi_ credentials. You can modify it according to your _ssid_ and _password_. If the WiFi is open, you can set the password as an _empty string_.
5. `URL` is the server address. You can modify it according to your server setting. The valid format is `ws://{hostname}:{port}/device`.
6. `KEY_XXX` is the key of the variable saved in the device database. This value **should not be modified**.

&nbsp;
## Application
#### 1. Server
The server application was build on top of _NodeJS Express_ framework. There is only one server running at a time and is handling all of the incoming request, i.e. _authentication_, _managing database_, and responsible for the communication between Web Application and Microcontroller. The key feature is described as the following.

##### 1.1 Security
The server is secured by OAuth 2.0 credentials. The web application must use `accessToken` in order to be able to communicate with the server (except for public area). You can request `accessToken` using `POST` request by providing `username`, `password`, `clientId`, and `clientSecret` to `\oauth\token`. You can also retrieve `accessToken` by providing `refreshToken` to the same address. The `refresh token` is always sent along with the `accessToken`.

##### 1.2 User
The default user is _admin_ and will be created automatically when the server start. You can add or modify user through Web Application. There is three different roles of user, which is:
1. `Admin` can access all the field including _modifying device setting_, _modifying employee list_, and _modifying users_.
2. `Manager` can modify _employee list_ but can't modify _device setting_ and _user list_.
3. `Operator` can modify _device setting_ and read _employee list_ (only read, modifying employee list is not allowed). _Operator_ can't access _user list_.

##### 1.3 API
The communication between client and server are using _WebSocket_. There is no single _REST API_ defined in this server. All of the transactions are _cached_ on both server and client, and will be destroyed after the client has been _signed out_. There are three different _Web Socket_ servers available, which is:
1. `/app` is used to manage _employee list_, _user list_, and modifying _device setting_.
2. `/data` is a public area, which means any client can access it without providing credentials. This socket is only used by the server to send currently registered employee (i.e. when an employee tap a card) to the `Web Application`.
3. `/device` is used by the _Microcontroller_ to communicate with the server.

Upon connecting to the _Web Socket_ server, the client must provide `ticket` as part of the _url query_ (only if you are trying to connect to `\app`), otherwise the connection will immediately be closed by the server. You can retrieve a ticket by doing `POST` request to `\oauth\ticket`. Please note that the `ticket` can only be used once and only valid for _one minute_. The request must be composed with the following schema.
````
{
    type: "reg" | "action",
    data: {
        command: "add" | "get" | "update" | "delete",
        content: object
    }
}
````

#### 2. Web
There are four pages in the web application, including _Home_, _Login_, _Dashboard_, and _History_. Each of them are described in the following section.

##### 2.1 Home
Home is a public area, any client can access it. The main purpose of the home page are to display current active employee (i.e. any employee which is tapping the card). 

##### 2.2 Login
Login page is used to authorize a user using username and password. Whenever an unauthenticated user trying to access another page (except _Home_), they will be redirected into this page.

##### 2.3 Dashboard
Dashboard page is used for managing _employee list_, _user list_, and _device setting_. The visual and/or functional elements are different between users, depending on their role. An _Admin_ can access all of the available options, whereas a _Manager_ or an _Operator_ can only access part of the available options. The functionality of this page are as described below.
1. `Adding`, `Updating`, and `Deleting` users. This feature is only available for _Admin_. You can access this functionality inside the _Setting_ section of the page (by clicking the _Setting_ icon).
2. `Adding`, `Updating`, and `Deleting` employees. This feature is only available for _Admin_ and _Manager_. You can update an employee by selecting it in the list. By doing so, you can also `Reset` and `Save` the updated employee. When the _Reset_ button is clicked, the employee field will return to it's original value. To `Delete` and employee, you can _Right click_-ing it and then choose _Delete_. In order to add a new employee, you can _click_ the `Add Employee` button. Right after you _click_ it, a _Modal_ will appear, you need to fill all of the necessary field to continue. You may notice that after _Adding_ or _Updating_ an employee, a colored marker showing up in the right corner of the field. The meaning of each marker colors are described as of the following.
   - `RED` means there is error in the field (e.g. there is an empty field). Whenenever this happens, the _Save_ button is grayed out and you can't save the updated value.
   - `ORANGE` means the value has been modified but is not saved yet.
   - `GREEN` means a newly added employee. An employee is considered as new if it was created less than a day ago.
3. `Modifying` _Device Setting_. This feature is only available for _Admin_ and _Operator_. You can access it by _click_-ing the setting icon. All of the available options for _Device Setting_ are described below.
   - `Status` tells whether the device is currently connected to the server. If so, the indicator will show _online_, otherwise it will show _offline_.
   - `Pump` tells whether the _Relay_ is currently _active_. If so, the indicator will show _ON_, otherwise it will show _OFF_.
   - `Flow Constant` is the calibration constant of the water flow sensor.
   - `Auto Invalidation Timeout` is used to determine how long an employee (i.e. who is tapping their card) is held active. You can still press the _Stop Button_ to immediately invalidate the currently active employee. Please note that when an employee triggering (approaching) the _Proximity_ sensor, the timeout will be cancelled, and will be set again after the employee is away from the sensor (or when it's quota has been reached).
   - `Temporary Open` tells the microcontroller to temporarily open it's functionality for the given quota. It means that anyone can get a water without tapping their card. Please note that if an employee tapping their card, their quota will be used instead of the _Temporary Open_ quota. This feature is very useful to be used as calibration tools. For example, you can set the _Temporary Open_ value to 100 ml, then you can take water using a measuring cup. If the water that comes out is less than 100 ml, then you need to reduce the `Flow Constant`. Otherwise, you need to increase the `Flow Constant` until the water that comes out is around 100 ml.

##### 2.4. History
History page gives you information about how much water is used by the employee each day. You can _save_ it as `.json` or `.xlsx` format. If you want to go to the _Dashboard_ page, you can _click_ the dashboard icon. You can always go back to the _Home_ page by _click_-ing the application icon on the top-left corner.
