# Wazuh Intro

## About Wazuh
### What is Wazuh?

- Open source XDR & SIEM
	- Security Information Event Manager
- Collects data from many machines
- Parses that data into searchable events
- Real-time alerting based on that data

### Why should we use Wazuh?

- Allows for much more efficient defense
	- Instead of each person watching their own logs, a couple people watch Wazuh while the rest keep doing regular Sysadmin
			- Scripts can be made to quickly forward any event stream on a box to Wazuh
	- If red teamers know we're using centralized logging, they need to be stealthier and (usually) slower
- Vulnerability Mapping
	- Automatically gathers information about each of the systems the logger is deployed on
	- Checks for known CVEs and displays to dashboard
	- File integrity monitoring (like A.I.D.E.)
	- Security Configuration Assessmnt
		- checking service configs
	- Command monitoring

> **essentially, it makes the paranoid schizo part of blue team more manageable**

## Installing Wazuh

### Server/Indexer/Dashboard

> pulled mostly from [here](https://documentation.wazuh.com/current/quickstart.html#operating-system)

This contains the Wazuh Server, Indexer, and Dashboard.

1. Download and run the installation assistant:
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
Once the assistant finishes the installation, the output shows the access credentials and a message that confirms that the installation was successful.
```
INFO: --- Summary ---
INFO: You can access the web interface https://<WAZUH_DASHBOARD_IP_ADDRESS>
    User: admin
    Password: <ADMIN_PASSWORD>
INFO: Installation finished.
```
You now have installed and configured Wazuh.

2. Access the Wazuh web interface with `https://<WAZUH_DASHBOARD_IP_ADDRESS>` and your credentials:
```
    Username: admin

    Password: <ADMIN_PASSWORD>
```

To print passwords, run:
```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

### Agent

> see https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html

