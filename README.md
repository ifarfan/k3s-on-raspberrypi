# `K3S` installation on `Raspberry Pi` cluster


## Pre-requisites
1. Download [Raspbian lite](https://downloads.raspberrypi.org/raspbian_lite/images/), look for most recent OS image –– unzip file

1. Use [balena's Etcher](https://www.balena.io/etcher/) to write the Raspbian image to an SD card; to install it on a Mac via [Homebrew](https://brew.sh/):
   ```
   $> brew cask install balenaetcher
   ```

1. Mount SD card (i.e., eject + reinsert SD card)

1. Create `/boot/ssh` file to allow SSH on RPi
   ```
   $> touch /Volumes/boot/ssh
   ```

1. Append IP settings to end of `/boot/cmdline.txt` file to set static IP
   ``` ip=192.168.136.XX::192.168.136.1:255.255.255.0:eth0:false```
   Refer to [this page](https://kr15h.github.io/RPi-Setup/) for an explanation + configuration string format

1. Unmount + eject SD card, pop it in a Raspberry-pi and boot

1. Repeat the previous 4 steps on each SDs per RPi available

1. On first boot, login as `pi` and use `raspberry` as default password
   ```
   $> ssh pi@192.168.136.23
   ```
1. Rejoice + enjoy!


## Cluster Specs
Below are listed my cluster's current configuration specs:

| Model         | IP             | Proc   | CPU | Mem   | Disk |
| ------------- | -------------- | ------ | --- | ----- | ---- |
| RPI 3 Model B | 192.168.136.30 | armv7l |  4  | 1GB   | 32GB |
| RPI 3 Model B | 192.168.136.31 | armv7l |  4  | 1GB   | 32GB |
| RPI 3 Model B | 192.168.136.32 | armv7l |  4  | 1GB   | 32GB |
| RPI 3 Model B | 192.168.136.33 | armv7l |  4  | 1GB   | 32GB |
| RPI Model B+  | 192.168.136.34 | armv6l |  1  | 512MB |  8GB |

See: [Raspberry Pi model comparison](https://www.element14.com/community/servlet/JiveServlet/previewBody/82195-102-3-346675/PiPoster_14Jun16.pdf)


## Installation + configuration via Ansible
1. Check for correct IP settings under `hosts.yml`, note that the master is tagged as `k3s_master` and the agents as `k3s_agent`

1. Reset `pi` user's password and add Ansible ssh key (you'll be prompted for default password and the ssh key under `vars/main.yml > ansible_ssh_key` will be added)
   ```
   $> ansible-playbook playbooks/reset_pi_password.yml
   ```

1. Ensure you can ssh to the RPi cluster, see sample `~/.ssh/config` excerpt below
   ```
   Host 192.168.136.3*
        User pi
        IdentityFile ~/.ssh/ansible_rsa
        IdentitiesOnly yes
        PasswordAuthentication no
        StrictHostKeyChecking no
        UserKnownHostsFile=/dev/null
        ControlMaster auto
        ControlPath /tmp/%r@%h:%p
        LogLevel FATAL
   ```

1. Configure default RPis host settings (using your ssh key from now on)
   ```
   $> ansible-playbook playbooks/configure_host.yml
   ```
1. Install k3s on all RPis (1 master + 3 agents)
   ```
   $> ansible-playbook playbooks/k3s_install.yml
   ```


## Oh god, why?!?
Kubernetes is all the rage these days but it's bloated `as hell!` :scream:. I've had a small cluster of Raspberry Pis that I've used for all sorts of shenanigans (even trying to install Kubernetes on them :see_no_evil:) and ever since I've heard that the `Rancher` guys managed to slash the bejesus out of `k8s` it was just a matter of time before these puppies were going to be repurposed (once again!?! :flushed:).

Performance is great and what better way to get k8s in every IOT device in the world!

Next stop... getting rid of the OS altogether and running bare [k3os](https://k3os.io/) on these RPis. :smirk:
