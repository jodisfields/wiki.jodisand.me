#  Network Automation

##  Requirements To Run The Scripts
The only requirements to run these scripts is the python runtime itself which can be downloaded directly from [python.org](https://www.python.org/downloads/ ) and a python module called [Netmiko](https://ktbyers.github.io/netmiko/docs/netmiko/index.html ).
- ***Python 3.6 or newer is required***
- Once installed simply run the following command to install Netmiko and its dependencies.
- Once the dependencies have been installed you will be able to execute python code simply by writting ```python``` followed by the name of the script you wish to run.thon
```
pip install -r requirements.txt
cd Scripts
python .\NAME-OF-SCRIPT.py
```

##  Getting Started

The below instructions will allow you to get started on your local machine for development and testing purposes. The following steps are to be used as a quick start guide for those who are new to the project and would like to get up and running quickly. Please note that these instructions are not intended to be a comprehensive guide but rather a quick start guide to deploy an environment similar to the one I have used to develop and test the scripts. I will assume that most users will be using a windows machine for this guide. However, if you are using a linux machine let me know and i will be happy to update the guide to include the linux equivalent of the steps below.

##  Setting Up a Local Development Environment

- The below instructions will allow you to get started on your **local machine** for development and testing purposes. ***Please Note the following instructions are not intended to be a comprehensive guide but rather a quick start guide to deploy an environment similar to the one I personally used to develop and test everything found in the project***.
- I have included a very simple network topology consisting of four Cisco IOSv Routers with OSPF and SSH pre configured to allow you to test the functionality of the scripts yourself. ***Please Note that you will need to change the IP addresses of each device  to match your local subnet.***
- Everything found in this project has been tested on both GNS3 and Cisco Modelling Labs, although I have not personally tested EVE-NG, I do not believe there would be any issues in using EVE-NG.
- Lastly, I will assume that most users will be using a windows machine for this guide. However, if you are using a linux machine let me know and i will be happy to update the guide to include the linux equivalent of the steps below.

###  Installing Python

There are a number of ways you can install Python on your machine, the below is my personal preference so feel free to install Python on your machine in any manner you prefer. Open a terminal and type in the following commands to install Python on your machine.

```bash
cd ~/Downloads/
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe --output miniconda.exe
./miniconda.exe
```

###  Installing Visual Studio Code
My prefered text editor is Visual Studio Code, however you can install any text editor you like on your machine. Popular text editors include Sublime Text, Atom, Brackets and Notepad++. Open a terminal and type in the following commands to install Visual Studio Code on your machine.

```bash
cd ~/Downloads/
curl https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user --output vscode.exe
./vscode.exe
```

###  Installing Dependencies

First, download the project files as a zip by clicking on the icon next to the clone button, and selecting "download as zip" then extract the files to your desktop. After you have the project files downloaded we can proceed to installing the dependencies. When using Python it is best practive to install the dependencies for each individual project in a separate virtual environment in order to negate the risk of packages conflicting with eachother. To create a virtual environment and install the dependencies for this project open a terminal and type in the following commands.

```bash
conda create -n network_automation python=3.9.5
conda activate network_automation
pip install -r requirements.txt
```

!!!warning What if im not using conda?
```bash
python -m venv network_automation
network_automation\\Scripts\\activate
pip install -r requirements.txt
```

###   Why network engineers should learn Python?

- It is easy to learn and read.
- It is perfect for scripting tasks which is perfect for network automation.
- Network vendors, especially Cisco have adopted Python as the language of choice.
- Lots of network automation tools are written in Python.
- Cisco already embed a Python shell in many switches.
- Networks are getting bigger but IT teams are not, we need to find better ways to do more with less.
- Stand out from the crowd and improve your career prospects.
- Python also has a great community, as one of the most popular programming languages in use today, it is very easy to find a solution to your problems.
- Python is already installed on Linux & MacOS and is now available on a lot of Cisco Switches.

###   How Python is useful in networking?

Python allows you to build scripts to automate complex network configuration. It is the most widely used programming language for software-defined networking, and is a critical skill for new network engineers. Learn the fundamentals of the language, including objects and variables, strings, loops, and functions.

###   Learning Python Will Be Highly Useful For Network Engineer

Python language has turned out to be one of the most recognized programming languages of recent times. The network engineers can also gain from it, this talent is most sought after by employers. Well, as you have understood by now, coding is crucial for the present network engineers; however, the network engineer is not going to concentrate only on it. This is where Python enters. You needn’t learn complex languages like C++ or Java but learn Python and take your networking career to the next level.

It is a simple tool for the management and server configurations and tasks. Python is that you require only less number of the code which is one of the main advantages. If you are an aspiring IT consultant, network engineer, or network consultant, then you will gain a lot after learning Python. The lowest line is that networking engineers should also have programming skills to utilize the new tools and also the latest programming wave.

---

## Handy Scripts and Snippets

### Using Netmiko

```python
from netmiko import ConnectHandler
from getpass import getpass

net_connect = ConnectHandler(
device_type="cisco_ios",
host="10.0.0.10",
username="jodis",
password=getpass(),
)

print(net_connect.find_prompt())
net_connect.disconnect()
```

### Connect using a dictionary

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

net_connect = ConnectHandler(**cisco1)
print(net_connect.find_prompt())
net_connect.disconnect()
```

### Dictionary with a context manager

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

## Will automatically 'disconnect()'
with ConnectHandler(**cisco1) as net_connect:
print(net_connect.find_prompt())
```

### Enable mode

```python
from netmiko import ConnectHandler
from getpass import getpass

password = getpass()
secret = getpass("Enter secret: ")

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": password,
"secret": secret,
}

net_connect = ConnectHandler(**cisco1)
## Call 'enable()' method to elevate privileges
net_connect.enable()
print(net_connect.find_prompt())
net_connect.disconnect()
```

### Connecting to multiple devices.

```python
from netmiko import ConnectHandler
from getpass import getpass

password = getpass()

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": password,
}

cisco2 = {
"device_type": "cisco_ios",
"host": "10.0.0.11",
"username": "jodis",
"password": password,
}

nxos1 = {
"device_type": "cisco_nxos",
"host": "10.0.0.12",
"username": "jodis",
"password": password,
}

srx1 = {
"device_type": "juniper_junos",
"host": "10.0.0.13",
"username": "jodis",
"password": password,
}

for device in (cisco1, cisco2, nxos1, srx1):
net_connect = ConnectHandler(**device)
print(net_connect.find_prompt())
net_connect.disconnect()
```

### Executing show command.

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

## Show command that we execute.
command = "show ip int brief"

with ConnectHandler(**cisco1) as net_connect:
output = net_connect.send_command(command)

## Automatically cleans-up the output so that only the show output is returned
print()
print(output)
print()
```

!!!note Output:
     ```bash
     Password:

     IP-    OK? Method Status    Protocol
     FastEthernet0    unassignedYES unset      down
     FastEthernet1    unassignedYES unset      down
     FastEthernet2    unassignedYES unset  down
     down
     FastEthernet3    unassignedYES unset      down
     FastEthernet4    10.220.88.20    YES NVRAM  up    up
     Vlan1    unassignedYES unset      down
     ```

### Adjusting delay_factor

```python
from netmiko import ConnectHandler
from getpass import getpass
from datetime import datetime

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

command = "copy flash:c880data-universalk9-mz.155-3.M8.bin flash:test1.bin"

## Start clock
start_time = datetime.now()

net_connect = ConnectHandler(**cisco1)

## Netmiko normally allows 100 seconds for send_command to complete
## delay_factor=4 would allow 400 seconds.
output = net_connect.send_command_timing(command, strip_prompt=False, strip_command=False, delay_factor=4)

if "Destination filename" in output:
print("Starting copy...")
output = net_connect.send_command("", delay_factor=4, expect_string=r"#")
net_connect.disconnect()

end_time = datetime.now()
print(f"{output}")
print("done")
print(f"Execution time: {start_time - end_time}")
```

### Using global_delay_factor

[Additional details on global_delay_factor](https://pynet.twb-tech.com/blog/automation/netmiko-what-is-done.html)

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
## Multiple all of the delays by a factor of two
"global_delay_factor": 2,
}

command = "show ip arp"
net_connect = ConnectHandler(**cisco1)
output = net_connect.send_command(command)
net_connect.disconnect()

print(f"{output}")
```

### Using TextFSM

[Additional Details on Netmiko and TextFSM](https://pynet.twb-tech.com/blog/automation/netmiko-textfsm.html)

```python
from netmiko import ConnectHandler
from getpass import getpass
from pprint import pprint

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

command = "show ip int brief"
with ConnectHandler(**cisco1) as net_connect:
## Use TextFSM to retrieve structured data
output = net_connect.send_command(command, use_textfsm=True)

print()
pprint(output)
print()
```
!!!note Output:
     ```json
     Password:

     [{'intf': 'FastEthernet0',
     'ipaddr': 'unassigned',
     'proto': 'down',
     'status': 'down'},
     {'intf': 'FastEthernet1',
     'ipaddr': 'unassigned',
     'proto': 'down',
     'status': 'down'},
     {'intf': 'FastEthernet2',
     'ipaddr': 'unassigned',
     'proto': 'down',
     'status': 'down'},
     {'intf': 'FastEthernet3',
     'ipaddr': 'unassigned',
     'proto': 'down',
     'status': 'down'},
     {'intf': 'FastEthernet4',
     'ipaddr': '10.220.88.20',
     'proto': 'up',
     'status': 'up'},
     {'intf': 'Vlan1',
     'ipaddr': 'unassigned',
     'proto': 'down',
     'status': 'down'}]
     ```

### Using TTP

```python
from netmiko import ConnectHandler
from getpass import getpass
from pprint import pprint

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

## write template to file
ttp_raw_template = """
interface {{ interface }}
description {{ description }}
"""

with open("show_run_interfaces.ttp", "w") as writer:
writer.write(ttp_raw_template)

command = "show run | s interfaces"
with ConnectHandler(**cisco1) as net_connect:
## Use TTP to retrieve structured data
output = net_connect.send_command(
command, use_ttp=True, ttp_template="show_run_interfaces.ttp"
     )

print()
pprint(output)
print()
```
!!!note Output:
     ```json
     [[[{'description': 'Router-id-loopback',
     'interface': 'Loopback0'},
     {'description': 'CPE_Acces_Vlan',
     'interface': 'Vlan778'}]]]
     ```
### Using Genie

```python
from getpass import getpass
from pprint import pprint
from netmiko import ConnectHandler

device = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass()
}

with ConnectHandler(**device) as net_connect:
output = net_connect.send_command("show ip interface brief", use_genie=True)

print()
pprint(output)
print()
```

!!!note Output:
     ```js
     $ python send_command_genie.py
     Password:

     {'interface': {'FastEthernet0': {'interface_is_ok': 'YES',

     'ip_address': 'unassigned',
     'method': 'unset',

     'protocol': 'down',
     'status': 'down'},

     'FastEthernet1': {'interface_is_ok': 'YES',
     'ip_address': 'unassigned',
     'method': 'unset',
     'protocol': 'down',
     'status': 'down'},
     'FastEthernet2': {'interface_is_ok': 'YES',
     'ip_address': 'unassigned',
     'method': 'unset',
     'protocol': 'down',
     'status': 'down'},
     'FastEthernet3': {'interface_is_ok': 'YES',
     'ip_address': 'unassigned',
     'method': 'unset',
     'protocol': 'down',
     'status': 'down'},
     'FastEthernet4': {'interface_is_ok': 'YES',
     'ip_address': '10.220.88.20',
     'method': 'NVRAM',
     'protocol': 'up',
     'status': 'up'},
     'Vlan1': {'interface_is_ok': 'YES',
     'ip_address': 'unassigned',
     'method': 'unset',
     'protocol': 'down',

     'status': 'down'}}}
     ```

### Handling commands that prompt (timing)

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

command = "del flash:/test3.txt"
net_connect = ConnectHandler(**cisco1)

## CLI Interaction is as follows:
## cisco1#delete flash:/testb.txt
## Delete filename [testb.txt]?
## Delete flash:/testb.txt? [confirm]y

## Use 'send_command_timing' which is entirely delay based.
## strip_prompt=False and strip_command=False make the output
## easier to read in this context.
output = net_connect.send_command_timing(
command_string=command,
strip_prompt=False,
strip_command=False
)
if "Delete filename" in output:
output += net_connect.send_command_timing(
command_string="
",
strip_prompt=False,
strip_command=False
)
if "confirm" in output:
output += net_connect.send_command_timing(
command_string="y",
strip_prompt=False,
strip_command=False
)
net_connect.disconnect()

print()
print(output)
     print()
```

!!!note Output:
     ```bash
     Password:

     del flash:/test3.txt
     Delete filename [test3.txt]?
     Delete flash:/test3.txt? [confirm]y
     cisco1#
     cisco1#
     ```

### Handling commands that prompt (expect_string)

```python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

command = "del flash:/test4.txt"
net_connect = ConnectHandler(**cisco1)


output = net_connect.send_command(
command_string=command,
expect_string=r"Delete filename",
strip_prompt=False,
strip_command=False
)
output += net_connect.send_command(
command_string="
",
expect_string=r"confirm",
strip_prompt=False,
strip_command=False
)
output += net_connect.send_command(
command_string="y",
expect_string=r"#",
strip_prompt=False,
strip_command=False
)
net_connect.disconnect()

print()
print(output)
print()
```

!!!note Output:
     ```
     $ python send_command_prompting_expect.py
     Password:

     del flash:/test4.txt
     Delete filename [test4.txt]?
     Delete flash:/test4.txt? [confirm]y
     cisco1#
     ```

### Configuration changes

```python
#!/usr/bin/env python
from netmiko import ConnectHandler
from getpass import getpass

device = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

commands = ["logging buffered 100000"]
with ConnectHandler(**device) as net_connect:
output = net_connect.send_config_set(commands)
output += net_connect.save_config()

print()
print(output)
print()
```

!!!note Output:
     ```bash
     $ python config_change.py
     Password:

     configure terminal
     Enter configuration commands, one per line.  End with CNTL/Z.
     cisco1(config)#logging buffered 100000
     cisco1(config)#end
     cisco1#write mem
     Building configuration...
     [OK]
     cisco1#
     ```

### Configuration changes from a file

```python
#!/usr/bin/env python
from netmiko import ConnectHandler
from getpass import getpass

device1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

## File in same directory as script that contains
#
## $ cat config_changes.txt
## --------------
## logging buffered 100000
## no logging console

cfg_file = "config_changes.txt"
with ConnectHandler(**device1) as net_connect:
output = net_connect.send_config_from_file(cfg_file)
output += net_connect.save_config()

print()
print(output)
print()
```

#### Netmiko will automatically enter and exit config mode.
!!!note Output:
          ```bash
          $ python config_change_file.py
          Password:

          configure terminal
          Enter configuration commands, one per line.  End with CNTL/Z.
          cisco1(config)#logging buffered 100000
          cisco1(config)#no logging console
          cisco1(config)#end
          cisco1#write mem
          Building configuration...
          [OK]
          cisco1#
               ```

### SSH keys

```python
from netmiko import ConnectHandler

key_file = "~/.ssh/test_rsa"
cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "testuser",
"use_keys": True,
"key_file": key_file,
}

with ConnectHandler(**cisco1) as net_connect:
output = net_connect.send_command("show ip arp")

print(f"{output}")
```

### SSH Config File

```python
#!/usr/bin/env python
from netmiko import ConnectHandler
from getpass import getpass
cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
"ssh_config_file": "~/.ssh/ssh_config",
}

with ConnectHandler(**cisco1) as net_connect:
output = net_connect.send_command("show users")

print(output)
```

#### Contents of 'ssh_config' file

```ini
host jumphost
IdentitiesOnly yes
IdentityFile ~/.ssh/test_rsa
User gituser
HostName pynetqa.lasthop.io

host * !jumphost
User jodis
## Force usage of this SSH config file
ProxyCommand ssh -F ~/.ssh/ssh_config -W %h:%p jumphost
## Alternate solution using netcat
#ProxyCommand ssh -F ./ssh_config jumphost nc %h %p
```


### Session log

```python
#!/usr/bin/env python
from netmiko import ConnectHandler
from getpass import getpass

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
## File name to save the 'session_log' to
"session_log": "output.txt"
}

## Show command that we execute
command = "show ip int brief"
with ConnectHandler(**cisco1) as net_connect:
output = net_connect.send_command(command)
```

#### Contents of 'output.txt' after execution

```bash
$ cat output.txt

cisco1#
cisco1#terminal length 0
cisco1#terminal width 511
cisco1#
cisco1#show ip int brief
Interface
IP-    OK? Method Status    Protocol
FastEthernet0    unassignedYES unset      down
FastEthernet1    unassignedYES unset      down
FastEthernet2    unassignedYES unset      down
FastEthernet3    unassignedYES unset      down
FastEthernet4    10.220.88.20    YES NVRAM  up    up
Vlan1    unassignedYES unset      down
cisco1#
cisco1#exit
```

### Standard Logging

The below will create a file named "test.log". This file will contain a lot of
low-level details.

```python
from netmiko import ConnectHandler
from getpass import getpass

# Logging section ##############
import logging

logging.basicConfig(filename="test.log", level=logging.DEBUG)
logger = logging.getLogger("netmiko")
# Logging section ##############

cisco1 = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

net_connect = ConnectHandler(**cisco1)
print(net_connect.find_prompt())
net_connect.disconnect()
```

### Secure Copy

```python
from getpass import getpass
from netmiko import ConnectHandler, file_transfer

cisco = {
"device_type": "cisco_ios",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

## A secure copy server must be enable on the device ('ip scp server enable')
source_file = "test1.txt"
dest_file = "test1.txt"
direction = "put"
file_system = "flash:"

ssh_conn = ConnectHandler(**cisco)
transfer_dict = file_transfer(
ssh_conn,
source_file=source_file,
dest_file=dest_file,
file_system=file_system,
direction=direction,
## Force an overwrite of the file if it already exists
overwrite_file=True,
)

print(transfer_dict)
```

### Auto detection using SSH

```python
from netmiko import SSHDetect, ConnectHandler
from getpass import getpass

device = {
"device_type": "autodetect",
"host": "10.0.0.10",
"username": "jodis",
"password": getpass(),
}

guesser = SSHDetect(**device)
best_match = guesser.autodetect()
#print(best_match)  # Name of the best device_type to use further
#print(guesser.potential_matches)  # Dictionary of the whole matching result
## Update the 'device' dictionary with the device_type
device["device_type"] = best_match

with ConnectHandler(**device) as connection:
print(connection.find_prompt())
```

### Auto detection using SNMPv2c

Requires 'pysnmp'.

```python
import sys
from getpass import getpass
from netmiko.snmp_autodetect import SNMPDetect
from netmiko import ConnectHandler

host = "10.0.0.10"
device = {
"host": host,
"username": "jodis",
"password": getpass()
}

snmp_community = getpass("Enter SNMP community: ")
my_snmp = SNMPDetect(
host, snmp_version="v2c", community=snmp_community
)
device_type = my_snmp.autodetect()
print(device_type)

if device_type is None:
sys.exit("SNMP failed!")

## Update the device dictionary with the device_type and connect
device["device_type"] = device_type
with ConnectHandler(**device) as net_connect:
print(net_connect.find_prompt())
```

### Auto detection using SNMPv3

Requires 'pysnmp'.

```python
import sys
from getpass import getpass
from netmiko.snmp_autodetect import SNMPDetect
from netmiko import ConnectHandler

device = {"host": "10.0.0.10", "username": "jodis", "password": getpass()}

snmp_key = getpass("Enter SNMP community: ")
my_snmp = SNMPDetect(
"10.0.0.10",
snmp_version="v3",
user="pysnmp",
auth_key=snmp_key,
encrypt_key=snmp_key,
auth_proto="sha",
encrypt_proto="aes128",
)
device_type = my_snmp.autodetect()
print(device_type)

if device_type is None:
sys.exit("SNMP failed!")

## Update the device_type with information discovered using SNMP
device["device_type"] = device_type
net_connect = ConnectHandler(**device)
print(net_connect.find_prompt())
net_connect.disconnect()
```

### Terminal server and redispatch

```python
import os
from getpass import getpass
from netmiko import ConnectHandler, redispatch

## Hiding these IP addresses
terminal_server_ip = os.environ["TERMINAL_SERVER_IP"]
public_ip = os.environ["PUBLIC_IP"]

s300_pass = getpass("Enter password of s300: ")
term_serv_pass = getpass("Enter the terminal server password: ")
srx2_pass = getpass("Enter SRX2 password: ")

## For internal reasons I have to bounce through this small switch to access
## my terminal server.
device = {
"device_type": "cisco_s300",
"host": public_ip,
"username": "admin",
"password": s300_pass,
"session_log": "output.txt",
}

## Initial connection to the S300 switch
net_connect = ConnectHandler(**device)
print(net_connect.find_prompt())

## Update the password as the terminal server uses different credentials
net_connect.password = term_serv_pass
net_connect.secret = term_serv_pass
## Telnet to the terminal server
command = f"telnet {terminal_server_ip}\
"
net_connect.write_channel(command)
## Use the telnet_login() method to handle the login process
net_connect.telnet_login()
print(net_connect.find_prompt())

## Made it to the terminal server (this terminal server is "cisco_ios")
## Use redispatch to re-initialize the right class.
redispatch(net_connect, device_type="cisco_ios")
net_connect.enable()
print(net_connect.find_prompt())

## Now connect to the end-device via the terminal server (Juniper SRX2)
net_connect.write_channel("srx2\
")
## Update the credentials for SRX2 as these are different.
net_connect.username = "jodis"
net_connect.password = srx2_pass
## Use the telnet_login() method to connect to the SRX
net_connect.telnet_login()
redispatch(net_connect, device_type="juniper_junos")
print(net_connect.find_prompt())

## Now we could do something on the SRX
output = net_connect.send_command("show version")
print(output)

net_connect.disconnect()
```

#### Output from execution of this code (slightly cleaned-up).

```bash
$ python term_server.py
Enter password of s300:
Enter the terminal server password:
Enter SRX2 password:

sf-dc-sw1#

twb-dc-termsrv>
twb-dc-termsrv#

jodis@srx2>

Hostname: srx2
Model: srx110h2-va
JUNOS Software Release []
```

### Term Server - DND

