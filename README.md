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
- First, plug in the USB flashed USB drive. 
- To get into the BIOS, repeatedly press F2 right after powering up. There, turn off Secure Boot and change the Boot options to list your USB first.
- When you restart, you will boot into your USB drive, where you can install Ubuntu.

