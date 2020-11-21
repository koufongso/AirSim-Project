# AirSim-Project

## Tested Platform: Ubuntu 16.04 LTS (It should also work on Ubuntu 18.04 LTS)
## Required Software
* Unreal Engine 4.24
* AirSim
* Ardupilot

<br>

## Installation
### Unreal Engine 4.24 
1. Register a Unreal account and link the account with your github account.
2. Use the following command to clone the Unreal engine repository
        ```
        git clone -b 4.24 https://github.com/EpicGames/UnrealEngine.git
        cd UnrealEngine 
        ```
    * <b>Make sure you are in the correct branch (4.24) before continue.</b> You can use `git branch` or `git statue` to chekc the cuurent branch, and use `git checkout 4.24` to switch to branch 4.24 if you are not in the right one.

3. Then use the following commands to build the Unreal engine
        ```
        ./Setup.sh
        ./GenerateProjectFiles.sh
        make
        ```
    * The installation require more than 90GB disk space
    * The process is time consuming, and depending on network condistion and the computer specs, the installation can take up to 4~5 hours
    <p>For more information please check: <a>https://docs.unrealengine.com/en-US/Platforms/Linux/BeginnerLinuxDeveloper/SettingUpAnUnrealWorkflow/index.htmldownload</a></p>

### AirSim
* Clone and build AirSim with the following commands:
    
    ```
    git clone https://github.com/Microsoft/AirSim.git
    cd AirSim
    ./setup.sh
    ./build.sh
    ```  
    <p>For more information, please check: <a>https://microsoft.github.io/AirSim/build_linux/</a></p>

### ArduPilot
1. Clone and build the required packages for Ardupilot

    ```
    git clone https://github.com/ArduPilot/ardupilot.git
    cd ardupilot
    git submodule update --init --recursive
    Tools/environment_install/install-prereqs-ubuntu.sh -y
    . ~/.profile
    ```
2. Then build the software-in-the-loop (SITl)
    
    ```
    ./waf configure --board sitl
    ./waf copter
    ```

<br>

## Customize Unreal Environment
<p>This section is to enable the AirSim plugin in your customized unreal envoriment. Make sure <b>Unreal Engine 4.24</b> and <b>AirSim</b> are installed before you continue. For installation detail, please refer to the <b>Installation</b> section.</p>

1. Create an empty Unreal project (or any existing projects, make sure they are compiled in 4.24 version)
2. Open `File` on the tool bar, select `New C++ Class` to create an empty class.
3. Open the `project_name.uproject` file in the project root directory with text editor.
4. Add the following text in the `"AdditionalDependencies"` under `"Modules"` so it looks like:
    ```
    "Modules": [
		{
			"Name": "<project_name>",
			"Type": "Runtime",
			"LoadingPhase": "Default",
			"AdditionalDependencies": [
                "AirSim"
			]
		}
	]
    ```
5. Add the following text in the `"Plugin"` field:
    ```
    "Plugins": [
        ...,
        {
            "Name":"AirSim",
            "Enabled":true
        }
    ]
    ```
6. Copy the `Plugin` folder in `./path_to_AirSim/AirSim/Unreal/` to the project root directory.
7. Launch the project, either from `.uproject` or from `UE4Editor`.
8. Select `Window`->`World Settings`. And on the right windows, you should be able to see section `Game Mode`. On the `GameMode Override` drop-down window, choose `AirSimGameMode`.
9. Now you can click `Play` button on the top to start the simulation.

## Run SITL using Ardupilot and AirSim

### Multidrone System
1. In Unreal Editor, go to `Edit->Editor Preferences` in the tool bar, search using the keyword `CPU` and ensure that the `Use Less CPU when in Background` is <b>unchecked</b>.
2. Modified the AirSim setting file. It should locate in `./document/AirSim` directory
```
{
  "SettingsVersion": 1.2,
  "SimMode": "Multirotor",
  "OriginGeopoint": {
    "Latitude": -35.363261,
    "Longitude": 149.165230,
    "Altitude": 583
  },
  "Vehicles": {
    "Copter1": {
      "VehicleType": "ArduCopter",
      "UseSerial": false,
      "LocalHostIp": "127.0.0.1",
      "UdpIp": "127.0.0.1",
      "UdpPort": 9003,
      "ControlPort": 9002
    },
    "Copter2": {
      "VehicleType": "ArduCopter",
      "UseSerial": false,
      "LocalHostIp": "127.0.0.1",
      "UdpIp": "127.0.0.1",
      "UdpPort": 9013,
      "ControlPort": 9012,
      "X": 0, "Y": 3, "Z": 0
    }
  }
}
```
3. Open a terminal, go to the ardupilot root directory and use the command (IP is optional and it is 127.0.0.1 by default):
    
    ```
    libraries/SITL/examples/Airsim/follow-copter.sh <IP>
    ```
4. Open an new terminal `(control terminal)`, go to the ardupilot root directory and use the command:
    
    ```
    mavproxy.py --master=127.0.0.1:14550 --source-system 1 --console --map
    ```
    This will open a console for all the drone command information, and a map indicates the drones location.
5. Start the Simulation by clicking the `Play` button in the unreal editor.
6. Below are some simple commands to control the drones. In the `control terminal`:  
* Use command `vehicle <id>` to select the drone you want to control.
* Use command `arm throttle` to arm the selected drone.
* Use command `mode guided` to switch to guided mode and then use command `takeoff <height[m]>` to take off the selected drone.
* You can right click the map and select `Flyto` to fly the drone to the designated location.
