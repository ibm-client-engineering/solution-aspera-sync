---
id: solution-prepare
sidebar_position: 1
title: Prepare
---

## Pre-Requisites

- 2 **reachable hosts** that the servers will run on
- 2 **Aspera License** with sync for each server
  - /resources/ansible/roles/configure-aspera/files/`AsperaEnterprise-license-primary`
  - /resources/ansible/roles/configure-aspera/files/`AsperaEnterprise-license-secondary`
- An install of `ibm-aspera-hsts` binary to be installed for the servers
  - /resources/ansible/roles/install-aspera/files/`ibm-aspera-hsts-4.4.2.550-linux-64-release.deb`

## Optional: Running Solution locally using Virtual Machines, Vagrant

You are able to create the provided solution locally using `VMs`. HashiCorp Vagrant is used to simplify this step, run the following command below within the vagrant directory to provision `2 virtual machines`. Assuming Vagrant is installed and configured properly.

- hosts public key are required to be in the vagrant directory and called `id_rsa.pub`. This will allow you to ssh into the provisioned machines.

```sh
vagrant up
```

## Update the playbook configurations as needed to suit your environment
