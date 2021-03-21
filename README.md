# Splunk & Sysmon installation and configuration

This is a tutorial in which it will be shown how to collect Sysmon logs in Splunk in order to perform a cybersecurity lab oriented to Purple Team.
In this tutorial we are going to use two virtual machines: A Debian 9 which will be the Splunk server, and a client host which will be a Windows 10 with Sysmon. 

This are the versions of the software used:

  - Splunk Enterprise 8.1.2
  - Splunk UniversalForwarder 8.1.2
  - Sysmon v13.01

## Splunk Installation

First, in our **Debian 9** download the latest version of Splunk Enterprise (which can also be done via command line)

  - https://www.splunk.com/en_us/download/splunk-enterprise/thank-you-enterprise.html

We can also do it by executing the following wget:

```
wget -O splunk-8.1.2-545206cc9f70-Linux-x86_64.tgz 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=8.1.2&product=splunk&filename=splunk-8.1.2-545206cc9f70-Linux-x86_64.tgz&wget=true'
```

Unzip the downloaded file in the /opt/ directory.

```
tar -xvxf splunk-8.1.2-545206cc9f70-Linux-x86_64.tgz -C /opt/
```


Go to the directory where Splunk has been unzipped the binaries

```
cd /opt/splunk/bin/
```

Then start Splunk for the first time and you will have to accept the license. Also, you will be prompted to enter a Splunk user and password. This user will be for logging into Splunk Enterprise, it will not generate a local user. 

```
./splunk start accept--license
```

Now we will start the Splunk service again to start the server. The ports and services that Splunk will use by default will appear, as well as the indexers and other configurations.

```
./splunk start
./splunk status
```

We can also check the ports and services with the Splunk binary

```
./splunk show web-port
./splunk show splunkd-port
```

Access port 8000 on your localhost and you will be able to log in to Splunk with the user that we created in the installation process.


```
https://127.0.0.1:8000/en-GB/account/login
```

## Splunk Configuration

In order to ingest Sysmon logs in Splunk we need to configure a listening port to which the Universal Forwarders can send data.
In Splunk Enterprise go to:

```
Settings > Forwarding and receiving
```

In the Recieve data section choose the Configure receiving

```
Recieve data > Configure receiving
```

A new menu will open showing all listening ports. Go to:

```
New Receiving Port
```

Configure a TCP port on which our host, will be listening for TCP connections that send events to it. For that we have to give a specific configuration to the Universal Forwarders. For example the port 9001:

```
Listen on this port: 9001
```
The Receive Data menu will show the new port on which we will be listening


One way to check that this port is indeed open and working correctly is to perform TELNET to the port we have just configured:

```
telnet 127.0.0.1 9001
```

In order to be able to efficeinty view and analyse logs we are going to use the Splunk App "Splunk Add-On for Microsoft Sysmon 10.6.2" that will provide a data input and CIM-compliant field extractions for Microsoft Sysmon. Donwload from the Splunkbase website:

  - https://splunkbase.splunk.com/app/1914/

To install this App on the Splunk Enterprise, go to the following route

```
Settings > Apps > Install app from file
```

We can add the file in TAR format, or SPL or GZ format, as we wish. After we add it, it will appear in the applications menu.

```
"Microsoft Sysmon Add-on" was installed successfully
```

## Sysmon Installation & Configuration

Sysmon is a Windows system service that logs system activity to the Windows event log. It is a very powerful logging system but generates a lot of noise if not properly configured.

Switch to the **Windows 10 client** to be monitored and download Sysmon.

  - https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon

Install Sysmon in the x64 version and accept the license

```
Sysmon64.exe -accepteula -i
```

Check that the service is up

```
Sysmon64
```

Choose which configuration to use in Sysmon, currently the two most popular are. 
SwiftOnSecurity is a complete configuration but less up to date, Olafhartong is more up to date but is intended to be modular needing more maintenance

  - https://github.com/SwiftOnSecurity/sysmon-config
  - https://github.com/olafhartong/sysmon-modular

Apply the SwiftOnSecurity configuration.

```
Sysmon64  -c sysmonconfig-export.xml
```

And check, by performing a dump of the current configuration, if it has been applied correctly:

```
Sysmon64 -c | more
```

## Adding UniversalForwarders

UniversalForwarders provide reliable, secure data collection from remote sources and forward that data into Splunk software for indexing and consolidation.

At the **Windows 10 client** , download Splunk's Universal Forwarder

  - https://www.splunk.com/en_us/download/universal-forwarder/thank-you-universalforwarder.html

It can be done by executing the following wget:

```
wget -O splunkforwarder-8.1.2-545206cc9f70-x64-release.msi 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=windows&version=8.1.2&product=universalforwarder&filename=splunkforwarder-8.1.2-545206cc9f70-x64-release.msi&wget=true'
```

Run the MSI file from the command console with administrator permissions. It's required to do it in this way to access to certain log files in some routes.

```
.\splunkforwarder-8.1.2-545206cc9f70-x64-release.msi
```

The program will start and guide us through an installation wizard. It is not necessary to select anything in this installation wizard as we will later add certain configuration files that will provide everything needed to send logs to Splunk.
If the default installation directory has not been modified these are the two important configuration files

```
C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf
C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf
```

There is a lot of information in the following links

  - https://docs.splunk.com/Documentation/Forwarder/8.1.2/Forwarder/Configureforwardingwithoutputs.conf
  - 

The **inputs.conf** controls how the forwarder collects data. Is required to specify the WinEventLog channel that is going to be monitored.
Example:

```

[WinEventLog://Security]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
checkpointInterval = 5
current_only = 0
disabled = 0
start_from = oldest
```

The **outputs.conf** file defines how forwarders send data to receivers. Is required to specify the IP, or Hostname, and port of the server where this information is to be sent.
Example:

```

[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.52.154:9001

[tcpout-server://192.168.52.154:9001]
```

In order to make sure that the configuration was applied correctly, restart the service:

```
C:\Program Files\SplunkUniversalForwarder\bin\
splunk.exe restart
```

Check on Splunk if logs are being received correctly in the sourcetype:
```
sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational"
```

Finally, you now have a small lab to collect your Sysmon logs in Splunk in order to create Use Cases.
I hope the tutorial has been useful for someone!