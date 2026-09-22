Splunk Installation Guide

## Requirements

- Splunk account
- Disregard for personal mental health
- `wget`

## Host Specs

- Lubuntu 24.04RAM
- 4 CPU

### Splunk Server Install

1. Download the Splunk **.deb** package:
   ```
   wget -O splunk-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.4.3/linux/splunk-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package (this installs to **/opt/splunk**):
   ```
   dpkg -i {downloaded .deb}
   ```
3. Fix ownership of the install directory:
   ```
   sudo chown -R splunk:splunk /opt/splunk
   ```

- Fresh minimal install
- Updated
- 64GB storage
- 8GB RAM
- 4 CPU

## Splunk Server Install

1. Download the Splunk `.deb` package:
   ```
   wget -O splunk-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.4.3/linux/splunk-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package (this installs to `/opt/splunk`):
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
   - Login: `kuisc:kuisc123` (replace with comp-ready creds)

Once running, navigate to `http://localhost:8000` and log in with `kuisc:kuisc123` (or previously set creds).

### Configuration

1. **Enable a receiving port:**
   Settings → Forwarding and Receiving → Receive Data → New Receiving Port
   - Pick a port (default `9997`)
   - Make sure it's enabled
2. **Create an index:**
   Settings → Indexes → New Index
   - Name it to match whatever you're configuring in `inputs.conf` (here, `os`)
   - Set any other options as needed

## Forwarder Install

1. Download the Universal Forwarder `.deb` package:
   ```
   wget -O splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.4.3/linux/splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package:
   ```
   dpkg -i {forwarder.deb}
   ```
3. Give `splunkfwd` access to read the syslog file:
   ```
   # Allow the splunk user to read the current syslog file
   sudo setfacl -m u:splunkfwd:r /var/log/syslog

   # Allow the splunk user to read the parent directory to handle log rotations
   sudo setfacl -m u:splunkfwd:rx /var/log
   ```
4. Switch to the `splunkfwd` user:
   ```
   su splunkfwd
   ```
5. Move into the forwarder's local config directory:
   ```
   cd /opt/splunkforwarder/etc/system/local
   ```
6. Create `outputs.conf`:
   ```
   [tcpout]
   defaultGroup = primary_indexers

   [tcpout:primary_indexers]
   server = 127.0.0.1:9997
   ```
7. Create `inputs.conf`:
   ```
   [monitor:///var/log/syslog]
   disabled = false
   index = os
   sourcetype = syslog
   ```
8. Restart the forwarder:
   ```
   cd /opt/splunkforwarder/bin && ./splunk restart
   ```
   - Accept the TOS prompt if shown
9. The beast has been slain.
