*ubuntu_on_2024_asus_oled_13*
# Installing Ubuntu 24.10 and 24.04 on the Asus Oled 13 (UX5304 CU7155U/32/1/13P)
![image](https://github.com/user-attachments/assets/10b3cb19-ec8c-4017-bcc4-52139d66189d)

This is my journey towards installing [Ubuntu 24.10 (Oracular Oriole)](https://releases.ubuntu.com/oracular/) and [Ubuntu 24.04 LTS (Noble Numbat)](https://releases.ubuntu.com/noble/) on the awesomely light and reasonably fast [2024 Asus Zenbook S13 OLED](https://www.asus.com/us/laptops/for-home/zenbook/asus-zenbook-s-13-oled-ux5304/). I recommend 24.04 LTS as it is a Long Term Service version that is still under service by Ubuntu, and will be until 2034 with a free Pro account (i.e., security updates and patches will be up to date up until then), while 24.10 is not under service anymore (this non-LTS versions had less than a year of service). The first part discusses installation for general usage, and, as a reference for future me, I then discuss how to set up the laptop for scientific computing.

## Right after unboxing: use restraint! 
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
1. **Update the system.** As with all new installs: `> sudo apt update; sudo apt upgrade`; also, run “Software Updater”.
1. **Look for additional drivers.** Then look for the “Software and Updates” app and go to the “Additional Drivers” tab to see if any proprietary drivers exist for download and usage (none so far on my laptop).
1. **Kernel. Install latest kernel using mainline.**  ***NOTE: KERNEL UPDATES CAN BE DANGEROUS, SO, ONLY DO THIS IF YOUR HARDWARE IS NOT SUPPORTED BY THE NATIVE KERNEL.***
    For most new laptops, there may be an issue with some of the new laptop hardware to interface well with the OS (as of August 2025, both Ubuntu 24.10 and 24.04 have no issues with the hardware; so you can skip this step). 
    This is done through the kernel, and normally Ubuntu ships with an older kernel. I installed `Mainline`, which later will allow you to swap in the latest kernel for the existing one:
    ```sh
    sudo add-apt-repository ppa:cappelikan/ppa 
    sudo apt update -y 
    sudo apt install -y mainline
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

### An important bug "fix": USB ports randomly become unresponsive
*Postscript: this bug may have been fixed, as I have not yet experienced it over the last few months.* While the this Asus laptop is awesome (light and reasonably punchy), it has one nagging issue: every few weeks/months, all (USB-based) peripherals simply stop being responsive. I have seen this issue happening in both Windows and Linux. An obviously simple fix is to restart the laptop, which is incredibly inefficient. I finally cobbled together a solution that works (took a long time, since I could not replicate the issue, so I had to wait for the issue to actually happen before creating and testing the solution). Here it is:
1. Create a script that resets/restarts all USB ports (say that its location is `~/reset_usbs.sh`):
    ```sh
    #!/usr/bin/env bash
    for DEVICE in $(lspci -Dm | grep "USB controller" | cut -f1 -d' '); do
            echo $DEVICE
            echo $DEVICE > /sys/bus/pci/drivers/xhci_hcd/unbind
            echo $DEVICE > /sys/bus/pci/drivers/xhci_hcd/bind
    done
    ```
1. Make it executable: `chmod +x ~/reset_usbs.sh`
1. Run it: `cd ~/; ./reset_usbs.sh`
1. Bonus: register this script as an app, so you can run it by pressing the windows button and then plugging in the name of the "app". This is done by creating the following `reset_usbs.deskdop` file in `~/.local/share/applications/`:
    ```sh
    [Desktop Entry]
    Encoding=UTF-8
    Version=1.0
    Type=Application
    Terminal=false
    Exec=pkexec env DISPLAY=$DISPLAY XAUTHORITY=$XAUTHORITY ~/reset_usbs.sh
    Name=Reset USBs
    ```
    The scary command at the beginning of the "Exec=" command initiates a sudo environment via the GUI (vs via the terminal). 
    Now, typing in "reset usbs" in the OS should activate the resetting process.


**IF YOU DON'T NEED TO DO ANYTHING ELSE (E.G. SCIENTIFIC COMPUTING), THEN YOU ARE DONE!** However, the following is a record of what I did post install to ensure my computer works smoothly.

## Post-install

### Remapping my kensington trackball

I installed input-remapper([Github link](https://github.com/sezanzeb/input-remapper); install using: `sudo apt install input-remapper`) and mapped the top right trackball button to also register a left click when pressing. This way, the mouse can be used with both hands (the top right button is the left hand's mouse click, the top left button is the right hand's mouse click).

## Finally, the fun stuff: installs!

1. **Some pre-recs and other common installs:**
    1. VIM:    
        ```sh
        > sudo apt install vim
        ```
    1. To use flatpaks, you need to install FUSE (https://github.com/AppImage/AppImageKit/wiki/FUSE):
        ```sh
        > sudo add-apt-repository universe
        > sudo apt install libfuse2t64
        ```
    1. The C# version of godot needs a .Net runtime. To do that, follow the instructions on https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install 
        I used following commands:
        ```sh
        > sudo apt-get update && sudo apt-get install -y dotnet-sdk-9.0
        > sudo apt-get update && sudo apt-get install -y aspnetcore-runtime-9.0
        ```
1. **Install your fav apps:**
    1. Via .deb downloads:
        1. Chrome
        1. Visual Studio Code
    1. Via the App Center:
        1. Blender
        1. GIMP
        1. Inkscape
        1. Libreoffice
    1. Note on Blender: when using OpenGL, setting the `Viewport Shading` to `Material Preview` freezes up Blender (it freezes and then eventually goes black). 
        I find that installing vulkan using the line below, and then switching the backend display graphics to Vulkan solves that problem. You can switch
        to Vulkan by going to the Preferences window (Menu->Edit->Preferences), clicking on the `System` tab, and then in the Display Graphics header, change
        the `Backend` value from from `OpenGL` to `Vulkan` (might need a restart or at least `source ~/.bashrc`).
        ```sh
        sudo apt install vulkan-tools
        ```
    1. I downloaded the binary/flatpak installs for the following programs:
        1. Bambu studio: https://github.com/Bambulab/BambuStudio/releases/tag/v01.10.01.50
        1. Cursor: https://www.cursor.com/ 
        1. Godot game engine: https://godotengine.org/download/linux/ 
        1. LogSeq (for mind maps): https://logseq.com/downloads 
        1. Obsidian (for mind maps): https://obsidian.md/download
    1. For scientific computing:
        1. Docker (https://docs.docker.com/engine/install/ubuntu/)
            ```sh
            > sudo apt update
            > sudo apt install ca-certificates curl
            > sudo install -m 0755 -d /etc/apt/keyrings
            > sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
            > sudo chmod a+r /etc/apt/keyrings/docker.asc
            # Add the repository to Apt sources:
            > echo \
             "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
            $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
            sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
            > sudo apt update
            ```
       1. Anaconda
          ```sh
           wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
           > bash ~/Miniconda3-latest-Linux-x86_64.sh
           # All other python installs are normally put into a conda environment via
           # (3.10 seems more stable than 3.12 for sci comp)
           > conda create --name research python=3.10
           ```
       1. Tachyon (a raytracer for VMD)
           ```sh
           sudo apt-get install tachyon
           ```
       1. VMD (VIsual Molecular Dynamics from UIUC)
           1. Go to the VMD download page (https://www.ks.uiuc.edu/Development/Download/download.cgi?PackageName=VMD)
           1. Download the latest version of LINUX_64 (mine was the V2.0 Alpha version for [LINUX_64 (RHEL 8+) OpenGL, CUDA, OptiX RTX, RTX RTRT](https://www.ks.uiuc.edu/Development/Download/download.cgi?UserID=&AccessCode=&ArchiveID=1730)).
           1. Extract it and cd into the source directory.
               ```sh
               # Extract
               tar xzf vmd-2.0.0a7.bin.LINUXAMD64.tar.gz
               # Change directory
               cd vmd-2.0.0a7.bin.LINUXAMD64/src
               # Install! (to change location of the install, modify the
               # vmd-2.0.0a7.bin.LINUXAMD64/configure file in the parent directory)
               sudo make install
               # VMD should be installed in /usr/local/bin/vmd, and should be available
               # to run from any command prompt
               ```
       1. **If VMD installation does not work**, here are two other molecular visualizers (I prefer VMD for speed publication quality):
           1. Install Avogradro (had a hard time getting VMD for molecules to work):
               ```sh
               sudo apt install avogadro
               ```
           1. Install open sourced PyMol (https://pymol.org/conda/):
               ```sh
               conda install -c conda-forge pymol-open-source
               ```

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

## Other useful links
Here are other links that were useful to me when I was starting out

1. Winston Ma (Jan 2023) *Install Ubuntu 24.04 LTS on Asus Zenbook S13 OLED*. [Medium link](https://winstonhyypia.medium.com/install-ubuntu-22-04-1-lts-on-asus-zenbook-s13-oled-53523ac68046)
1. Reddit thread: [ubuntu 22.04 on Asus Zenbook S 13 OLED](https://askubuntu.com/questions/1442499/ubuntu-22-04-on-asus-zenbook-s-13-oled)
1. Kicking and Screaming's Asus Zenbook S13 Oled 2024 [notes](https://kickingandstreaming.ca/2024/04/asus-zenbook-s13-oled-2024/)

