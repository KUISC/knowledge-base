# Splunk Installation Guide

## Requirements

- Splunk account
- Disregard for personal mental health
- `wget`

## Tested Hosts

These are the VMs that this guide has been tested with.

### Splunk Server

Lubuntu 24:

* Fresh minimal install
* Updated
* 64GB storage
* 8GB RAM
* 4 CPU

### Splunk Forwarder

Lubuntu 24.04:

* Fresh full install
* updated
* 32gb storage
* 2gb ram
* 2 cpu

Fedora Server 44:

* Fresh server install
* updated
* 32gb storage
* 2gb ram
* 2 cpu
* cockpit disabled


## Splunk Server Install

1. [Download](https://www.splunk.com/en_us/download/splunk-enterprise.html) the Splunk `.deb` package (use RPM for RHEL-based distros:
   ```
   wget -O splunk-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.4.3/linux/splunk-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package (this installs to `/opt/splunk`, use dnf for RHEL):
   ```
   dpkg -i {downloaded .deb}
   ```
3. Fix ownership of the install directory:
   ```
   sudo chown -R splunk:splunk /opt/splunk
   ```
4. Switch to the `splunk` user:
   ```
   sudo su splunk
   ```
5. Move into the Splunk `bin` directory:
   ```
   cd /opt/splunk/bin
   ```
6. Start Splunk:
   ```
   ./splunk start
   ```
   - Accept the license agreement
   - Login: `kuisc:kuisc123`

Once running, navigate to `http://localhost:8000` and log in with `kuisc:kuisc123`.

### Configuration

1. **Enable a receiving port:**
   Settings → Forwarding and Receiving → Receive Data → New Receiving Port
   - Pick a port (default `9997`)
   - Make sure it's enabled
2. **Create an index:**
   Settings → Indexes → New Index
   - Name it `linux` — team decision to use a single index for all Linux logging (syslog, journald, etc.)
   - Set any other options as needed

## Forwarder Install

Ships all journald logs to a remote indexer using `journald_input` — the journald input app bundled with the Universal Forwarder. This should be done on a **separate host** that you want to monitor.

1. [Download](https://www.splunk.com/en_us/download/universal-forwarder.html) the Universal Forwarder `.deb` package (rpm for RHEL):
   ```
   wget -O splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.4.3/linux/splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package (dnf for RHEL):
   ```
   dpkg -i {forwarder.deb}
   ```
3. Enable boot-start as the `splunkfwd` user, accepting the license and seeding the admin password:
   ```
   /opt/splunkforwarder/bin/splunk enable boot-start -user splunkfwd  --accept-license --answer-yes --no-prompt --seed-passwd 'kuisc123'
   ```
4. Switch to the `splunkfwd` user:
   ```
   su splunkfwd
   ```
5. Start the forwarder:
   ```
   /opt/splunkforwarder/bin/splunk start
   ```
6. Create `outputs.conf` (`/opt/splunkforwarder/etc/system/local/outputs.conf`) pointing at the remote indexer:
   ```
   [tcpout]
   defaultGroup = default-autolb-group

   [tcpout:default-autolb-group]
   server = <indexer-ip>:9997
   ```
7. Create the `local/` config directory for the bundled `journald_input` app, if it doesn't already exist:
   ```
   mkdir -p /opt/splunkforwarder/etc/apps/journald_input/local/
   ```
8. Create `inputs.conf` in that directory with a catch-all journald input:
   ```
   [journald://catch-all]
   index = linux
   sourcetype = journald
   disabled = false
   journalctl-quiet = true
   ```
9. Restart the forwarder to apply changes:
   ```
   /opt/splunkforwarder/bin/splunk restart
   ```
   - Accept the TOS prompt if shown
10. The beast has been slain.
