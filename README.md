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

<!-- skip the latest Kanata release (1.12.0)
[dl-kanata]:   https://github.com/jtroo/kanata/releases/latest
-->
[dl-kanata]:   https://github.com/jtroo/kanata/releases/tag/v1.11.0
[dl-aekeynox]: https://github.com/OneDeadKey/kanata-config-aekeynox/releases/latest


Usage
--------------------------------------------------------------------------------

- Get Ækeynox:
  - download the [latest release][dl-aekeynox],
    or check out this repository with Git
  - [configure `kanata.kbd`](#configuration): Ækeynox-kanata does nothing by
    default, every feature must be enabled on an opt-in basis
- Get Kanata:
  - download a [pre-built executable][dl-kanata]
  - unzip it into the Ækeynox folder
- Run Kanata.

See OS-specific instructions below.


### Windows

There are several Kanata variants for Windows. We recommend using the
`gui_winIOv2` variant: put it in the Ækeynox folder, start it
(double-click), and Kanata will run in the background. It can be reloaded or
exited with a right-click on its systray icon.

> [!WARNING]
> Due to a regression in Kanata 1.12.0 ([issue #2115]), Windows users should
> skip this version. A [workaround] can be used with the next release, by
> setting `windows-sync-keystates` to `none` in `defsrc/settings.kbd`:

```lisp
  ;; Works around a regression introduced by Kanata-winIOv2 1.12.0,
  ;; but requires Kanata 1.13.0 or newer.
  ;; https://github.com/jtroo/kanata/issues/2115
  windows-sync-keystates none
```

[issue #2115]: https://github.com/jtroo/kanata/issues/2115
[workaround]:  https://github.com/jtroo/kanata/issues/2122

### Linux

Kanata is available as an official package in most Linux distributions.

For Kanata to start automatically after login, follow the two sections below.

<details>
<summary>Run Kanata without <code>sudo</code></summary>

Kanata has to access the `input` and `uinput` subsystems, which requires proper
authorizations. To avoid running Kanata with `sudo`, your user must be added to
these groups:

```bash
# Create the `uinput` group if it doesn't exist
sudo groupadd --system uinput

# Add your user to the `input` and `uinput` groups
sudo usermod -aG input $USER
sudo usermod -aG uinput $USER
```

You’ll have to log out and relog in (or even reboot) for these changes to apply.
Make sure the `input` and `uinput` groups appear in the output of the `groups`
command.

Now create a `udev` rule to give `uinput` the required permissions:

```bash
# Create the udev rule for `uinput`
sudo tee /etc/udev/rules.d/99-input.rules > /dev/null <<EOF
KERNEL=="uinput", MODE="0660", GROUP="uinput", OPTIONS+="static_node=uinput"
EOF

# Reload udev rules
sudo udevadm control --reload && udevadm trigger
```

</details>

<details>
<summary>Make a user-side <code>systemd</code> service for Kanata</summary>

Now that Kanata can run without `sudo` permissions, we can use a systemd
service to run it as a daemon right after logging in.

Create this file:

```
~/.config/systemd/user/kanata.service
```

with the following content:

```properties
[Unit]
Description=Kanata keyboard remapper
Documentation=https://github.com/jtroo/kanata

[Service]
Environment=PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/bin
Type=simple
ExecStart=/path/to/kanata --cfg /path/to/kanata/config.kbd
Restart=no

[Install]
WantedBy=default.target
```

Replace `/path/to/kanata` and `/path/to/kanata/config.kbd` by the proper paths
of the Kanata executable and the Ækeynox configuration file, respectively.

Now enable `kanata.service` and make sure it’s running:

```bash
systemctl --user daemon-reload
systemctl --user enable kanata.service  # auto-start Kanata when user logs in
systemctl --user start  kanata.service  # start Kanata manually
systemctl --user status kanata.service  # check that the service is running
```

</details>

[See the Kanata documentation](linux) for further information.

[linux]: https://github.com/jtroo/kanata/blob/main/docs/setup-linux.md

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

> [!TIP]
> When layer-taps are activated, the whole configuration can be live-reloaded
> with <kbd>Space</kbd>+<kbd>Esc</kbd>.

<details>
<summary>Bonus: restrict Kanata to the laptop keyboard</summary>

On macOS and Linux, starting with version v1.10.0, Kanata can be configured to
ignore external keyboards.

First, find your laptop keyboard with `kanata --list`. The output looks like
this:

```
Available keyboard devices:
===========================
Found 2 keyboard device(s):

  1. "ThinkPad Extra Buttons"
     Path: /dev/input/event9
     Vendor ID: 6058 (0x17AA), Product ID: 20564 (0x5054)

  2. "AT Translated Set 2 keyboard"
     Path: /dev/input/event3
     Vendor ID: 1 (0x0001), Product ID: 1 (0x0001)

Configuration example:
  (defcfg
    linux-dev-names-include (
      "ThinkPad Extra Buttons"
      "AT Translated Set 2 keyboard"
    )
  )
```

Then copy the `linux-dev-names-include` section (or `macos-dev-names-include` on
Mac) into the `defcfg` section in `defsrc/settings.kbd`.

More information in Kanata’s user guide:
[linux-dev-names-include](https://jtroo.github.io/config.html#linux-only-linux-dev-names-include),
[macos-dev-names-include](https://jtroo.github.io/config.html#macos-only-macos-dev-names-include).

</details>


Keyboard Layout
--------------------------------------------------------------------------------

The `Symbols` layer, as well as the keyboard shortcuts in the `Navigation`
layer, depend on the keyboard layout: QWERTY, AZERTY, QWERTZ… This keyboard
layout must be selected accordingly in the last configuration section.

If your keyboard layout isn’t already supported, creating the corresponding
aliases file should be straightforward. Pull requests are welcome, and our
maintainers can assist you if needed.

We recommend using the default `Symbols` layer (`symbols.kbd`) for any keyboard
layout; however, there are two edge cases where you could prefer an
<kbd>AltGr</kbd>/<kbd>Option</kbd> layer (`symbols_altgr`):

- when your keyboard layout already has an optimized `Symbols` layer (e.g.
  Ergo‑L or QWERTY-Lafayette, which already have Ækeynox’s Symbols layer,
  or Neo, which has its own Symbols layer);
- when your keyboard layout requires <kbd>AltGr</kbd> or <kbd>Option</kbd> for
  common text input (e.g. Bépo).

(Note for European users: don’t worry about the € sign, it’s available on
<kbd>Nav</kbd><kbd>'</kbd> or <kbd>Sym</kbd><kbd>'</kbd>.)


Troubleshooting
--------------------------------------------------------------------------------

Some combinations of three keys might not work on a standard keyboard, due to
[ghosting], which is a hardware problem that Kanata cannot fix.

Here’s a common example on ThinkPad, with the `NumRow` layer:

- press <kbd>Sym</kbd>
- press <kbd>Shift</kbd> (=> brings the `NumRow` layer)
- tap <kbd>G</kbd>

Expected result: types a `5`.<br>
Actual result: nothing happens.

The workaround consists in releasing a key:

- press <kbd>Sym</kbd>
- press <kbd>Shift</kbd> (=> brings the `NumRow` layer)
- *release <kbd>Sym</kbd>* (=> `NumRow` is still active)
- tap <kbd>G</kbd>

[ghosting]: https://en.wikipedia.org/wiki/Key_rollover#Ghosting


Why the name?
--------------------------------------------------------------------------------

Any name containing `key` and easy to search would’ve been a good fit,
but here’s Nox:

![My name is Nox and I approve this project.](img/nox.jpg)
