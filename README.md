# Drupal Pi

**Drupal on a single Raspberry Pi**

<p align="center"><img src="https://raw.githubusercontent.com/geerlingguy/drupal-pi/master/images/drupal-pi-model-2.jpg" alt="Drupal 8 on a Raspberry Pi" /></p>

This project is an offshoot of the [Rasbperry Pi Dramble](https://github.com/geerlingguy/raspberry-pi-dramble) project, which helps install Drupal on a cluster ('Bramble') of Raspberry Pi 2 computers.

This playbook/project makes setting up Drupal on a _single_ Raspberry Pi a very easy/simple operation.

## Currently-supported Raspberry Pi models

  - Raspberry Pi model 1 B+ (512MB RAM, single CPU core)
  - Raspberry Pi model 2 B (1GB RAM, four CPU cores)

The Raspberry Pi model 2 B is much better supported, though, and will give a much better user experience, since the Pi 2 is almost 4x faster in many common scenarios (and has double the RAM to boot!).

## Set up the Raspberry Pi

Drupal requires as good a Raspberry Pi as you can afford. While Drupal will run okay on any Raspberry Pi, it's best to use a B+ (which has a single core processor and 512MB RAM) or B model 2 (which has a four-core processor and 1GB RAM).

Once you have your Raspberry Pi and a good microSD card (the fastest/best one you can get!), you will need to do a few things to set up the Raspberry Pi and get it ready to run the Drupal installation playbook:

  1. Download the latest 'Raspbian' image from the [Raspberry Pi Downloads page](https://www.raspberrypi.org/downloads/)†.
  2. Follow the [image installation guide](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) to transfer the image to your microSD card.
  3. Once Raspbian is loaded on the card, insert the card in your Pi, and plug in your Pi to boot it up.
  4. **If you don't have a monitor attached to the Pi**: TODO.
  5. **If you have a monitor attached to the Pi**:
    1. Boot up the Raspberry Pi. It will automatically launch the `raspi-config` tool to configure some important settings.
    2. Select the 'Expand filesystem' option and confirm.
    3. Select the 'Change User Password' option and enter a secure password for the `pi` account.
    4. Select the 'Localization options' option and set your language, timezone, and keyboard settings.
    5. Select the 'Advanced options' option and choose a good hostname (e.g. `my-pi`), and also set the 'memory split' to `16` MB, unless you're going to use the Pi with a GUI and need the video memory.
    6. Select the 'Finished' option to reboot the Pi.
  6. **If you have a wireless network adapter you need to set up**: Please see this guide: [Setting up a WiFi Adapter on a Raspberry Pi](http://www.midwesternmac.com/blogs/jeff-geerling/edimax-ew-7811un-tenda-w311mi-wifi-raspberry-pi)
  7. Install Ansible:
    1. Update the local apt cache: `sudo apt-get update`
    2. Install pip and Python dev tools: `sudo apt-get install -y python-dev python-pip`
    3. Install Ansible: `sudo pip install ansible`
  8. Test the Ansible installation: `ansible --version` (should output the Ansible version).

*† If you plan on using your Pi as a headless Drupal server, you don't need all the extra software included with the default Raspbian image. I recommend instead using the latest version of the [Diet Raspbian](http://files.midwesternmac.com/#raspberry-pi-images) image, which is less than half the size of Raspbian.*

## Install LEMP software stack and Drupal

Now that the Raspberry Pi is set up and ready to go, you need to download this repository to the Pi, then run the included playbook to install and configure everything.

  1. Clone the `drupal-pi` project: `git clone https://github.com/geerlingguy/drupal-pi.git`
  2. cd into the project directory: `cd drupal-pi`
  3. Install required Ansible roles: `sudo ansible-galaxy install -r requirements.txt`
  4. Run the Ansible playbook: `ansible-playbook -i inventory -c local main.yml`†

After a few minutes, the playbook should complete successfully, and you should have Drupal running on your Raspberry Pi, accessible via `http://drupalpi.dev/` (make sure you [add an entry to your local hosts file](http://www.rackspace.com/knowledge_center/article/how-do-i-modify-my-hosts-file) for the Pi's address, e.g. `[PI_IP_ADDRESS]  drupalpi.dev`).

> † _Note for Pi B+_: For the Rasbperry Pi model 1 B+ (which only has 512MB of RAM and one CPU core), you need to override the `raspberry_pi_model` variable when running the Ansible playbook (step 4 above). The command you should run (if you don't want to change the variable directly inside `vars/main.yml`) is: `ansible-playbook -i inventory main.yml --extra-vars="raspberry_pi_model='1-b+'"`

## Updating your Pi (for future versions of Drupal Pi)

If you need to update Drupal Pi, do the following:

  1. cd into the project directory: `cd /path/to/drupal-pi`
  2. Pull the latest changes: `git pull`
  3. Update all required Ansible roles (and install new ones): `sudo ansible-galaxy install -r requirements.txt --force`
  4. Run the Ansible playbook: `ansible-playbook -i inventory -c local main.yml`†

> † _See note for Pi B+ in the above section._

## Running the Ansible playbook from another host

If you want to run the Ansible playbook from another host (instead of from within the VM—this also allows you to do everything without installing `pip` and `ansible` on the Raspberry Pi itself!), you can change the inventory file to read as below, and use the same `ansible-playbook` command without the `-c local` option (e.g. `ansible-playbook -i inventory main.yml`):

    [pi]
    PI_IP_ADDRESS_HERE ansible_ssh_user=pi

Replace `PI_IP_ADDRESS_HERE` with the IP address or domain name of your Raspberry Pi.

## Resetting the Drupal Install

There is a `reset.yml` playbook included that will reset the environment so you can install a fresh copy of Drupal. To run the playbook, enter the following command in the same directory as this README:

    ansible-playbook -i inventory reset.yml

After it finishes resetting the environment, you can run the `main.yml` playbook again to rebuild the Drupal site.
