# PlexAmp-installer for Raspberry Pi with ARMv8, 64-bit HW.

For more information and hardware used, see here:<br /> https://github.com/odinb/bash-plexamp-installer/wiki

Assumes 64-bit capable Raspberry Pi HW and Pi OS that is 64-bit.

Currently installs/upgrades to: Plexamp-Linux-headless-v4.5.3 and NodeJS v16.

## Burning the image.
Burn the OS-image to the Micro-SD card using Raspberry pi imager, etcher (or app of your choice).

This will currently only work with 64-bit capable Raspberry Pi and Pi OS that is 64-bit.
Images for Bullseye can be found here:
https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit

For the below 2 tasks (user/password and SSH-access) the Raspberry Pi imager can be used as an alternative. The advanced option (cogwheel) will allow you to add both options before writing to SD-card.
https://www.raspberrypi.com/software/

[The main screen of Pi-imager:](https://www.raspberrypi.com/software/)
<br /> <img src="https://github.com/odinb/bash-plexamp-installer/blob/main/Pi-imager_Main.png" width="450">

[The advanced screen of Pi-imager:](https://www.raspberrypi.com/software/)
<br /> <img src="https://github.com/odinb/bash-plexamp-installer/blob/main/Pi-imager_Advanced.png" width="400">

### Create initial user
For security reasons, as of the spring 2022 release, Raspbian OS no longer has a default pi user wit assigned password.
To set up a user on first boot on headless, create a file called userconf or userconf.txt in the boot partition of the SD card.
This file should contain a single line of text, consisting of username:encrypted-password – so your desired username, followed immediately by a colon, followed immediately by an encrypted representation of the password you want to use.

To generate the encrypted password, the easiest way is to use OpenSSL on a Raspberry Pi that is already running (or most any linux you have running) – open a terminal window and enter:
```echo ‘mypassword’ | openssl passwd -6 -stdin```

This will produce what looks like a string of random characters, which is actually an encrypted version of the supplied password.<br />

### Enable SSH.
How to enable SSH:<br />
For security reasons, as of the November 2016 release, Raspbian OS has the SSH server disabled by default. You will have to enable it manually.
1. Mount your SD card on your computer.
2. Create or copy a file called ssh in /boot. 
On most Linux-distros, after re-mount of micro-SD-card, run: ```touch /media/$user/boot/ssh```.
On MacOS, after re-mount of micro-SD-card, run: ```touch /Volumes/boot/ssh```.

Then unmount and insert card into Raspberry Pi and boot it.

SSH access on "DietPi OS" as user: dietpi/dietpi and as root: root/dietpi<br />
NOTE!!! DietPi is best-effort, and might not work.

After SSH-ing to the SBC, on the "Raspberry Pi OS", change to root (```sudo -i```) and run script with:

```bash <(wget -qO- https://raw.githubusercontent.com/odinb/bash-plexamp-installer/main/install_Plexamp_pi.sh)```

### Post-install and post-reboot tasks
Note !! Only needed if fresh install, not if upgrading. Tokens are preserved during upgrade.
After reboot, as your regular user please run the command:

```node /home/USER/plexamp/js/index.js```
where USER is your user, normally pi.
Now, go to the URL provided in response, and enter the claim token at prompt.
Please give the player a name at prompt (can be changed via Web-GUI later).
At this point, Plexamp is now signed in and ready, but not running!

Now either start Plexamp manually using: ```node /home/"$USER"/plexamp/js/index.js```
or enable the service and then start the Plexamp service.
If process is running, hit ctrl+c to stop process, then enter:
```sudo systemctl enable plexamp.service && sudo systemctl restart plexamp.service```

Once done, the web-GUI should be available on the ip-of-plexamp-pi:32500 from a browser.
On that GUI you will be asked to login to your Plex-account for security-reasons, and then choose a librabry where to fetch/stream music from.

Wherever possible, you should choose output device via the script (gets set in "/boot/config.txt") when installing, to make it default system-wide.
If you have audio-problems, or want to choose output after install, go to the web-GUI.
Here the menu is found via: Settings (cogwheel lower right corner) >> Playback >> Audio output >> Audio Device.
As an example, if you have chosen the “ Digi/Digi+“ option during install in the script, pick “Default” if the card is not showing, then reboot the pi. Now the card will show up in the list, and you can choose it!

Now play some music! Or control it from any other instance of Plexamp.


### Re-running the script

The script can be re-run to fix configuration/setup errors, just say no/bypass the sections you do not want to re-run!

If there is a new version, and script has been updated, you can upgrade by re-running the script, and reboot.

For hostname-change, please make sure to reboot in-between, or you might face issues.

### Q & A

Q:
The RPi will show up in my cast list. However the moment I select a song to play, it either keeps loading forever (black screen with loading circle) or it flips back to the album overview without starting the selected song. This is the case for all file types tried.

A:
Go into the plexamp settings (via the web UI) and select the right audio output device. Rebooting the RPi a second time sometimes also helps (or restarting/veirfying the PlexAmp service), this of course will only help if correct audio device is already chosen.
(```sudo systemctl enable plexamp.service && sudo systemctl restart plexamp.service```)

======

Q:
When starting Plexamp (4.3.0), I get this:
Starting Plexamp 4.3.0 DEVICE: No provider for source 80df67e4336a1a1c9911d2b4b6b4133c9f7d4cab
Any idea what this is about?

A:
You need to choose your source, i.e. Plex server or TIDAL other. This can be done from the GUI.

======

Q:
I got an error during the first Plexamp install and start up:

xxx@PlexampPi:~ $ node /home/xxx/plexamp/js/index.js
Starting Plexamp 4.3.0
Please visit https://plex.tv/claim and enter the claim token: claim-xxxxxxxxxxxxxxxxxxxx
Please give the player a name (e.g. Bedroom, Kitchen): Marantz
DEVICE: Error loading cloud players from plex.tv HTTP status 403

A:
That is not a bad thing per se, it just means you didn’t link any “cloud players”.
See also:
https://forums.plex.tv/t/plexamp-headless-not-playing-music/809078/21

======

Q:
After upgrading to 4.4.0 (or later), I am getting "ALSA lib pcm_dmix.c:1075:(snd_pcm_dmix_open) unable to open slave".

A:
Go to the GUI and re-select your Audio device under Settings (cogwheel lower right corner) >> Playback >> Audio output >> Audio Device. Reboot if it does not fix it, and verify setting is still there. If your device is missing, and reboot does not fix it, powercycle the RPi and try again.

======

Q:
My HifiBerry card (or clone) is not detected after installation and reboot with ``` aplay -l ``` command, what could be wrong?

A1: Sometimes the card is not detected after upgrade, try doing a hard reboot, i.e. pull the power-cable for 10 seconds and then re-insert power to boot. Card should now be detected. If needed, go to: Settings (cogwheel lower right corner) >> Playback >> Audio output >> Audio Device and re-select your audio-card.

A2:
On your Raspberry Pi, run the following command: ``` cat /proc/device-tree/model ```. If it says “Raspberry Pi 4 Model B Rev 1.5”, the HifiBerry card might not work as expected.
The new Pi release uses a new power management circuit that doesn’t ramp up the voltages as clean as all previous versions did. This results in an inconsistent state of the circuit and the HifiBerry card isn’t detected by the Pi anymore.

Note that this affects only the Digi+ Pro and Digi2 Pro (and some clones of these cards).

For more information:
https://www.hifiberry.com/blog/digi2-pro-raspberry-pi-4-1-5-incompatibilities/
https://www.hifiberry.com/blog/compatibility-issues-of-the-digi2-pro-and-raspberry-pi-4-rev-1-5/
https://forums.raspberrypi.com/viewtopic.php?t=329299


======
