# Splunk & Sysmon installation and configuration

This is a tutorial in which it will be shown how to collect Sysmon logs in Splunk in order to perform a cybersecurity lab oriented to Purple Team.
In this tutorial we are going to use two virtual machines: A Debian 9 which will be the Splunk server, and a client host which will be a Windows 10 with Sysmon. 

This are the versions of the software used:

  - Splunk Enterprise 8.1.2
  - Splunk UniversalForwarder
  - Sysmon 13.1

## Splunk Installation

First we download the latest version of Splunk Enterprise (which can also be done via command line)

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

## Sysmon Installation & Configuration

## Adding UniversalForwarders











