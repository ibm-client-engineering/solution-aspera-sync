---
id: solution-deploy
sidebar_position: 2
title: Deploy
---

# Deploy

:::info

** Manually deploy to learn. Automatically deploy to scale.**

:::

## Manual Deployment

The following steps are meant to be followed on both servers. The following will be targeted to ubuntu servers.

### Install HSTS

```
Summary: Install Aspera HSTS from link below. Once installed, copy the binaries over to the hosts machines to be run and get Aspera installed.
```

Download Aspera High Speed Transfer Server (HSTS) binary [here](https://www.ibm.com/support/fixcentral/swg/selectFixes?parent=ibm~Other%20software&product=ibm/Other%20software/IBM%20Aspera%20High-Speed%20Transfer%20Server&release=All&platform=All&function=all)

Copy the following binary over to your two host machines via `scp`.

```sh
scp -i {vmHosts.pem} ibm-aspera-hsts-4.4.2.550-linux-64-release.deb {hostUsername}@{hostIpAddress}:/tmp
```

Install the binary by running the following in the `/tmp` directory.

```sh
dpkg -i ibm-aspera-hsts-4.4.2.550-linux-64-release.deb
```

### Configure Aspera License

```
Summary: Download your aspera license and transfer over to the hosts.
```

Run the following to transfer your aspera license over

```sh
scp -i {vmHosts.pem} aspera-license {hostUsername}@{hostIpAddress}:/opt/aspera/etc/aspera-license
```

To verify your license key, run the following command

```sh
ascp -A
```

### Services Configuration

Enable the following services for Watch Folders. The following command is provided below.

```sh
systemctl enable systemd-networkd
systemctl enable systemd-networkd-wait-online.service
```

### Firewall Configuration

Port `33001` must be opened for Aspera to run. You can open it by running the following below.

```sh
ufw enable # enables ufw
ufw allow 22/tcp # ssh port
ufw allow 33001/tcp
ufw allow 33001/udp
```

### SSH Configuration

To keep this minimal, we will be using password authentication for aspera sync. Thus `PasswordAuthentication` must be set to `yes` and to allow port `33001` act as an ssh medium. Make sure the following exists in `/etc/ssh/sshd_config`

```
Port 33001
PasswordAuthenticaiton yes
```

Make sure to restart SSH service once done by running

```sh
systemctl restart sshd.service
```

### User Configuration

For Aspera to work, we will need a transfer user. To simplify it, we will create a user called `asperauser` and make him into a transfer user in the next following step.

#### Create user `asperauser`

```sh
adduser asperauser
```

Create 3 folders in asperauser's directory shown below. `asperacontent` folder will be used to sync between the two servers.

```sh
mkdir /home/asperauser/{asperaContent,asperaSyncLogs,asperaSyncDbdir}
```

### Aspera Configuration

Run the following to configure Aspera Sync.

```sh
asconfigurator -x "set_user;username,asperauser,absolute,/home/asperauser/asperaContent"
asconfigurator -x "set_user;username,asperauser,read_allowed,true"
asconfigurator -x "set_user;username,asperauser,write_allowed,true"
asconfigurator -x "set_user;username,asperauser,dir_allowed,true"
asconfigurator -x "set_user;username,asperauser,authorization_transfer_in_value,allow"
asconfigurator -x "set_user;username,asperauser,authorization_transfer_out_value,allow"
asconfigurator -x "set_user;username,asperauser,async_log_dir,/home/asperauser/asperaSyncLogs"
asconfigurator -x "set_user;username,asperauser,async_db_dir,/home/asperauser/asperaSyncDbdir"

```

Make sure to restart aspera by running the following

```sh
systemctl restart asperanoded
systemctl restart asperacentral
```

### Aspera Sync

To start the sync between both servers, run the following command on one of the server and target it to the other server by running the following

```sh
async -N asperaSync -d /home/asperauser/asperaContent -r {otherServerUsername}@{otherServerIpaddress}:/ -l 100M -a fair -g 1M -G 1M -C -K BIDI
```

You are able to read more of async commands by clicking [here](https://www.ibm.com/docs/en/ahts/4.4?topic=async-command-reference)

## Automated Deployment via Ansible

**Ensure that the inventory file is properly configured for your environment**

#### Hosts validation

You can verify that you can reach your host by running the following command inside the `/resources/ansible` directory.

```sh
ansible all -m ping
```

`Do not proceed till you are able to verfiy the hosts`

### Running the playbook

#### Before running the playbook, complete the following

- Place `ibm-aspera-hsts-*.deb` inside /resources/ansible/roles/install-aspera/files directory
- Place 2 Aspera license inside /resources/ansible/roles/configure-aspera/files directory
  - `AsperaEnterprise-license-primary`
  - `AsperaEnterprise-license-secondary`

You can run the playbook by running the following. The playbook should take an average of 4 mintues

```sh
ansible-playbook playbook.yml
```
