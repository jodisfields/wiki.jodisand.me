#  Network Automation

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

#### What if im not using conda?

```bash
python -m venv network_automation
network_automation\\Scripts\\activate
pip install -r requirements.txt
```

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

