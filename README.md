# Droidian for the Xiaomi Mi A2
A collection of tips on how to install and use Droidian on the Xiaomi Mi A2 (jasmine_sprout)

### Requirements
- A computer with fastboot and adb access to the phone
- A USB 2.0 port/hub with actual USB 2.0 controller is recommended (fastboot on USB 3 can be problematic with Mi A2)
- Unlocked bootloader
- Backup all your data, as **your phone will be WIPED**
- It is also recommended to take a note of your `Acces Point Name` in Android `Settings` before starting the procedure (you will need it for setting up mobile data)

## Installation
### 0. Download files
- [Droidian rootfs](https://github.com/droidian-images/rootfs-api28gsi-all/releases/download/droidian%2Fbullseye%2F22/droidian-rootfs-api28gsi-arm64_20210531.zip) (specific version needed)
- [Droidian devtools](https://github.com/droidian-images/rootfs-api28gsi-all/releases/download/droidian%2Fbullseye%2F22/droidian-devtools-amd64_20210531.zip) (specific version needed)
- [Boot image](https://github.com/Droidian-Mi-A2-6X/linux-android-xiaomi-wayne-jasmine/releases/download/20210913/boot.img)
- [Android 9 Pie Firmware](https://github.com/ubports-xiaomi-sdm660/artifacts/releases/download/v0.1/jasmine_sprout_stock_android9.zip)
- Latest [TWRP image](https://dl.twrp.me/jasmine_sprout/)

### 1. Boot into TWRP 
- Boot to fastboot mode by pressing the `Vol-` and `Power` buttons until the phone vibrates
- Check that the phone is recognized by running `fastboot devices`
- Boot to TWRP by running `fastboot boot twrp-*-jasmine.img`

### 2. Install Droidian in TWRP
TWRP:
- Go to `Wipe` and `Format data` (type yes)
- IMPORTANT: **Do the 'Boot into TWRP' step again**

PC:
- Connect the phone via USB
- The internal storage is now available over MTP from the PC
- Copy the downloaded files to the internal storage of the phone

TWRP:
- Go to the `Reboot` menu and see which slot is active
- If it says `Slot A`, then select `Slot B` to be the active slot, and boot TWRP again


**With `Slot B` as active:**
- Install zip file: `jasmine_sprout_stock_android9.zip`
- Now switch back to `Slot A` and boot TWRP again

**With `Slot A` as active:**
- Install image: `boot.img` to `Boot` partition
- Install zip file: `droidian-rootfs-api28gsi_arm64_20210531.zip` 
- Install zip file: `droidian-devtools_arm64_20210531.zip`
- Go back to the main menu and reboot to `System` (TWRP might complain that there is no OS installed, but that's fine)
- The first boot may take longer, and at least one spontaneous reboot is expected during the process
- If all goes well, your phone will boot to the Droidian lock screen, the unlock code is `1234`
- Installation is complete
- Flashing the `devtools` zip enabled `SSH` over USB. To use it, connect your phone to your computer and type:
```
ssh droidian@10.15.19.82
```
The password is `1234` (on Windows, you may need [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/))
- Apply device-specific tweaks by running:

```
wget https://github.com/Droidian-Mi-A2-6X/droidian-tweaks/raw/master/setup.sh && chmod +x setup.sh && ./setup.sh
```

- You can also upgrade to the latest `bookworm` version of Droidian **after** doing all available updates from the `Software` application. Run this on the device itself:

```
sudo apt install droidian-upgrade-bookworm
sudo apt update
sudo apt upgrade
sudo apt update
sudo apt dist-upgrade
sudo apt clean
systemctl reboot
```

## Notes

### Mobile data
Mobile data should work, but it needs some manual tweaking. The good part is that the configuration is persistent, so you need to do this only once.
- Open the `Settings` app and tap `Mobile`, then tap `Access Point Names`
- Tap the `+` in the top right corner of the pop-up
- Fill out the textboxes based on your settings in Android. The `Name` can be anything, e. g., the name of your carrier. The `APN` is likely some sort of an URL, like `internet.carrier.net` or something similar. If you didn't see a `Username` and `Password` in Android, you can skip these. 

In most cases, rebooting your phone or turning mobile data on and off a few times is enough.
Otherwise we'll have to give the same settings to [oFono](https://en.wikipedia.org/wiki/OFono) as well.
- Open the `King's Cross` terminal app or use `SSH` and type `sudo apt install ofono-scripts && cd /usr/share/ofono/scripts`
- To check your current setup, type `./list-contexts`, look for the first context, called `[ /ril_1/context1 ]`
- Notice that context has `Name`, `Type`, `AccessPointName`, `Username`, and `Password` attributes among others, we'll need to change these to match the valid settings taken from Android. 
- Now deactivate this context to be able to make changes to it: `./deactivate-context 1`
- And now make your changes like this: `./set-context-property 1 $ATTRIBUTE $VALUE`, where the `1` is for `[ /ril_1/context1 ]`, the `$ATTRIBUTE` is one of the attributes listed above, and the `$VALUE` will be the value you need to enter. make sure that the `Name` and `AccessPointName` match the `Name` and `APN` you entered in the settings menu. If an attribute needs to be empty, set it to `""`.
- When you make your changes, you can use `./list-contexts` to see them.
- When everything looks good, activate the updated context: `./activate-context 1`
- If all went well, by running`./list-contexts` again you should see that the `Active` attribute is now `1`, and under the `Settings` attribute you can see the network parameters (Address, Netmask, Gateway, etc.) assigned by your carrier. If you get an error, try rebooting the phone and turning mobile data on and off a few times, and try running the `/usr/share/ofono/scripts/activate-context 1` script again.
- In the end, mobile data should be working now.

- Pro tip: once you configured everything, you can type `history` to see your previous commands. Save them to a script that does all of this for you next time.


## Credit
[Shouko](https://いらっしゃい.みんな/)

[Droidian](http://droidian.org/)

[UBports](https://ubuntu-touch.io/)



For further assistance, visit [Shouko's Lab](https://t.me/shoukolab) and the [Droidian](https://t.me/droidianlinux) Telegram groups.
