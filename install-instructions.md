# Instructions for trying Trunk

## Please Read First

Reach out if you're having trouble; I can help you and it helps me learn where the dark corners are. Seriously. If you ran into this somehow and don't know how to contact me, <https://github.com/erikh> can help you. There is a place for reporting issues long-form [here](https://github.com/trunk-os/community/issues).

## Setup

First, install [VirtualBox](https://virtualbox.org) if you have not already. This will allow you to import the OVA, which you will download in a minute. You may need to reboot after you install it.

## Download the Image

Go [here](https://drive.google.com/file/d/1xw9scBbmD2KpALgtcjAHht8njd3P5LHx/view?usp=drive_link) to download the OVA, which is a full VM application complete with proper disk, network, etc setup.

## Import the Image

After opening VirtualBox, look for an "Import" button that looks like a yellow, upside-down magnet; it's either in the top-left drop-down menu or a button on the screen.

Click it and select the file you just downloaded. You should be given some options to change the machine.

### Changing the Machine Settings

This is the default for this configuration: they are the minimum recommended through experimentation. Notes are provided on what increasing them benefits

- 4096MB ram (increasing ram will dramatically increase disk performance)
- 2 cpus (increasing cpus will have a minimal effect unless you're serving something)
- Bridged network
    - **IMPORTANT**: In many cases, the adapter will not be set right when importing the application. You must change it to the one you use, otherwise the machine will be impossible to find and will not be able to see the network. It may not even boot.
        - On linux, likely starting with `en` for a ethernet wire, or `wl` for wireless.
        - Mac and Windows should have less options and it should be a little more obvious I think. Please reach out if you have trouble.
- Disks:
    - Disk ordering matters. See below for how things should be organized, configuration-wise.
    - 1 host disk (max cap 100GB, ~6GB current)
        - **Do not replace this disk.**
    - 4 disks for zfs (each @ 8GB, uses ~2MB without writes)
        - **NOTE** If you replace these disks, you **MUST** do it before the \*FIRST BOOT\*\*. They need to be of uniform size, and 4 disks. While configurations of 2 and 1 are also theoretically supported, they are not currently tested and may result in issues.

### Other Notes for Tinkering

- Do not enable secure boot. It will yield a malfunctioning system.
- If you want to dig deeper into how things are put together, you can ssh in as `root@trunk.local`. The password is `trunkrules`. Note, you cannot modify the vast majority of the filesystem, including most configuration and nearly all of the binaries installed. If you want help trying to do something different in this regard, reach out.

## Start the VM

Press the Start button, near the top in the Machines panel for Trunk. If you have trouble starting:

### If this is your first time using VirtualBox

If you have not before, you will need to reboot to BIOS and enable either SVM support on AMD CPUs or VT-X on Intel. This is so you can run virtual machines on your hardware. It is not needed for Apple and note that not all PCs (Laptops especially) support the feature. Contact me if you're in this situation. There are solutions, they just aren't here yet; it's a priority thing. Raise your voice.

### Linux, KVM, and VirtualBox

VirtualBox often conflicts with KVM, so try this if the VM refuses to start:

`sudo modprobe -r kvm_amd && sudo modprobe -r kvm`

## Multicast DNS and why it's important

Multicast DNS is a protocol trunk uses heavily to present itself openly on a local area network. You must currently have a multicast capable computer to work with Trunk.

Note, the issues here are largely aimed at Linux; both Windows and Mac OS have thorough support for Multicast DNS out of the box. Some instructions to resolve things are below.

But because of this: **YOU MUST THE BE THE ONLY PERSON ON YOUR NETWORK RUNNING TRUNK**. This is not a long term strategy, and it will be fixed as soon as I get an opportunity.

### Fixing Multicast on Linux

If you want to test your multicast capabilities, try running this in a shell: `ping $(hostname -s).local`. If you get an error, you need to follow these instructions. You can press `Control+C` to exit the program after it shows you response times.

If you cannot see `trunk.local` either with ping, ssh or a browser 3 minutes or so after boot; ensure the following stack of tools installed on your desktop with your distribution package tool.

These are not package names; you will need to find those for your distribution. They are the software names, so search for them with your preferred package installation tool.

- nss MDNS library
    - this is used to look names up
- avahi-daemon
    - this is used to manage DNS itself
    - may also be called `avahi`
    - ensure systemd service is functioning: `systemctl start avahi-daemon`
- you may also want `systemd-network`; this will ensure maximum compatibility with the rest of linux.
    - `systemctl enable systemd-networkd && systemctl start systemd-networkd`
    - `systemctl enable systemd-resolved && systemctl start systemd-resolved`

## Accessing Trunk

After a while, you should be able to visit <http://trunk.local>. How long it takes to come up depends on your computer.

The current interface is intended as a way to exercise the system functionality; please do not look at this as a representation of the final product. It's just crossing the t's and dotting the i's.

### Creating an account

First click the "Create an Account" button near the bottom of the login screen. Fill it out, click submit and wait for a green confirmation dialog. Go back to <http://trunk.local> and login with the information.

### Installing a Package

One thing you can do is provision a stock nginx. It should forward the ports directly on your router, and be available externally in a secure manner.

To do this, go to the `Packages` panel. Then, you can click the red "x" next to the nginx-example-page package to start the installation process. Give it a port number, which is something you will want to write down for later, and hit OK. Then, wait for the green check to appear. Report any errors you get.

Then, you will want to do this in a console: `curl ipinfo.io` or you can also visit a site like <https://ipchicken.com> to get your external IP address.

**NOTE** if this IP address does not work, first ensure you are not connected through a VPN, or google or apple security proxy in your browser. If in doubt, try the console command above; it will usually be right.

Then, you will want to visit `http://<ip>:<port>` which is the external IP and the port number you gave the package during install. Assuming everything works, you will see the nginx default example page.

Clicking the green check next to the nginx-example-page in the packages panel will uninstall it for you, after a confirmation.

### Reviewing System Statistics

The Systems Stats Dashboard is a comprehensive panel that can be customized. A default is installed and you can play with the viewing options such as timespan and how often refresh, etc.

The dashboard is locked from being edited and is customized through external means. I want to make it so that you can drop in your own dashboards, but make it hard enough so that people can't have unhappy accidents while playing with it.

It is also available at <http://trunk.local:3030>.

### Other things you can do

There are a number of things in the UI I think would be worth going into, but I kind of want to leave it here and see how well you do from there. If it's hard to use, I'd love to hear about it. Please! I don't want to give you instructions to ensure this feedback is as pure as possible.

## Sincerest Gratitude

Thanks and Happy Holidays,

Erik Hollensbe <erik@hollensbe.org>
