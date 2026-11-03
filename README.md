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

Now I am given an option to select which package i need to install; RPM AMD64, RPM AARCH64, DEB AMD64, DEB AARCH64. I chose to go with DEB AMD64

Next I am instructed to input this command to start the installation of the agent:

wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.9-1_amd64.deb && sudo WAZUH_MANAGER='192.168.1.173' WAZUH_AGENT_NAME='arch.dillon' dpkg -i ./wazuh-agent_4.14.9-1_amd64.deb

I am receiving an error message saying; "sudo: dpkg: command not found"

This error just means that dpkg isn't installed on my device, so instead we will be using Archlinux's native repository to install the agent.

  sudo yay-S wazuh-agent

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

The command asks to return the information typed under the container server, in our case our IP address, meaning that the change worked. 

Next, we need to enroll with manager;

sudo /var/ossec/bin/agent-auth -m 192.168.1.173 -A arch.dillon

Once finished I restarted and agent to confirm all changes.

   sudo systemctl restart wazuh-agent
   sudo systemctl status wazuh-agent

Going back to the dashboard, it seems that the agent had successfully deployed and is receiving logs and information from the Archlinux device.

The endpoint has been successfully installed. 

