Sysmon is a microsoft tool that is part of the internal suite. Provides a lot of telemetry can monitor events and is customizable using a config file. 

The config file tells sysmon what to log. 

## Steps
1. Download from the official page. and the configuration file attached. sysmonconfig.xml is listed in this repo.  
2. To install sysmon I open a powershell to the same location and fire the installation command using the config file previously given. 
3. Using services it is possible to see that it is successfully installed. From the services handler of windows it is possible to start and restore the service at need. 
