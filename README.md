# Wazuh-SIEM-Documentation
Documentation of the installation process of the open source SIEM tool into my own home network.

documentation link for reference: https://documentation.wazuh.com/current/index.html

For this installation, I am thinking about following the virtual machine installation guide since it is much easier to set-up and install. 

Wazuh already provides a pre-built virtual machine image in the Open Virtual Appliance (OPA) format. 

Information about this virtual machine worth noting;
  Operating System: Amazon Linux 2023
  Architecture: 64-bit x86_64/AMD64 architecture
  VM Format: Open Virtual Appliance (OPA)
  Version: 4.14.9

Default Wazuh VM configurations:
  Component: Wazuh v4.14.9 OVA
  CPU (cores): 4
  RAM (GB): 8
  Storage (GB): 50

# Deploying an agent on ArchLinux

https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html

Starting off with the installation, it seemed pretty straight forward since Wazuh already provides a dedicated pre-built virtual machine. 

  1. Install latest version of Wazuh (4.14.9)
  2. Launch VMWare Workstation
  3. File > Open > Downloads > wazuh-4.14.9.ova

Once installed the credentials are already given and login is simple. 

Once inside, start up the command to launch the dashboard;

  sudo systemctl start wazuh-dashboard

It should take a little bit. Once finished you must find your ip address. Wazuh dashboard will launch by typing your ip address into the search bar and hitting enter. If there is an error message about credentials: continue anyway. 

note: you can find your ip address using the command: ip a

The dashboard has launched and login for the dashboard was simple as well. 

On the documentation website, there is a walkthrough on how to deploy an agent on different operating systems, but on the dashboard - there is already a section detailing how to deploy an agent. I will be following this method.

<img width="788" height="207" alt="Screenshot 2026-11-03 at 14-47-28 Wazuh" src="https://github.com/user-attachments/assets/b277fd49-474e-42aa-9333-d648d4dc01c8" />

Now I am given an option to select which package i need to install; RPM AMD64, RPM AARCH64, DEB AMD64, DEB AARCH64. I chose to go with DEB AMD64

Next I am instructed to input this command to start the installation of the agent:

wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.9-1_amd64.deb && sudo WAZUH_MANAGER='xxx.xxx.x.xxx' WAZUH_AGENT_NAME='arch.dillon' dpkg -i ./wazuh-agent_4.14.9-1_amd64.deb

I am receiving an error message saying; "sudo: dpkg: command not found"

This error just means that dpkg isn't installed on my device, so instead we will be using Archlinux's native repository to install the agent.

  sudo yay -S wazuh-agent

Starting up the agent gave an error message. 

wazuh-agent.service - Wazuh agent
     Loaded: loaded (/usr/lib/systemd/system/wazuh-agent.service; enabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2026-11-03 15:17:35 CST; 3min 35s ago
 Invocation: 21c067f48cc345699b3c425c2f37a4b2
    Process: 48646 ExecStart=/usr/bin/env /var/ossec/bin/wazuh-control start (code=exited, status=1/FAILURE)
   Mem peak: 3M
        CPU: 110ms

Nov 03 15:17:35 archlinux systemd[1]: Starting Wazuh agent...
Nov 03 15:17:35 archlinux env[48654]: 2026/11/03 15:17:35 wazuh-agentd: ERROR: (4112): Invalid server address found: 'MANAGER_IP'
Nov 03 15:17:35 archlinux env[48654]: 2026/11/03 15:17:35 wazuh-agentd: ERROR: (1215): No client configured. Exiting.
Nov 03 15:17:35 archlinux env[48646]: wazuh-agentd: Configuration error. Exiting
Nov 03 15:17:35 archlinux systemd[1]: wazuh-agent.service: Control process exited, code=exited, status=1/FAILURE
Nov 03 15:17:35 archlinux systemd[1]: wazuh-agent.service: Failed with result 'exit-code'.
Nov 03 15:17:35 archlinux systemd[1]: Failed to start Wazuh agent.

The agent has failed because we used Archlinux's native repository (AUR) instead of the given command, meaning we must manually go in and change all the placeholder sections with our actual information. 

We must change the placeholder manager ip with our own, so we use this command;

  sudo sed -i 's/MANAGER_IP/xxx.xxx.x.xxx/' /var/ossec/etc/ossec.conf

and to confirm that it worked;

   grep -A3 '<server>' /var/ossec/etc/ossec.conf

The command asks to return the information typed under the container "<server">, in our case our IP address, meaning that the change worked. 

Next, we need to enroll with manager;

sudo /var/ossec/bin/agent-auth -m xxx.xxx.x.xxx -A arch.dillon

Once finished I restarted and agent to confirm all changes.

   sudo systemctl restart wazuh-agent
   sudo systemctl status wazuh-agent

Going back to the dashboard, it seems that the agent had successfully deployed and is receiving logs and information from the Archlinux device.

The endpoint has been successfully installed. 

# Deploying an agent on Windows

https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html

To deploy an agent on windows I will be following the same steps as I did with my archlinux system. Though this time seems to be a lot easier. 

Following the same steps the prompt is asking me to perform this command. 

Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.9-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='xxx.xxx.x.xxx' WAZUH_AGENT_NAME='windows.dillon' 

Note: You will need administrator privileges to perform this installation. Powershell 3.0 or greater is required. Keep in mind you need to run this command in a Windows PowerShell terminal. 

Once installation is finished - launch the agent;

NET START Wazuh

Both the installation prompt and the startup command worked successfully, and yet it is not appearing in my Wazuh dashboard, I will wait a few minutes in case it needs some time to register. 

My first instinct is to check the manager connection between the computer and the agent. 

Test-NetConnection 192.168.1.173 -Port 1514
Test-NetConnection 192.168.1.173 -Port 1515

(Ports 1514 is used for events, and Port 1515 is used for enrollment.)

Both ports are open and active, meaning this could be an issue with the agent or the manager side. This command will show the agent logs and see what is failing. 

Get-Service WazuhSvc
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50

logs retuned as;

2026/11/05 13:33:47 wazuh-agent: INFO: Trying to connect to server ([xxx.xxx.x.126]:1514/tcp).
2026/11/05 13:34:08 wazuh-agent: ERROR: (1216): Unable to connect to '[xxx.xxx.x.126]:1514/tcp': 'A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.'.

Immediately we see the problem, the agent is trying to connect to the IP ending with 126, and respond with nothing because nothing is there. We must reconfigure the IP to our currently active one. 

In the admin powershell run;

notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"

This command opens this configuration file using notepad, within it we can look under the <server> block and manually change our IP address to the right one. 

Once we change it, we save and restart the service.

Restart-Service WazuhSvc
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 20

The Get-Content command gives us the logs showing that the agent has successfully started. Now the agent is showing up on the dashboard, which means the installation is complete. 

# Integrating Wazuh File Integrity Monitoring (FIM)

https://documentation.wazuh.com/current/proof-of-concept-guide/poc-file-integrity-monitoring.html

Wazuh's file integrity monitoring system to quite easy to implement into the Wazuh SIEM workspace. 

First, go into the Wazuh agent's /var/ossec/etc/ossec.conf configuration file using nano. 

sudo nano /var/ossec/etc/ossec.conf

Next, find the <syscheck> block, it is within this block we will add the directories for monitoring. Paste this in this block. (to save nano edit; ctrl + o to save, then enter to confirm filename, finally ctrl + x to exit)

<directories check_all="yes" report_changes="yes" realtime="yes">/root</directories>

<img width="775" height="296" alt="image" src="https://github.com/user-attachments/assets/5c1a3ce7-b645-4eb7-be73-9f33e58f2144" />

In our case, we will be configuring Wazuh to monitor the /root directory. Once done, restart the agent to apply the configuration changes. 

For windows we will be doing the same thing but with notepad instead. 

notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"

<img width="623" height="150" alt="image" src="https://github.com/user-attachments/assets/92826dc2-ce6a-409d-949d-242fbc46ca55" />

restart the agent to apply the configuration changes. 

# Integrating VirusTotal to work with Wazuh FIM

First I have to get my VirusTotal API key, all I needed to do to acquire it is to log into the VirusTotal website and go to my profile, where I was given my own API key.

Then I had to go back into /var/ossec/etc/ossec.conf configuration file and add an integration block within <syscheck> in order to connect VirusTotal and Wazuh together.


<img width="790" height="177" alt="image" src="https://github.com/user-attachments/assets/de104b16-f6b0-4aca-9c45-63e2d6c4c8c4" />

edit: we have changed the directories to:
  <directories check_all="yes" realtime="yes" report_changes="yes">/etc</directories>
  <directories check_all="yes" realtime="yes">/usr/bin,/usr/sbin,/boot</directories>
  <directories check_all="yes" realtime="yes" report_changes="yes">/home/dillonv/Downloads</directories>

After restarting to the manager and agent to both apply the configuration changes I was still not able to see the events show up on the dashboard. Further troubleshooting will be taken with the assistance of Claude AI. 

- Troubleshooting FIM dashboard not showing events

First we must confirm if syscheck is running on the Arch endpoint. 

  sudo /var/ossec/bin/wazuh-control status
  sudo grep -i syscheck /var/ossec/logs/ossec.log | tail -20

This will output the logs of what will be happening. Then we must generate some test event in a directory we have configured with 'realtime"yes"'.

  Added
  sudo touch /etc/fim_test.txt

  Modified (content)
  echo "hello" | sudo tee /etc/fim_test.txt

  Modified (permissions)
  sudo chmod 600 /etc/fim_test.txt

  Deleted
  sudo rm /etc/fim_test.txt

These three commands will prompt the creation, modification, permissions, and delete the test txt file. The purpose of this is to show if the manager catches the alert. 

We can now check that by using this command;

  sudo tail -f /var/ossec/logs/alerts/alerts.json | grep syscheck

On the dashboard, it still has not shown up. 

Let's see if we can confirm if the manager sees the agent. 

  sudo /var/ossec/bin/agent_control -l

It does. The problem could lie within the indexing pipeline (Filebeat to the indexer) rather than FIM. We can check to see if filebeat is active. 

  sudo systemctl status filebeat
  sudo filebeat test output

All commands return as active, meaning the problem isn't with Filebeat. 

Going back to the dashboard, I have set the time range to the last seven days. Now I am able to see the events. 

Running 'date -u' on both machines will give us the system date. For arch, displays 11/09, while the manager displays 11/06. We have found the problem. It is an asynchronization error from the manager. 

To fix this we need to set ntp to true using;

  timedatectl
  sudo timedatectl set-ntp true

and now we need to install chrony;

  sudo dnf install chrony
  sudo systemctl enable --now chronyd

Now we give it a minute then do;

  timedatectl
  timedatectl timesync-status

Since we have concluded that it was a time problem, we can also assume that the virtual machine also does not have time synchronization active. Be sure to this fix. 

Now we will force a one-time correction to make sure;

  sudo timedatctl set-ntp false
  sudo timedatectl set-time "2026-11-09 15:30:00" 

Note: this is a temporary fix, without ntp the clock will start to drift again.

Once the clock is correct we restart wazuh and retest. 

  sudo systemctl restart wazuh-manager filebeat

Now we must fix the clock inside the guest first:

On the wazuh server;

  timedatectl
  sudo timedatectl set-ntp true

then we must install Vmware's open-vm-tools to enable synchronize guest time with host in vm settings. 

to install we must run the package installation for RHEL;

  sudo dnf install open-vm-tools

now enable and start the service;

  sudo systemctl enable --now vmtoolsd

and to confirm if it is running;

  systemctl status vmtoolsd
  vmware-toolbox-cmd -v

and it has returned active, but if it was disabled this is how you enable it;

  sudo vmware-toolbox-cmd timesync enable

Now once we have the tools installed we can force another one-time resync;

  sudo vmware-toolbox-cmd timesync enable
  sudo systemctl restart vmtoolsd
  timedatectl

We restart wazuh and retest. 

  sudo touch /root/fim_test.txt
  sleep 10
  echo "hello" | sudo tee /root/fim_test.txt
  sleep 10
  sudo rm /root/fim_test.txt

and to check the manager (on wazuh-server)

  sudo grep '"location":"syscheck"' /var/ossec/logs/alerts/alerts.json | grep fim_test | tail -3

Now to chekc the dashboard, we now can see the alerts and events under File Integrity Monitoring. 



