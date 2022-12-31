#  Network Automation with Python

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=2 orderedList=true } -->

<!-- code_chunk_output -->

1. [What is Automation?](#what-is-automation)
2. [Why Automate?](#why-automate)
3. [Requirements To Run The Scripts](#requirements-to-run-the-scripts)
4. [Getting Started](#getting-started)
5. [Setting Up a Local Development Environment](#setting-up-a-local-development-environment)
6. [Figuring Out What to Automate](#figuring-out-what-to-automate)
7. [Limitations of Python](#limitations-of-python)
8. [Why Python?](#why-python)
9. [Handy Scripts and Snippets](#handy-scripts-and-snippets)
10. [Netmiko Scripts for Inspiration](#netmiko-scripts-for-inspiration)
11. [Frequently Asked Questions](#frequently-asked-questions)

<!-- /code_chunk_output -->

##  What is Automation?

Network automation refers to the configuration, provisioning, testing and deployment of network equipment whilst employing techniques to minimize the need to introduce the potential for human error. This alone has the potential to increase the overall efficiency of the network whilst simultaneously lowering operational costs.

##  Why Automate?

The decision to automate should be driven by the need for a more reliable, manageable and scalable network. For the purposes of this document, I will be specifically focusing on what we are currently able to achieve with the tooling we have  already developed, tested and implemented as a means for effective configuration management and device auditing capability.

* Enables the ability to deeply analyse both current and historical performance of devices deployed to production can help assist in identifying any degradation of equipment before it is able to cause a major impact on the network.
* Enables the ability to maintain an accurate list of each devices version in order to effectively mitigate the risk of potential security incidents caused by running out of date or end-of-life software.
* Enables the ability to make a wide-spread configuration change to a large number of network devices simultaneously and reliably in the event of a data breach is a key factor in reducing the risk of a major security incident.

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

---

##   Figuring Out What to Automate

1.  Identify tasks that are performed frequently.
2.  Estimate how much time it takes to perform one of those tasks one time. Time yourself doing the task.
3.  If anyone else performs this task, estimate how often they do it, and how long it takes them. Add everyone’s time together.
4.  Break down the steps to the task into the smallest chunks possible.
-   Here’s an example from my experience when configuring a device with a template:
5.  Once a task is automated, record how long it takes to run and compare it to how long it takes to run manually and how often it’s run.


###   Open-Source Tools

When getting started, I recommend beginning to learn a few common tools, then expanding from there.


1.  **Python** - While everyone has their programming language of choice, there’s no disputing Python’s popularity and community support. If you’ve never written any code before, I highly recommend starting with Python. It’s easy to learn, beginner friendly, and there are lots of resources out there. Once you get familiar with Python, you can expand your skill set by learning additional programming languages.

2.  **Ansible** - Ansible is one of the most popular open-source automation tools out there. Begin by learning how Ansible works, and use it to start automating basic tasks in your network.

3.  **Git** - The de facto version control system out there, it’s entirely open-source and relatively easy to learn. Start by familiarizing yourself with how Git works, how to save (commit) changes, how to reverse (revert) changes when mistakes are made, etc. Note that Git is not the same as GitHub or GitLab. You do NOT need a GitHub/GitLab/etc. account to use Git.


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

##   Limitations of Python

There are times when Python isn’t the best language. There are some specific applications where other languages like C or Java are going to be better. Let’s see why.

###   Speed isn’t Python’s strong suit

Python is an interpreted language, meaning everything you do goes through an extra layer so the target machine can read and execute the code. It’s similar to talking through a translator with someone who speaks a different language.
Comparing Python with C, a compiled programming language, it’s true that Python code runs slower if we measure the execution time. Yet Python’s flexibility serves as a counterweight here. The language offers practical ways to solve problems and features dynamic typing, which contributes to rapid development. So Python beats C in terms of development time, which is often far more crucial than runtime performance, since less development time converts into lower costs and faster time to market.
Conclusion: If you have algorithms that need to run quickly (say, for sorting, searching, or just doing something on an integrated piece of hardware that needs to meet a specific speed requirement), you probably shouldn’t use Python.

###   High memory usage

Because of Python’s structure, it demands a lot of memory.

---

##   Why Python?

Well-structured, straightforward, easy to learn and use, concise yet expressive, versatile, and neat. That’s what Python is. Python focuses on code readability and visibility, which means developers can easily read, understand, and modify existing code and spend less time and effort actually coding. These advantages make Python one of the best languages for startups, since getting to market fast often means a competitive advantage and a faster return on investment.

###   It's Powerful

Python can be used to build almost anything, from websites to AI-powered solutions to numeric and scientific apps. The language provides a lot of standard libraries and features that address almost any programming need, which again results in rapid development.

###   Security is a priority

Cybersecurity threats are rapidly growing, which is why businesses look for ways to ensure maximum security.

Python proves to be a great choice for those focusing on data security. It provides features for authentication and authorization, email verification, and resetting passwords. Also, Python and its frameworks offer different mechanisms to address and reduce security-related issues such as cross-site request forgery (CSRF), cross-site scripting (XSS), SQL injection, and clickjacking.

###   Large community and high developer availability

Having a strong community is crucial for any language, since community members share experiences and help one another, create and upgrade features, update documentation, and troubleshoot problems. A large community, in short, grows the language and makes it develop faster.

Additionally, a large community means developers are easy to find. For example, there were 2,843 Python/Django outsourcing development companies listed on Clutch as of October 2020. As for Python developers, there were 8.2 million active Pythonistas in the world according to the Global Developer Population 2019 report by SlashData.

###   Good with AI and ML tasks

How does Amazon choose items just for you? And how does Spotify know what you want to listen to before you actually turn on the music? The answer is machine learning. And Python is the king of machine learning and artificial intelligence. Among its top machine learning use cases are content personalization, recommendations, image recognition, machine translation, speech recognition, fraud detection, and user behavior analytics.

---

## Handy Scripts and Snippets

### SSL Checker

Collects SSL/TLS information from all hosts defined in devices.txt

### Usage

```
python ssl_checker.py -h
```

`-f, --host-file` File containing hostnames for input

`-H, --host ` Enter the hosts separated by space

`-s, --socks ` Enable connection through SOCKS server

`-c, --csv ` Enable CSV file export by specifying filename.csv after this

#### Argument:

`-j, --json ` Use this if you want to only have the result in JSON

`-S, --summary ` This argument will show quick summary in the output

`-x, --html ` Enable HTML file export

`-J, --json-save` Use this if you want to save as JSON file per host

`-a, --analyze` This argument will include security analyze on the certificate. Takes more time. No result means failed to analyze.

`-v, --verbose` Shows more output. Good for troubleshooting.

`-h, --help`Shows the help and exit

#### Example

```bash
jodis@jodis-laptop:$ python check-ssl.py -H time.com github.com:443
+---------------------+
| Analyzing 2 host(s) |
+---------------------+
[+] time.com
-------------
Issued domain: time.com
Issued to: None
Issued by: Amazon (US)
Valid from: 2019-09-06
Valid to: 2020-10-06 (78 days left)
Validity days: 396
Certificate valid: True
Certificate S/N: 20641318859548253362475798736742284477
Certificate SHA1 FP: D5:CE:1B:77:AB:59:C9:BE:37:58:0F:5D:73:97:64:98:C4:3E:43:30
Certificate version: 2
Certificate algorithm: sha256WithRSAEncryption
Expired: False
Certificate SAN's:
    DNS:time.com
    DNS:*.time.com
[+] github.com
---------------
Issued domain: github.com
Issued to: GitHub, Inc.
Issued by: DigiCert Inc (US)
Valid from: 2020-05-05
Valid to: 2022-05-10 (659 days left)
Validity days: 735
Certificate valid: True
Certificate S/N: 7101927171473588541993819712332065657
Certificate SHA1 FP: 5F:3F:7A:C2:56:9F:50:A4:66:76:47:C6:A1:8C:A0:07:AA:ED:BB:8E
Certificate version: 2
Certificate algorithm: sha256WithRSAEncryption
Expired: False
Certificate SAN's:
    DNS:github.com
    DNS:www.github.com
+-------------------------------------------------------------------------------------------+
| Successful: 2 | Failed: 0 | Valid: 2 | Warning: 0 | Expired: 0 | Duration: 0:00:07.694433 |
+-------------------------------------------------------------------------------------------+
```

!!!NOTE: Keep in mind that if the certificate has less than 15 days validity, the script will consider it as a warning in the summary.

### Censored?

Try passing the `-s/--socks` argument to the script with the `HOST:PORT` format to connect through SOCKS proxy.

```shell
jodis@jodis-laptop: python check-ssl.py -H facebook.com
+-------------------+
|Analyzing 1 host(s)|
+-------------------+

[-] facebook.com    Failed: [Errno 111] Connection refused

+-------------------------------------------------------------------------------------------+
| Successful: 0 | Failed: 1 | Valid: 0 | Warning: 0 | Expired: 0 | Duration: 0:00:04.109058 |
+-------------------------------------------------------------------------------------------+

jodis@jodis-laptop: python check-ssl.py -H facebook.com -s localhost:9050
+---------------------+
| Analyzing 1 host(s) |
+---------------------+
[+] facebook.com
-----------------
Issued domain: *.facebook.com
Issued to: Facebook, Inc.
Issued by: DigiCert Inc (US)
Valid from: 2020-05-14
Valid to: 2020-08-05 (16 days left)
Validity days: 83
Certificate valid: True
Certificate S/N: 19351530099991824979726880175805235719
Certificate SHA1 FP: 89:7F:54:63:61:34:2F:7E:B4:B5:68:E2:92:79:D2:98:B4:97:D8:EA
Certificate version: 2
Certificate algorithm: sha256WithRSAEncryption
Expired: False
Certificate SAN's:
    DNS:*.facebook.com
    DNS:*.facebook.net
    DNS:*.fbcdn.net
    DNS:*.fbsbx.com
    DNS:*.messenger.com
    DNS:facebook.com
    DNS:messenger.com
    DNS:*.m.facebook.com
    DNS:*.xx.fbcdn.net
    DNS:*.xy.fbcdn.net
    DNS:*.xz.fbcdn.net
+-------------------------------------------------------------------------------------------+
| Successful: 1 | Failed: 0 | Valid: 1 | Warning: 0 | Expired: 0 | Duration: 0:00:00.416188 |
+-------------------------------------------------------------------------------------------+
```

### Quick Summary

Sometimes you need to run the script and get the quick summary of the hosts. By passing `-S/--summary` you will get the quick overview of the result.

```bash
jodis@jodis-laptop: python check-ssl.py -H narbeh.org:443 test.com twitter.com -S
+-------------------------------------------------------------------------------------------+
| Successful: 3 | Failed: 0 | Valid: 3 | Warning: 0 | Expired: 0 | Duration: 0:00:01.958670 |
+-------------------------------------------------------------------------------------------+
```

### Security Analyze

By passing `-a/--analyze` to the script, it will scan the certificate for security issues and vulnerabilities. It will also mark a grade for the certificate. **This will take more time to finish.**

```bash
jodis@jodis-laptop: python check-ssl.py -H narbeh.org:443 -a
+---------------------+
| Analyzing 1 host(s) |
+---------------------+

Warning: -a/--analyze is enabled. It takes more time...

[+] narbeh.org

Issued domain: narbeh.org
Issued to: None
Issued by: Let's Encrypt (US)
Valid from: 2018-04-21
Valid to: 2018-07-20 (88 days left)
Validity days: 90
Certificate S/N: 338163108483756707389368573553026254634358
Certificate version: 2
Certificate algorithm: sha256WithRSAEncryption
Certificate grade: A
Poodle vulnerability: False
Heartbleed vulnerability: False
Hearbeat vulnerability: True
Freak vulnerability: False
Logjam vulnerability: False
Drown vulnerability: False
Expired: False

+------------------------------------------------------+
| Successful: 1 | Failed: 0 | Duration: 0:00:01.429145 |
+------------------------------------------------------+
```

### JSON, HTML and CSV Output

Example only with the `-j/--json` argument which shows the JSON only. Perfect for piping to another tool.

```json
{
"narbeh.org": {
"host": "narbeh.org",
"issued_to": "sni.cloudflaressl.com",
"issued_o": "Cloudflare, Inc.",
"issuer_c": "US",
"issuer_o": "CloudFlare, Inc.",
"issuer_ou": null,
"issuer_cn": "CloudFlare Inc ECC CA-2",
"cert_sn": "20958932659753030511717961095784314907",
"cert_sha1": "FC:2D:0E:FD:DE:C0:98:7D:23:D2:E7:14:4C:07:6A:3D:25:25:49:B6",
"cert_alg": "ecdsa-with-SHA256",
"cert_ver": 2,
"cert_sans": "DNS:sni.cloudflaressl.com; DNS:narbeh.org; DNS:*.narbeh.org",
"cert_exp": false,
"cert_valid": true,
"valid_from": "2020-04-02",
"valid_till": "2020-10-09",
"validity_days": 190,
"days_left": 81,
"valid_days_to_expire": 81,
"tcp_port": 443
}
}
```

CSV export is also easy. After running the script with `-c/--csv` argument and specifying `filename.csv` after it, you'll have something like this:

```bash
jodis@jodis-laptop: ~/ssl-checker$ cat domain.csv
narbeh.org
issued_to,narbeh.org
valid_till,2018-07-20
valid_from,2018-04-21
issuer_ou,None
cert_ver,2
cert_alg,sha256WithRSAEncryption
cert_exp,False
issuer_c,US
issuer_cn,Let's Encrypt Authority X3
issuer_o,Let's Encrypt
validity_days,90
cert_sn,338163108483756707389368573553026254634358
```

Finally, if you want to export JSON's output per host in a separated file, use `-J/--json-save`. This will export JSON's output per host.

#### As a Python Module

```python
from ssl_checker import SSLChecker

SSLChecker = SSLChecker()
args = {
'hosts': ['google.com', 'cisco.com']
}

SSLChecker.show_result(SSLChecker.get_args(json_args=args))
```

---

## Netmiko Scripts for Inspiration

```python
from netmiko import ConnectHandler
from getpass import getpass

net_connect = ConnectHandler(
device_type="cisco_ios",
host="cisco1.lasthop.io",
username="pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
"password": password,
}

cisco2 = {
"device_type": "cisco_ios",
"host": "cisco2.lasthop.io",
"username": "pyclass",
"password": password,
}

nxos1 = {
"device_type": "cisco_nxos",
"host": "nxos1.lasthop.io",
"username": "pyclass",
"password": password,
}

srx1 = {
"device_type": "juniper_junos",
"host": "srx1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
"password": getpass(),
}

command = "copy flash:c880data-universalk9-mz.155-3.M8.bin flash:test1.bin"

## Start clock
start_time = datetime.now()

net_connect = ConnectHandler(**cisco1)

## Netmiko normally allows 100 seconds for send_command to complete
## delay_factor=4 would allow 400 seconds.
output = net_connect.send_command_timing(
command, strip_prompt=False, strip_command=False, delay_factor=4
)
## Router prompted in this example:
## -------
## cisco1#copy flash:c880data-universalk9-mz.155-3.M8.bin flash:test1.bin
## Destination filename [test1.bin]?
## Copy in progress...CCCCCCC
## -------
if "Destination filename" in output:
print("Starting copy...")
output += net_connect.send_command("
", delay_factor=4, expect_string=r"#")
net_connect.disconnect()

end_time = datetime.now()
print(f"
{output}\
")
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
"password": getpass(),
## Multiple all of the delays by a factor of two
"global_delay_factor": 2,
}

command = "show ip arp"
net_connect = ConnectHandler(**cisco1)
output = net_connect.send_command(command)
net_connect.disconnect()

print(f"
{output}\
")
```

### Using TextFSM

[Additional Details on Netmiko and TextFSM](https://pynet.twb-tech.com/blog/automation/netmiko-textfsm.html)

```python
from netmiko import ConnectHandler
from getpass import getpass
from pprint import pprint

cisco1 = {
"device_type": "cisco_ios",
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
"password": getpass(),
}

command = "del flash:/test4.txt"
net_connect = ConnectHandler(**cisco1)

## CLI Interaction is as follows:
## cisco1#delete flash:/testb.txt
## Delete filename [testb.txt]?
## Delete flash:/testb.txt? [confirm]y

## Use 'send_command' and the 'expect_string' argument (note, expect_string uses
## RegEx patterns). Netmiko will move-on to the next command when the
## 'expect_string' is detected.

## strip_prompt=False and strip_command=False make the output
## easier to read in this context.
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
User pyclass
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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
"host": "cisco1.lasthop.io",
"username": "pyclass",
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

host = "cisco1.lasthop.io"
device = {
"host": host,
"username": "pyclass",
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

device = {"host": "cisco1.lasthop.io", "username": "pyclass", "password": getpass()}

snmp_key = getpass("Enter SNMP community: ")
my_snmp = SNMPDetect(
"cisco1.lasthop.io",
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
"""
This is a complicated example.

It illustrates both using a terminal server and bouncing through multiple
devices.

It also illustrates using 'redispatch()' to change the Netmiko class.

The setup is:

Linux Server
--> Small Switch (SSH)
--> Terminal Server (telnet)

--> Juniper SRX (serial)
"""
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
net_connect.username = "pyclass"
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

pyclass@srx2>

Hostname: srx2
Model: srx110h2-va
JUNOS Software Release []
```

---

###   Traditional Network Management

With traditional networking, you can only manage network devices one at a time using SSH to the command line. This process is time-consuming, labor-intensive, and prone to human error. While this can work well in smaller networks, it does not work well in a large enterprise and does not also scale very well.
Traditional Network Monitoring Systems (NMS) such as SolarWinds, CiscoWorks, and Cisco Prime Infrastructure have also been available for a long time and use protocols such as SNMP and Netflow to gather information reports on the state of the network. While SNMP uses MIB and OID to collects useful data and can also be used to push configuration to devices, it does not build for real-time programmatic access and has limited functionality.
SNMP also has security concerns and can be complex to implement and operate. These are the main challenges of traditional Networks:

- Scalability Issues
- Management Complexity
- Slower Issue Resolution

###   Automate the network provisioning

- Perform network planning, designing, and implementation
- Supply virtual and physical networking devices or nodes
- Implement security compliance and improve the overall network security
- Simplify network management
- Automate reporting, monitoring, and management of enterprise network
- Analyze real-time network data to deliver insights
- To better understand what network automation is, let’s look at how it works.

###   How Does Network Automation Work?

Generally, you automate networks with programmable logic on the devices’ command line interface (CLI). This way, the nodes can dynamically perform automated actions. These include network filtering, port controlling, bandwidth control, etc.
However, this method isn’t scalable. It also limits the scope of automation as it needs to be done on a per device basis. As a result, most modern networking infrastructures offer a centralized control panel. This panel will connect all the networking devices within the network.
Network administrators or programmers can create programmable logic and scripts. This way, they can control and automate your network. To do this, they can use the devices’ CLI or graphical UI. Alternatively, admins can rely on external systems or automation tools. After that, programmers can execute these scripts using the CLI or API.
Automation tools and services can help you automate several everyday networking tasks. For instance, they can help you analyze and predict bandwidth usage. Automation will also help you with network inventory management and dynamic provisioning. You’ll also be able to backup and change configurations across your business. Finally, network automation tools can help you remotely control access points and ports.
###   Benefits of Network Automation

Generally, growing companies can’t manage and scale their infrastructure easily. But network automation helps you overcome this challenge. In turn, you’ll also improve your overall efficiency. Let’s take a more detailed look at the benefits of network automation:

- Eliminates manual tasks. This results in an increase in accuracy and efficiency. Standardizes processes. This reduces the risks of network outages and scalability issues. Monitors your network constantly and generates reports. This can help you better control your network infrastructure.
- Makes changes faster and builds a reliable network.
- Gives you network visibility and ease of control through a centralized dashboard.
- Tracks, analyzes, and resolves issues.
- Reduces human errors. This increases network resilience.
- Reduces the workload on IT staff. This can help organizations save capital costs.

###   Best Practices for Network Automation

Network automation tools and services use software abstraction. They connect networking devices and nodes. This way, they give you an easy-to-control network management workflow. Here are some best practices you can adopt to get the most out of network automation:

###   Decide What to Automate
Often, companies aren’t attentive when selecting a network automation vendor. This is a huge mistake. Before you start choosing a vendor, know what you need to automate in your network. This will help you narrow down your search list for the vendor. It’ll also help you save costs.

###   Aim for Low-Code Network Automation
The industry faces several challenges with the network automation skill gap. In fact, programming experience is lacking. Network scripting also has a steep learning curve. Did you know that only 3% of networking teams know how to perform automation tasks? So, choosing a service that allows low-code network automation could prove to be very beneficial. This will eliminate the dependency associated with certain employees. Low-code network automation will also limit human errors.

###   Adopt a Vendor-Agnostic Approach
Often, network architectures include solutions from many vendors. This means you should design a vendor-agnostic network orchestration. This mitigates the overhead associated with fixing errors or making changes to your network. Alternatively, you can find and deploy automation tools that support different vendors. This way, you can implement a true single view multi-vendor support.

###   Integrate with Other Tools and Services
It’s important to choose a service that integrates directly with external systems. For instance, integration with ELK and ServiceNow helps with log management and incident reporting. If your service has broad API integration support, you can streamline your business operations. You’ll be more flexible and you’ll use external tools to fit your needs.

###   Automate Configuration
If you automate configuration, you’ll ensure consistency across all your applications. You’ll also mitigate human errors. This enables your network admins to convert the configuration into software code. In turn, you can build a single source of truth for all configurations across your organization.

---

## Frequently Asked Questions

###   What is an SDN?
A software-defined network (SDN) is a networking technology. It uses software systems to design, deploy, manage, and configure enterprise networks. SDN allows organizations to intelligently and centrally control their entire network. That includes network peripherals and bandwidth usage.

###   What is a CLI?
Command Line Interface (CLI) is a text-based user interface. You can use it to develop, run, manage, and control software systems, tools, and applications. You also can use CLI to automate your network. To do this, you deploy scripts and automation tasks.

###   What is an API?
An application programming interface (API) establishes communication between different systems. An API is an intermediary that allows software systems to request and access information. You can access several network automation tools and services using APIs.

###   What is network orchestration?
Network orchestration is a process where a network controller helps achieve business objectives. The controller may design the network. It’ll also set up network peripherals and devices, applications, and services. You can leverage network orchestration to automate enterprise networks.

### What industries can benefit the most from network automation?
Network automation is domain and industry-agnostic. This means it’s applicable in all industries that require networking. These include Information Technology, manufacturing, service-based industries, banking, and cloud services.

###   What are the top companies using Python?
There are quite a lot of world-famous companies that prefer Python to other languages. They include Google, Netflix, Spotify, Uber, Dropbox, Instacart, Instagram, Stripe, Disqus, and Mozilla.

###   Where is Python used?
According to the Python Developer Survey 2019 by JetBrains, Python is mainly used for data analysis, website development, machine learning, DevOps, automation scripts, system administration, web parsers, crawlers, and scrapers, testing, educational purposes, software prototyping, and network programming.

###   How stable is Python?
Very stable. Famous apps written in Python like Pinterest, Disqus, and Instagram prove that. These apps use Python to handle high loads and ensure fast and efficient app performance.

###  What Is Network Automation?
Network automation is the process of using software to manage network resources and services. You can achieve network automation through a software-defined network (SDN). An SDN introduces network virtualization capabilities. This makes it easy to automate and control the networks.
Network automation also can help you configure, test, deploy, and operate components in your network. Since enterprise networks are becoming more complex, network automation tools are gaining traction. Here are some of the things these tools can do:

### Scripts vs Modules
In computing, the word script is used to refer to a file containing a logical sequence of orders or a batch processing file. This is usually a simple program, stored in a plain text file. Scripts are always processed by some kind of interpreter, which is responsible for executing each command sequentially. A plain text file containing Python code that is intended to be directly executed by the user is usually called script, which is an informal term that means top-level program file. On the other hand, a plain text file, which contains Python code that is designed to be imported and used from another Python file, is called module. So, the main difference between a module and a script is that modules are meant to be imported, while scripts are made to be directly executed.
In either case, the important thing is to know how to run the Python code you write into your modules and scripts.

### What’s the Python Interpreter?
Python is an excellent programming language that allows you to be productive in a wide variety of fields.
Python is also a piece of software called an interpreter. The interpreter is the program you’ll need to run Python code and scripts. Technically, the interpreter is a layer of software that works between your program and your computer hardware to get your code running.
Depending on the Python implementation you use, the interpreter can be:

- A program written in C, like CPython, which is the core implementation of the language
- A program written in Java, like Jython
- A program written in Python itself, like PyPy
- A program implemented in .NET, like IronPython

Whatever form the interpreter takes, the code you write will always be run by this program. Therefore, the first condition to be able to run Python scripts is to have the interpreter correctly installed on your system.

The interpreter is able to run Python code in two different ways:

- As a script or module
- As a piece of code typed into an interactive session

### How Does the Interpreter Work?
When you try to run Python scripts, a multi-step process begins. In this process the interpreter will:

- Process the statements of your script in a sequential fashion
- Compile the source code to an intermediate format known as bytecode
- This bytecode is a translation of the code into a lower-level language that’s platform-independent. Its purpose is to optimize code execution. So, the next time the interpreter runs your code, it’ll bypass this compilation step.
- Strictly speaking, this code optimization is only for modules (imported files), not for executable scripts.
- Ship off the code for execution

At this point, something known as a Python Virtual Machine (PVM) comes into action. The PVM is the runtime engine of Python. It is a cycle that iterates over the instructions of your bytecode to run them one by one.

The PVM is not an isolated component of Python. It’s just part of the Python system you’ve installed on your machine. Technically, the PVM is the last step of what is called the Python interpreter. The whole process to run Python scripts is known as the Python Execution Model.

### How to Run Python Scripts From a File Manager

Running a script by double-clicking on its icon in a file manager is another possible way to run your Python scripts. This option may not be widely used in the development stage, but it may be used when you release your code for production.

In order to be able to run your scripts with a double-click, you must satisfy some conditions that will depend on your operating system. Windows, for example, associates the extensions .py and .pyw with the programs python.exe and pythonw.exe respectively. This allows you to run your scripts by double-clicking on them.

When you have a script with a command-line interface, it is likely that you only see the flash of a black window on your screen. To avoid this annoying situation, you can add a statement like input`('Press Enter to Continue...')` at the end of the script. This way, the program will stop until you press Enter.

This trick has its drawbacks, though. For example, if your script has any error, the execution will be aborted before reaching the input() statement, and you still won’t be able to see the result.

On Unix-like systems, you’ll probably be able to run your scripts by double-clicking on them in your file manager. To achieve this, your script must have execution permissions, and you’ll need to use the shebang trick you’ve already seen. Likewise, you may not see any results on screen when it comes to command-line interface scripts.

Because the execution of scripts through double-click has several limitations and depends on many factors (such as the operating system, the file manager, execution permissions, file associations), it is recommended that you see it as a viable option for scripts already debugged and ready to go into production.
