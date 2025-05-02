*ubuntu_on_2024_asus_oled_13*
# Installing Ubuntu 24.10 on the Asus Oled 13 (UX5304 CU7155U/32/1/13P)
![image](https://github.com/user-attachments/assets/10b3cb19-ec8c-4017-bcc4-52139d66189d)

When thinking of my next laptop in 2022, I wanted to continue with the Asus series, since my first Zenbook (a 2018 14" UX430U) was awesome. However, since their new models still used proprietary power plugs, I put a pin in that dream (who wants to keep around proprietary chargers when a few USB C chargers can be used by all the house's laptops!). Now it is 2025, and when I started my search for my next laptop after the sad passing of my Dell XPS, I decided to peek back at the Asus Zenbook 13, and behold: they finally sport standard USB C charging!

## When unboxing. 
When starting from a new Asus laptop that has Windows on it, take the time to update the BIOS and firmware using the MyAsus program (I know it is exciting to just raze windows to the ground to build a linux box, but it’s worth it to install those updates as that is the easiest way!). 

## Preparing the installation USB
**Download** the appropriate Ubuntu installable from https://ubuntu.com/download/desktop. I downloaded Ubuntu 24.10, particularly because it has a more recent kernel preinstalled (6.11 vs LTS Ubuntu 24.04’s 6.8). This means that more of your hardware has a chance to work (I have not tried 24.04 on this machine, so it also might work, and it is recommended, as it will have long term support. 

**Flashing the USB & Installing.** Follow Ubuntu’s [https://ubuntu.com/tutorials/install-ubuntu-desktop](https://ubuntu.com/tutorials/install-ubuntu-desktop) tutorial to write the downloaded ISO to USB.

### Asus-specific notes when loading the USB via the BIOS
1. First, plug in the USB flashed USB drive. 
1. To get into the BIOS, repeatedly press F2 right after powering up. There, **turn off Secure Boot** and change the Boot options to list your USB first.
1. When you restart, you will boot into your USB drive, where you can install Ubuntu.

## Post-installation 'fixes'

Here are the post install activities I performed:
1. **Update the system.** (after pressing the window button, run “Software Updater”).
1. **Look for additional drivers.** Then look for the “Software and Updates” app and go to the “Additional Drivers” tab to see if any proprietary drivers exist for download and usage (none so far on my laptop).
1. **Kernel.** Install latest kernel using mainline
    For most new laptops, there may be an issue with some of the new laptop hardware to interface well with the OS. 
    This is done through the kernel, and normally Ubuntu ships with an older kernel. I installed `Mainline`, which later will allow you to swap in the latest kernel for the existing one:
    ```sh
    > sudo add-apt-repository ppa:cappelikan/ppa 
    > sudo apt update -y 
    > sudo apt install -y mainline
    ```
    I then opened the mainline app and installed the latest kernel (6.12.3) with no problem.
    BTW, I tried installing Mainline kernels on the latest 2025 Dell XPS 13 and MSI Prestige 13 AI+ Evo,
    and both failed with many catastrophic warnings (I returned those laptops shortly after).
1. **Improving battery life.** Installing a battery control app that maintains a healthy battery (TLP; https://linrunner.de/tlp/index.html):
    ```sh
    # Add package Repository
    sudo add-apt-repository ppa:linrunner/tlp
    sudo apt update
    # Package Installation
    sudo apt install tlp tlp-rdw
    # The software will auto start after reboot
    ```

### An important bug fix: USB ports randomly become unresponsive
While the this Asus laptop is awesome (light and reasonably punchy), it has one nagging issue: every few weeks/months, all (USB-based) peripherals simply stop being responsive. An obviously simple fix is to restart the laptop, which is incredibly inefficient. I finally cobbled together a solution that works (took a long time, since I could not replicate the issue, so I had to wait for the issue to actually happen before creating and testing the solution). Here it is:





**IF YOU DON'T NEED TO DO ANYTHING ELSE (E.G. SCIENTIFIC COMPUTING), THEN YOU ARE DONE!** However, the following is a record of what I did post install to ensure my computer works smoothly.

## After installing


### Remapping my kensington trackball

I installed input-remapper (`sudo apt install input-remapper`) and mapped the top right trackball button to also register a left click when pressing. This way, the mouse can be used with both hands (the top right button is the left hand's mouse click, the top left button is the right hand's mouse click).












## Protip: Backing up from another computer? SCP don't CP (SKIP IF YOU HAVE A FRESH INSTALL)
If copying data from another computer, it is sometimes easier to connect both computers to the same wifi, install an SCP server on your new computer and scp the files over (copying from computer to hard disk and hard disk to computer took me 10x the time ... and I had a lot of trips on the way). Here is how you can set up the SSH server:
Set up the SSH server:

### Installing the SCP server

1. Install the client/server
    ```sh
    # On the old laptop:
    > sudo apt install openssh-client
    # On the new laptop:
    > sudo apt install openssh-server
    ```
1. Install tools to get the IP number etc.
    ```sh
    > sudo apt install net-tools
    ```
1. Get the local IP number of your computer
    ```sh
    > ifconfig -a
    ```
1. Modify `/etc/ssh/sshd_config` to contain "ListenAddress 192.X.X.X", where "192.X.X.X" is the internal ip address (they start with 192)
1. You can test the new ssh server config file with:
    ```sh
    > sudo sshd -t -f /etc/ssh/sshd_config
    ```
1. Then, start the service!
    ```
    > sudo systemctl start ssh.service
    ```
    *Troubleshooting:* if “ssh.service” is not found, then try  `> systemctl -l --type service --all|grep ssh` to identify the ssh server’s .service name.

### Using the SCP server
1. Test the ssh server from the client (old laptop) side by creating a dummy file and and sending it over
    ```sh
    > touch dummy_file.txt
    > ssh dummy_file.txt user@192.X.X.X:/home/user/
    ```
    If it works, you should see the file on your new laptop.
1. If the previous test works, use rsync to finally send in all directories that you care to transfer (here, we are to transfer the user’s Documents directory:
    ```sh
    > rsync -au /home/user/Documents user@192.168.XX.XX:/home/user/
    ```


