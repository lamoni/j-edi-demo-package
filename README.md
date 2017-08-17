# Introduction

j-edi-demo-package houses three Packer projects for building and provisioning three VMs: A J-EDI instance, a demo UI that integrates with the J-EDI instance, and a vSRX.  Using these Packer files, you can instantiate a J-EDI environment with a (relatively) few commands.

# Pre-requisites

- Install Packer (https://packer.io)
- Clone this repo (https://github.com/Juniper/j-edi-demo-package)
- VMWare Workstation / Fusion (Virtualbox support being tested, other hypervisor support will be added as requested)

# Step 1 - Build J-EDI Instance
```
Within the j-edi-demo-package/j-edi-demo-packer directory, execute:

packer build j-edi-demo-packer.json
```

When Packer is done building, you will have a VM with J-EDI installed and running


# Step 2 - Build Demo UI Instance
```
Within the j-edi-demo-package/j-edi-ui-packer directory, execute:

packer build j-edi-ui-packer.json
```

When Packer is done building, you will have a VM with a very bare demo UI that integrates with the J-EDI instance from Step

# Step 3 - Configure Demo UI Instance

NOTE: Replace the IP address "192.168.150.130" with the IP ADDRESS OF THE VM BUILT IN STEP 1.  It is imperative you do this, or the UI will not work.
```
source ~/set_salt_bus.sh 192.168.150.130
cd /opt/j-edi-ui/app
sudo -E npm install
sudo -E npm start

In another terminal session:

cd /opt/j-edi-ui/j-edi-ui
sudo -E npm install
sudo -E npm start


Both sessions need to be left open for the UI to continue running (there are ways to daemonize this, but... that's not really the point of this environment)
```

# Step 4 - Build vSRX

```

First, download this vSRX image that has been primed to work with Packer (https://www.dropbox.com/s/lbqcjfj7typ1vzf/vsrx.zip?dl=1), and unzip the files into the j-edi-vsrx/image/ directory

Within the j-edi-demo-package/j-edi-vsrx/ directory, execute:

packer build j-edi-vsrx.json
```

When Packer is done building, you will have a vSRX VM configured with the credentials of (root / Clouds123, juniper / Clouds123)


# Step 5 - Configure J-EDI instance

NOTE: Replace the following "192.168.150.134" with the IP ADDRESS OF THE VSRX THAT WAS BUILT IN STEP 4.
```
On the master VM:

sudo salt master state.apply jedi.junos_proxy
sudo salt-run proxy_minion.create_with_defaults vsrx1 192.168.150.134

```

# Step 6 - Verify the UI is connected to the J-EDI instance
```
- Browse to the IP address of the UI VM on port 4200, you should see a blue lightning bolt in the upper right corner
```

# Step 7 - Verify the VSRX has been correctly configured as a proxy minion on the J-EDI VM (Salt Master)
```
- On the J-EDI VM, execute the following command and ensure it returns successfully:
   salt 'vsrx1' test.ping
```


You now have a J-EDI environment that can be used for the included demo docs and any other documentation currently found for J-EDI.



NOTES:
```
For configuration backups, use:
Modify production.sls in gitlab instance to enable syslog_host, gitlab, and junos_backup
salt-run state.orch plugins.junos_backups.install

For config pushing, use:
salt-run state.orch plugins.junos_orchestration.install
```