# Splunk Installation Guide

Before diving into commands, the two-second version of how Splunk fits together: a **Splunk Enterprise** instance (the "indexer" or "server") is the thing with the web GUI that stores and lets you search logs. A **Universal Forwarder** is a stripped-down agent you put on every other host you actually want to monitor — it has no GUI, no search engine, no indexing, it just tails logs and ships them over TCP to the indexer. Everything below is either "set up the one indexer" or "set up another forwarder".

## Requirements

- Splunk account
- Disregard for personal mental health
- `wget` (Linux hosts only — Windows steps use the browser/MSI instead)

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

Windows:

* Not yet pinned to a specific tested image — the steps below assume a modern Windows Server (2019/2022) or Windows 10/11 host with PowerShell available, run as Administrator. 

## Splunk Server Install

1. [Download](https://www.splunk.com/en_us/download/splunk-enterprise.html) the Splunk `.deb` package (use RPM for RHEL-based distros). This is the full Splunk Enterprise install — GUI, indexer, the works — since this host is going to be the one everything else reports to:
   ```
   wget -O splunk-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.4.3/linux/splunk-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package (this installs to `/opt/splunk`, use dnf for RHEL):
   ```
   dpkg -i {downloaded .deb}
   ```
3. Fix ownership of the install directory. `dpkg` unpacked everything as `root`, but we don't want Splunk actually *running* as root day to day — a dedicated `splunk` user (created automatically by the package) keeps a compromised Splunk process from being a straight shot to root:
   ```
   sudo chown -R splunk:splunk /opt/splunk
   ```
4. Switch to the `splunk` user, so everything from here on runs — and gets configured — as the unprivileged account that'll actually own the running service:
   ```
   sudo su splunk
   ```
5. Move into the Splunk `bin` directory, which holds the `splunk` control script you'll be using for basically everything (start/stop/restart, config reloads, CLI admin commands):
   ```
   cd /opt/splunk/bin
   ```
6. Start Splunk. This first run is doing more than "start a process" — it's also generating default SSL certs, initializing the index storage on disk, and walking you through first-boot setup:
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

   Why: by default the indexer only listens for the web GUI (`8000`) and its own management port. Forwarders talk to it over a separate TCP port, and until you open one, forwarders have nowhere to send their data — they'll just queue it up and complain in their logs.

2. **Create an index:**
   Settings → Indexes → New Index
   - Name it `linux` — team decision to use a single index for all Linux logging (syslog, journald, etc.)
   - Set any other options as needed

   Why: an index is basically Splunk's version of a separate database/bucket on disk. Different indexes are useful for separating data with different retention needs, access permissions, or just to keep unrelated log sources from being mashed together in search results. We're deliberately keeping it simple with one index per OS family — `linux` here, `windows` later — and letting `sourcetype` do the finer-grained separation instead.

## Linux Forwarder Install

Ships all journald logs to a remote indexer using `journald_input` — the journald input app bundled with the Universal Forwarder. This should be done on a **separate host** that you want to monitor.

1. [Download](https://www.splunk.com/en_us/download/universal-forwarder.html) the Universal Forwarder `.deb` package (rpm for RHEL). Note this is a *different* download than the server — the Universal Forwarder is a much smaller package with no GUI or indexing, on purpose, since its only job on this box is to read logs and forward them:
   ```
   wget -O splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.4.3/linux/splunkforwarder-10.4.3-4174a2deda5d-linux-amd64.deb"
   ```
2. Install the package (dnf for RHEL):
   ```
   dpkg -i {forwarder.deb}
   ```
3. Enable boot-start as the `splunkfwd` user, accepting the license and seeding the admin password. `boot-start` registers Splunk as a proper init/systemd service so it comes back up on its own after a reboot — without this it's just a process you started by hand that dies the second the box restarts. The `--seed-passwd` flag exists because this is a non-interactive/scripted install, so we can't answer the "set a password" prompt a normal install would ask for:
   ```
   /opt/splunkforwarder/bin/splunk enable boot-start -user splunkfwd  --accept-license --answer-yes --no-prompt --seed-passwd 'kuisc123'
   ```
4. Switch to the `splunkfwd` user — same reasoning as the server: run the service as its own unprivileged account, not root:
   ```
   su splunkfwd
   ```
5. Start the forwarder:
   ```
   /opt/splunkforwarder/bin/splunk start
   ```
6. Create `outputs.conf` (`/opt/splunkforwarder/etc/system/local/outputs.conf`) pointing at the remote indexer. This file is what tells the forwarder *where to send data* — without it, the forwarder happily collects logs and just has nowhere to put them. `tcpout:default-autolb-group` defines the actual destination(s); `autolb` (auto load-balancing) means if you ever point this at more than one indexer, it'll spread the load across them instead of just failing over:
   ```
   [tcpout]
   defaultGroup = default-autolb-group

   [tcpout:default-autolb-group]
   server = <indexer-ip>:9997
   ```
7. Create the `local/` config directory for the bundled `journald_input` app, if it doesn't already exist. Splunk apps ship with a `default/` directory holding their factory config, which you're never supposed to edit directly — your own settings go in a sibling `local/` directory, which Splunk layers on top at runtime. This is the same pattern you'll see in basically every Splunk app:
   ```
   mkdir -p /opt/splunkforwarder/etc/apps/journald_input/local/
   ```
8. Create `inputs.conf` in that directory with a catch-all journald input. This is what actually tells the forwarder *what to monitor* — the previous steps just wired up the pipe, this is what starts pouring data into it. `index` sends these events to the `linux` index we created earlier, and `sourcetype` tags them so Splunk knows how to parse/field-extract them later at search time:
   ```
   [journald://catch-all]
   index = linux
   sourcetype = journald
   disabled = false
   journalctl-quiet = true
   ```
9. Restart the forwarder to apply changes — `inputs.conf` and `outputs.conf` are only read at startup, so nothing above takes effect until you do this:
   ```
   /opt/splunkforwarder/bin/splunk restart
   ```
   - Accept the TOS prompt if shown
10. The beast has been slain.

## Windows Forwarder Install

Same job as the Linux forwarder — get logs off a monitored host and onto the indexer — but Windows obviously has no journald. Instead you're after the **Windows Event Log** (Application, Security, System channels), which the Universal Forwarder can read natively once told to. Run all of this from an elevated (Administrator) PowerShell prompt.

1. [Download](https://www.splunk.com/en_us/download/universal-forwarder.html) the Windows Universal Forwarder `.msi` package. Windows doesn't ship `wget` by default the way Linux does, so grab this one from a browser or with `Invoke-WebRequest` if you have a direct link:
   ```
   Invoke-WebRequest -Uri "https://download.splunk.com/products/universalforwarder/releases/10.4.3/windows/splunkforwarder-10.4.3-4174a2deda5d-x64-release.msi" -OutFile "splunkforwarder.msi"
   ```
2. Install it silently from the command line, setting the indexer, credentials, and the license acceptance all in one shot. The MSI installer is doing what the `enable boot-start` + `--seed-passwd` dance did on Linux, plus actually installing the software — it registers a proper Windows service, so there's no separate "make it survive a reboot" step like on Linux:
   ```
   msiexec.exe /i splunkforwarder.msi AGREETOLICENSE=Yes RECEIVING_INDEXER="<indexer-ip>:9997" WINEVENTLOG_APP_CHECKBOX=1 SPLUNKUSERNAME=kuisc SPLUNKPASSWORD=kuisc123 /quiet
   ```
   - `RECEIVING_INDEXER` writes the equivalent of Linux's `outputs.conf` for you at install time.
   - `WINEVENTLOG_APP_CHECKBOX=1` installs the bundled Windows Event Log input app, which is the Windows analogue of the `journald_input` app on Linux.

   Note: unlike the Linux forwarder, the Windows service runs as `Local System` by default. There's no equivalent `chown`/`su` step here — the installer already sets up the service account for you, which is one of the few places Windows is actually *less* fiddly about this than Linux.

3. Confirm the service is installed and running:
   ```
   Get-Service SplunkForwarder
   ```
4. On the indexer, repeat the **Create an index** step from above, naming it `windows` instead of `linux` — same reasoning: keep OS families separated by index, let `sourcetype` handle the rest within each one.
5. Create `inputs.conf` at `C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf` (create the `local\` folder if it isn't there) to explicitly enable the three main event log channels. `WINEVENTLOG_APP_CHECKBOX=1` above sets up a default configuration, but writing it out yourself is worth doing once so you actually see what's being monitored and can point it at the `windows` index:
   ```
   [WinEventLog://Application]
   index = windows
   disabled = false

   [WinEventLog://Security]
   index = windows
   disabled = false

   [WinEventLog://System]
   index = windows
   disabled = false
   ```
6. Restart the service to pick up the new `inputs.conf` — same reason as Linux, config files are only read at startup/restart:
   ```
   Restart-Service SplunkForwarder
   ```
7. If events aren't showing up on the indexer, double-check Windows Defender Firewall isn't blocking outbound traffic on the receiving port (`9997` by default) — outbound is usually wide open, but it's the first thing to check if this silently doesn't work.
8. The beast has been slain, Windows edition.
