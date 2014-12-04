This is an [Ansible](http://ansible.com) playbook to bootstrap [DigitalOcean](https://www.digitalocean.com/) virtual server running a [Docker](https://www.docker.com/) instance for [Rancher](http://www.rancher.io/).

# Index
- [Setup](#setup)
  - [Python 2](#python-2)
  - [Python 3](#python-3)
- [Initial Configuration](#initial-configuration)
- [Run!](#run)
- [What this playbook do](#playbook-is-actually-doing-steps)
  - [Droplet](#droplet)
  - [Users and groups](#users-and-groups)
  - [Security settings](#security-settings)
  - [Install Software](#software)
  - [End](#end)
- [Troubleshooting](#troubleshooting)
- [Credits](#credits)

# Setup

## Python 2
With **python2** run:
```
sudo pip install -r requirements.txt
ansible-galaxy install -r requirements.yml
```

## Python 3
If you use **python3** on your system, it's **recommended** to run a **python2 virtualenv**. Assuming [Virtualenv](http://virtualenv.readthedocs.org/en/latest/) and [Virtualenvwrapper](http://virtualenvwrapper.readthedocs.org/en/latest/) are installed and in your `$PATH`, you should then run the following commands in the project folder:

```
mkvirtualenv -p /usr/bin/python2.7 ansible
echo "localhost ansible_python_interpreter=$(which python)" > hosts/localhost.yml
pip install -r requirements.txt
ansible-galaxy install -r requirements.yml
```

# Initial configuration

All configurable variables are in `vars.yml` file. Mandatory parameters are below
* DigitalOcean `Client ID`
* DigitalOcean `API key`
* DigitalOcean `hostname`- used for *[idempotency](http://docs.ansible.com/glossary.html#idempotency)*
* ssh public key file

Run the following command to **apply your configuration**:
```
ansible-playbook -i hosts/localhost.yml inventory.yml
```

# Run!

```
ansible-playbook -i hosts newdroplet.yml
```
Be aware playbook will change root password you received by email. SSH login for the root user will be disabled as well
so use admin user account and `sudo` instead. All new passwords will be saved into `./credentials/$hostname/$username` files.

# Playbook is actually doing steps

## Droplet
* Deploy **ssh keys** to DigitalOcean
* Create **droplet**

## Users and groups
* Configure remote access
  * Change default **root password** - always a good idea, if you received your password by email in clear text
  * Create **admin user profile** - it would be a main profile for all operations

## Security settings
* Configure **sudo** -- allow `sudo` for certain groups of users (administrators)
* Configure **ssh**
  * Create new server **ssh keys** -- Despite the fact that [this](https://www.digitalocean.com/company/blog/avoid-duplicate-ssh-host-keys/) is not an issue anymore it seems like good practice for me. The default keys have been created out of my control.
  * Restrict **sshd** settings
    * Restrict ssh login for certain group of users (`AllowGroup`)
    * **Disable root login** via ssh (`RootLogin no`)
    * **Password guessing protection** (`MaxAuthTries`, `LoginGraceTime`, `MaxSessions`, `MaxStartups`)
* Enable **ufw** (firewall)

## Software

* Install **updates** -- it goes without saying
* **Configure mail** forwarding for admin user -- to keep in touch with my servers
* Install **Docker**
* Run **[Rancher](http://www.rancher.io/) server** and **agent**
* **fail2ban** -- prevent password guessing
* **ntpd** is not about security however it's a vps so this thing is essential
* **tmux**, **git**, **wget**, **curl**, **rsync**, **zip**, **vim**, ...


# End

Visit http://YOUR_DROPLET_IP:8080

# Troubleshooting

## SSH connection problems (TODO)
- `known_hosts`: Try to resolve the problem using `ssh-keyscan` or remove the broken entry from your `known_hosts`

- If your host is already provisioned, you should then change the `user` in `newdroplet.yml` according to your `vars.yml`. It would be also necessary to add an additional field like the following one:
`sudo: true`

## Dynamic inventory problems
- `No JSON object could be decoded`: check and apply your [configuration](#initial-configuration)
# Credits

- Forked from [hostmaster/ansible-digitalocean-bootstrap](https://github.com/hostmaster/ansible-digitalocean-bootstrap)

- `hosts/digital_ocean.py` from [ansible/ansible](https://github.com/ansible/ansible)
