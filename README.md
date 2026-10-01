Ækeynox-kanata
================================================================================

Reference [Kanata] implementation of the [Arsenik] keymap.

[Kanata]:  https://github.com/jtroo/kanata
[Arsenik]: https://github.com/OneDeadKey/arsenik


In a Nutshell
--------------------------------------------------------------------------------

- [Arsenik] is a laptop-first, 33-key keymap that has been designed to work on
  any keyboard: ANSI, ISO, ergonomic…
- [Kanata] is a cross-platform Rust application that implements QMK-like
  features: mod-taps, layers…
- this repository provides Kanata configuration files to activate Arsenik on
  your PC or Mac

![base, navigation and sym layers on a 33-key keyboard](img/arsenik.svg)

<!--  No release yet. Propose a zip archive instead. -->
[dl-aekeynox]: https://github.com/OneDeadKey/kanata-config-aekeynox/archive/refs/heads/main.zip
[dl-kanata]:   https://github.com/jtroo/kanata/releases


Usage
--------------------------------------------------------------------------------

- Get Ækeynox:
  - check out this repository with Git or [download it][dl-aekeynox]
  - [configure `kanata.kbd`](#configuration): Ækeynox-kanata does nothing by
    default, every feature must be enabled on an opt-in basis.
- Get Kanata:
  - download a [pre-built executable][dl-kanata]
  - unzip it into the Ækeynox folder
- Run Kanata.

See OS-specific instructions below.


### Windows

There are several Kanata variants for Windows. We recommend using
`kanata_windows_gui_winIOv2_[arch].exe`: put it in the Ækeynox folder, start it
(double-click), and Kanata will run in the background. It can be reloaded or
exited with a right-click on its systray icon.

### Linux

<details>
<summary>Run Kanata without <code>sudo</code></summary>

Kanata needs to intercept `uinput` signals, which it cannot do without the
proper authorisations.

If you don’t want to run `kanata` with `sudo`, you’ll need to allow Kanata to
read from `uinput`. This requires the users to be part of both `input` and
`uinput` groups.

For that, you first need to create a `uinput` group if it doesn’t exist yet:

```bash
sudo groupadd -U $USERNAME uinput
```

where `$USERNAME` is the target user (or users in a comma-separated list). Then
add the target user (or users) to the group input:

```bash
sudo usermod -aG input $USERNAME
```

You can check after re-logging that both groups appear in the result of the
`groups` command launched as the target user.

Finally, you need to add a udev rule in `/etc/udev/rules.d/50-kanata.rules`:

```udev
KERNEL=="uinput", MODE="0660", GROUP="uinput", OPTIONS+="static_node=uinput"
```
</details>

<details>
<summary>Make a User-Side <code>systemd</code> Service for Kanata</summary>

Note: This only works if `kanata` is able to run without `sudo` (and is using
`systemd`).

Using a `systemd service` allows running `kanata` as a daemon, possibly right
after logging in. Here is a template for a service file:

```properties
[Unit]
Description=kanata keyboard remapper
Documentation=https://github.com/jtroo/kanata

[Service]
Environment=PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/bin
Environment=DISPLAY=:0
Environment=HOME=/path/to/home/folder
Type=simple
ExecStart=/usr/local/bin/kanata --cfg /path/to/kanata/config/file
Restart=no

[Install]
WantedBy=default.target
```

Copy-paste it into `~/.config/systemd/user/kanata.service`, fill in the
placeholders, then run one of the following commands:

- `systemctl --user start kanata.service` to manually start `kanata`
- `systemctl --user enable kanata.service` so `kanata` may autostart whenever
the current user logs in
- `systemctl --user status kanata.service` to check if `kanata` is running
</details>


### macOS

<details>
<summary>Prerequisite: Karabiner DriverKit</summary>

Do not install the latest version. Pick one of these two versions, according to your OS:

- macOS 11 and newer: [Karabiner DriverKit v6.2.0](https://github.com/pqrs-org/Karabiner-DriverKit-VirtualHIDDevice/releases/tag/v6.2.0)
- macOS 10 and older: [Karabiner DriverKit v4.3.0](https://github.com/pqrs-org/Karabiner-DriverKit-VirtualHIDDevice/releases/tag/v4.3.0)

To activate it:

```sh
/Applications/.Karabiner-VirtualHIDDevice-Manager.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Manager activate
```

You may have to allow Kanata’s execution in the *“Privacy & Security”* panel,
from the system settings.

As root, create this file:

```
/Library/LaunchDaemons/org.pqrs.service.daemon.Karabiner-VirtualHIDDevice-Daemon.plist
```

with the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>org.pqrs.service.daemon.Karabiner-VirtualHIDDevice-Daemon</string>
    <key>KeepAlive</key>
    <true/>
    <key>ProcessType</key>
    <string>Interactive</string>
    <key>ProgramArguments</key>
    <array>
      <string>/Library/Application Support/org.pqrs/Karabiner-DriverKit-VirtualHIDDevice/Applications/Karabiner-VirtualHIDDevice-Daemon.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Daemon</string>
    </array>
  </dict>
</plist>
```

A new item *Fumihiko Takayama* will be added in System Settings > Login Items.

Two Karabiner processes should be started:

```sh
ps aux | grep -i karabiner
```

```
_driverkit       26050   0.0  0.0 410598064   2256   ??  Ss    8:02PM   0:00.04 /Library/SystemExtensions/.../org.pqrs.Karabiner-DriverKit-VirtualHIDDevice.dext/org.pqrs.Karabiner-DriverKit-VirtualHIDDevice org.pqrs.Karabiner-DriverKit-VirtualHIDDevice 0x10002b929 org.pqrs.Karabiner-DriverKit-VirtualHIDDevice
root             25744   0.0  0.1 410756464   9872   ??  Ss    8:01PM   0:00.16 /Library/Application Support/org.pqrs/Karabiner-DriverKit-VirtualHIDDevice/Applications/Karabiner-VirtualHIDDevice-Daemon.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Daemon
```

</details>

<details>
<summary>Run Kanata at startup</summary>

Add a sudo rule in `/private/etc/sudoers.d/kanata`
(where `$USERNAME` is your username):

```bash
$USERNAME ALL=(ALL) NOPASSWD: /path/to/kanata/binary/kanata
```

To start Kanata at the beginning of the session, create this file:

```
~/Library/LaunchAgents/com.jtroo.kanata.plist
```

with the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.jtroo.kanata</string>

    <key>ProgramArguments</key>
    <array>
      <string>sudo</string>
      <string>/path/to/kanata/binary/kanata</string>
      <string>--cfg</string>
      <string>/path/to/kanata/config/file</string>
      <string>-n</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <true/>
  </dict>
</plist>
```

In the system settings, look for the *“Login Items”* menu and select the `sudo`
service in the *“Allow in the Background”* list.
Reload Kanata’s configuration by disabling and re-enabling this service.
</details>


Configuration
--------------------------------------------------------------------------------

Ækeynox-kanata is designed to ease your progression with mod-taps and layers.
Therefore, it does nothing by default; every feature must be enabled one by one.

The main `kanata.kbd` file should be enough for most users. All features are
(quickly) documented, just make sure that:
- existing sections are neither deleted nor reordered;
- each section has **one** and only one `(include)` statement.

Included `.kbd` files should be easy to customize if needed. Ækeynox-kanata is
proposed as a configurable keymap, but it’s easy to use it as a kickstarter if
you don’t want to follow the Arsenik spec.

> [!TIP]
> All `*custom*.kbd` files are git-ignored. Git users probably want to put
> their configuration in a `/custom.kbd` file, and pass it as an argument to
> Kanata:

```sh
./kanata --cfg custom.kbd
```


Why the name?
--------------------------------------------------------------------------------

Any name containing `key` and easy to search would’ve been a good fit,
but here’s Nox:

![My name is Nox and I approve this project.](img/nox.jpg)
