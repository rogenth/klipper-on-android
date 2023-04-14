# Using Android to run Klipper, Moonraker, Mainsail/Fuidd, and KlipperScreen with Termux
## Alternative Version with Linux Deploy: https://github.com/gaifeng8864/klipper-on-android (uses fluidd and simplifies the script creation)
### Disclaimer: this is an ongoing work, still some changes pending. I am not responsible if you brick your device or cause a fire. Make sure to decide to dispose of the battery of your phone or not (See https://www.youtube.com/watch?v=7f8SliNGeDM&t)
#### Fork from https://github.com/d4rk50ul1/klipper-on-android
#### Thanks to https://github.com/gaifeng8864/klipper-on-android
#### Original work by @RyanEwen (https://gist.github.com/RyanEwen/ae81fc48ad00397f1026915f0e6beed9)
#### Current Setup: Ender 3 Pro (SKR MINI E3 V2.0) with Klipper Firmware + Oneplus X (Lineage 18.1/Magisk) / Pixel 5 (Stock/Magisk) running Klipper + Moonraker+Fluidd + Klipperscreen

I installed Klipper succesfully on my Pixel 5 with the tutorial from gaifeng8864 but I wasn't so lucky with my Oneplus X as Linux Deploy couldn't install Debian, so I followed the original tutorial and added a debian installation through Termux.

## Requirements
- A rooted Android device (see Magisk) with the following installed:
  - Termux app: https://f-droid.org/packages/com.termux/
  - XServer-XSDL-1.20.51 app: https://sourceforge.net/projects/libsdl-android/files/apk/XServer-XSDL/
  - Octo4a app: https://github.com/feelfreelinux/octo4a
- An OTG+Charge cable up and running for the same device ( please check this video for reference: https://www.youtube.com/watch?v=8afFKyIbky0)
- An already flashed printer using Klipper firmware. 
  - For reference : https://3dprintbeginner.com/how-to-install-klipper-on-sidewinder-x2/
- Init scripts for Klipper and Moonraker (scripts folder).
- XTerm script for KlipperScreen (scripts folder).
 
## Setup Instructions
- Initialize Termux, acquire sudo, choose a password and install the Debian container:
  ```bash
  pkg update
  pkg upgrade
  pkg install wget openssl-tool termux-auth proot tsu -y && hash -r && wget https://raw.githubusercontent.com/EXALAB/AnLinux-Resources/master/Scripts/Installer/Debian/debian.sh && bash debian.sh
  su
  exit
  sudo
  passwd
  ```
- Boot Debian
  ```bash
  sudo ./start-debian.sh
  ```

- Add your username (in this case `print3D`), add a root password and upgrade packages:
  ```bash
  apt update
  apt upgrade
  apt install sudo vim wget
  adduser print3D --force-badname
  passwd
  ```
- Get sudo access:
  ```bash
  vi /etc/sudoers #Check vim tutorial, :x! for saving, Add this line: print3D ALL=(ALL) ALL under user
  usermod -aG sudo print3D
  ```
  
- Test sudo, add a password and log into `print3D`:
  ```bash
  sudo su
  su print3D
  ```
  
  Now on logged into `print3D`,  fix permissions and test sudo
  ```bash
  sudo usermod -a -G aid_inet,aid_net_raw root #Permissions Fix just in case
  ```
    
- Get back into root: install and start SSH:
  ```bash
  sudo su
  cd ~
  wget https://raw.githubusercontent.com/EXALAB/AnLinux-Resources/master/Scripts/SSH/Apt/ssh-apt.sh --no-check-certificate && bash ssh-apt.sh
  #Start SSH
  /etc/init.d/ssh start
  ```
- From there you can continue on your phone and pretty much follow the instructions as in Linux Deploy versions.
 
- SSH into the container (from your PC Terminal or other device `ssh -p 22 print3D@YOUR_DEVICE_IP`):
  ```bash
  su print3D #just if continuing from Termux
  sudo apt install git
  git clone https://github.com/th33xitus/kiauh.git
  ```
 
- Install Klipper, Moonraker, Mainsail (or Fluidd), and KlipperScreen:
  ```bash 
  kiauh/kiauh.sh
  ```
  *Note: KlipperScreen in particular will take a very long time (tens of minutes).*  
- Find your printer's serial device for use in Klipper's `printer.cfg`:  
  It will likely be `/dev/ttyACM0` or `/dev/ttyUSB0`. Check if either of those appear/disappear under `/dev/` when plugging/unplugging your printer (Check using 'ls -al /dev/').  
  
  If you cannot find your printer in `/dev/`, then you can check Octo4a app which includes a custom implementation of the CH34x driver. IMPORTANT: You don't need to run OctoPrint within it so once in the main screen of the app just stop it if it's running. To do this:   
    - Install Octo4a from https://github.com/feelfreelinux/octo4a/releases
    - Run Octo4a and let it install OctoPrint (optionally tap the Stop button once it's done installing).
    - Make sure Octo4a sees your printer (it will be listed with a checked-box next to it).
      - There will be a prompt in your android device asking for permission to connect to your printer if detected.
    - `/data/data/com.octo4a/files/serialpipe` is the serial port you need to use in your `printer.cfg`
    - Modify later `start-klipper-container` from the Termux root directory and add `command+=" -b /data/data/com.octo4a/files:/data/data/com.octo4a/files"` around the `#uncomment` section in order to mount the implementation of Octo4a into the Debian/Klipper container.
    
- Make the serial device accessible to Klipper (on both Termux and Debian Container):
    ```bash
    #from Termux and Debian:
    sudo chmod 777 /dev/ttyACM0
    # or
    sudo chmod 777 /dev/ttyUSB0
    # or
    sudo chmod 777 /data/data/com.octo4a/files/serialpipe
    ```
- Use XFTP or command line to delete the config files:
  ```bash
  cd /home/print3D/printer_data/config/
  ```
  
- Download the configuration files:
  ```bash
  sudo wget -P /home/print3D/printer_data/config/ https://raw.githubusercontent.com/gaifeng8864/klipper-on-android/main/fluidd.cfg
  sudo wget -P /home/print3D/printer_data/config/ https://raw.githubusercontent.com/gaifeng8864/klipper-on-android/main/homing_override.cfg
  sudo wget -P /home/print3D/printer_data/config/ https://raw.githubusercontent.com/gaifeng8864/klipper-on-android/main/moonraker.conf
  sudo wget -P /home/print3D/printer_data/config/ https://raw.githubusercontent.com/gaifeng8864/klipper-on-android/main/printer.cfg
  ```
  
  - Replace `printer.cfg` by the parameters of your printer now or later, remember to add `/dev/ttyACM0`, `/dev/ttyUSB0` or `/data/data/com.octo4a/files/serialpipe` under `[mcu]` on the configuration file.
  
- Install the init and xterm scripts from this gist:  
  ```bash
  sudo wget -O /etc/default/klipper https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/etc_default_klipper
  sudo wget -O /etc/init.d/klipper https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/etc_init.d_klipper
  sudo wget -O /etc/default/moonraker https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/etc_default_moonraker
  sudo wget -O /etc/init.d/moonraker https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/etc_init.d_moonraker
  sudo wget -O /usr/local/bin/xterm https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/usr_local_bin_xterm
  
  sudo chmod +x /etc/init.d/klipper 
  sudo chmod +x /etc/init.d/moonraker 
  sudo chmod +x /usr/local/bin/xterm
  
  sudo update-rc.d klipper defaults
  sudo update-rc.d moonraker defaults
  ```
- Copy the boot script on Termux (not Debian):
  ```bash
  mkdir /data/data/com.termux/files/home/.termux/boot
  
  sudo wget -O /data/data/com.termux/files/home/.termux/boot/start-termux https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/start-termux
  sudo wget -O /data/data/com.termux/files/home/start-klipper-container https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/start-klipper-container
  sudo wget -O /data/data/com.termux/files/home/debian-fs/root/start-klipper https://raw.githubusercontent.com/rogenth/klipper-on-android/main/scripts/start-klipper
  
  sudo chmod +x /data/data/com.termux/files/home/.termux/boot/start-termux
  sudo chmod +x /data/data/com.termux/files/home/start-klipper-container
  sudo chmod +x /data/data/com.termux/files/home/debian-fs/root/start-klipper
  ```
  
- Stop the Debian/Klipper container.
- Start XServer XSDL.
    - One time setup: 
        - Tap 'Change Device Configuration'
        - Change Mouse Emulation Mode to Desktop, No Emulation
- Start the Debian/Klipper container `./start-klipper-container`.
- KlipperScreen should appear in XServer XSDL and Mainsail and/or Fluidd should be accesible using your Android device's IP address in a browser.

- Then you can set up Termux Boot in order to autostart the container and klipper screen. I added a delay, but I haven't test it yet. If you want to manually start the container, use `./start-klipper-container` on Termux, just make sure to also start manually the XServer app.

## Misc
You can start/stop Klipper and Moonraker manually by using the `service` command (eg: `sudo service klipper start`).  
Logs can be found in `/home/android/klipper_logs`.

## Telegram Bot
You can find the instructions how to setup the Telegram Bot [here](https://github.com/rogenth/klipper-on-android/blob/main/telegram_instructions.md). You need to then modify `start-klipper` in the container and uncomment the Telegram start section: `/etc/init.d/telegram start`.

## Troubleshooting (ongoing section based on comments)
- After a phone reboot you may need to grant access again to the serial pipe, through: `sudo chmod 777 /data/data/com.octo4a/files/serialpipe`. Make sure that Octo4a recognizes the phone. You may also try different drivers through the app.
- There might be the case that when accessing Mainsail through Browser, you get an error message and no connection to moonraker: mainsail Permission denied while connecting to upstream in `klipper_logs/mainsail_error.log`. To fix this you must change the file `/etc/nginx/nginx.conf`, change `user www-data;` to `user android;` 
- If anyone is having network issues in the container as a non root user after a few minutes, you need to disable deep sleep/idle. You can do that by using this command in a shell (termux or adb doesn't matter): `dumpsys deviceidle disable`. You may also need this app: [Wake Lock - CPU Awake] (https://play.google.com/store/apps/details?id=com.dambara.wakelocker)
- As per [ZerGo0](https://gist.github.com/ZerGo0) comments - The latest moonraker update seems to break a few things. There are some changes about the directory and file locations, but you can just sym link the new directories to the old ones using the included script: 
  ```bash
  sudo /etc/init.d/moonraker stop
  cd ~/moonraker
  scripts/data-path-fix.sh
  sudo /etc/init.d/moonraker start
  ```
  You can obviously also do this manually.
  You also have to add the following section to your `moonraker.conf`:
  ```bash
  [machine]
  validate_service: False
  validate_config: False
  provider: none
  ```
  There are a few more deprecated settings now, check the notifications or moonraker docs to find out what you need to remove.
 
- When checking the OTG+Charge Cable, each phone "recognizes" a different resistor, my recommendation is that once you build your cable, try not to solder the resistor directly to the 5 pin plug. Instead, use a breadboard temporary and test with different resistors. My approach was the following:
    - Without connecting anything to the microusb port (not tested with type-c) Open Octo4a app in your mobile
    - connect the printer to the usb modified cable (assuming that you have already builded one ;) ) 
    - connect the charger to the microusb male port
    - connect the modified cable to the mobile phone.
        - If the phone detects charge and Octo4a shows a popup requesting access to a serial device, you're done! the resistor is working. 
        - If the phone detects only charge and Octo4a doesn't show a popup requesting serial access, you must remove the resistor from the breadboard and try with a different one. 
        - Repeat this process until you figure it out.
        - My case i bought a resistor kit with values from 1k Ohm to 1M (about 20 possibilities). I changed them 1 by 1 until it worked. 
        - Just as a reference in case anyone has the same device. I used a Lenovo tab M8 (TB 8505F) and it worked with a 10k Ohm resistor!
         
