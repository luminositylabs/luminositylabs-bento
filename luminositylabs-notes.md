# Luminosity Labs Notes

## Building the boxes with packer

These notes were last updated to reflect building on macOS Sonoma v14.5, packer v1.11.1, parallels v19.4.1, and
 virtualbox v7.0.18 (2024-07-09).

### Current known issues

- [2024-07-09] MacOS v14.5 / VirtualBox v7.0.18 / AlmaLinux & RockyLinux 8.10 amd64

The AlmaLinux8 and RockyLinux8 boxes build successfully for VirtualBox amd64 platform, but deploying the box with
 vagrant has problems with the VirtualBox Guest Additions.  Getting the additions work involves installing RPMs after
 initial creation of the VM by vagrant and then reloading the VM.
```
vagrant up --provider virtualbox
vagrant ssh
sudo dnf install -y kernel-headers kernel-devel
exit
vagrant reload
vagrant ssh
sudo systemctl status vboxadd-service vboxadd
```

- [2024-07-09] MacOS v14.5 / VirtualBox v7.0.18 / Ubuntu 24.04 amd64

The Ubuntu boxes build successfully for Virtualbox, but the Virtualbox Guest Additions are not able to load kernel
 modules in some cases, due to situations where the guest additions setup needed packages which were not installed.
 The bento packer scripts may not install them or remove them before packagint the box in order to save space.  The
 following is an example of installing the packages and triggering a guest additions setup to rebuild and load the
 kernel modules so that the guest additions services run.

```
vagrant up --provider virtualbox
vagrant ssh
sudo apt-get install -y linux-headers-generic gcc make
exit
vagrant reload
vagrant ssh
sudo systemctl status vboxadd-service vboxadd
```

### Plugin initialization

The boxes are built with packer which uses a plugin system for modularity.  The plugins require initialization before
they can be used.  To initialize/re-initialize for bento project, the following command can be used:
```
packer init -upgrade ./packer_templates
```
  
### AlmaLinux

#### almalinux-8-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/almalinux/almalinux-8-x86_64.pkrvars.hcl ./packer_templates
```
virtualbox
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/almalinux/almalinux-8-x86_64.pkrvars.hcl ./packer_templates
```

#### almalinux-9-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/almalinux/almalinux-9-x86_64.pkrvars.hcl ./packer_templates
```
virtualbox
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/almalinux/almalinux-9-x86_64.pkrvars.hcl ./packer_templates
```

#### almalinux-9-aarch64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/almalinux/almalinux-9-aarch64.pkrvars.hcl ./packer_templates
```

### RockyLinux

#### rockylinux-8-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/rockylinux/rockylinux-8-x86_64.pkrvars.hcl ./packer_templates
```
virtualbox
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/rockylinux/rockylinux-8-x86_64.pkrvars.hcl ./packer_templates
```

#### rockylinux-9-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/rockylinux/rockylinux-9-x86_64.pkrvars.hcl ./packer_templates
```
virtualbox
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/rockylinux/rockylinux-9-x86_64.pkrvars.hcl ./packer_templates
```

#### rockylinux-9-aarch64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/rockylinux/rockylinux-9-aarch64.pkrvars.hcl ./packer_templates
```

### Ubuntu

#### ubuntu-20.04-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-20.04-x86_64.pkrvars.hcl \
    -var "parallels_boot_wait=1s" \
    ./packer_templates
```
virtualbox (overrides are necessary to build)
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-20.04-x86_64.pkrvars.hcl \
    -var "vbox_boot_wait=5s" \
    -var "boot_command=[\"<wait><esc><wait><esc><wait><esc><wait><enter><wait>/casper/vmlinuz initrd=/casper/initrd quiet autoinstall ds=nocloud-net;s=http://{{.HTTPIP}}:{{.HTTPPort}}/ubuntu/<enter>\"]" \
    ./packer_templates
```

#### ubuntu-20.04-aarch64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-20.04-aarch64.pkrvars.hcl ./packer_templates
```

#### ubuntu-22.04-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-22.04-x86_64.pkrvars.hcl ./packer_templates
```
virtualbox
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-22.04-x86_64.pkrvars.hcl ./packer_templates
```

#### ubuntu-22.04-aarch64

parallels
```
time packer build -only=parallels-iso.vm -var "parallels_boot_wait=3s" -var-file=os_pkrvars/ubuntu/ubuntu-22.04-aarch64.pkrvars.hcl ./packer_templates
```

#### ubuntu-24.04-x86_64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-24.04-x86_64.pkrvars.hcl ./packer_templates
```
virtualbox
```
time packer build -only=virtualbox-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-24.04-x86_64.pkrvars.hcl ./packer_templates
```

#### ubuntu-24.04-aarch64

parallels
```
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-24.04-aarch64.pkrvars.hcl ./packer_templates
```

## Using the boxes with vagrant

1. Create a new box
```
vagrant cloud box create -s "AlmaLinux 9 prepared with packer templates from Chef Bento project" --no-private luminositylabsllc/bento-almalinux-9
```

2. Create a new version of the box
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-almalinux-9 20240709.01
```

3. Create a provider for the version of the box:
```
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 parallels  20240709.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 virtualbox 20240709.01
```

4. Upload the box file for the provider:
```
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 parallels  20240709.01 <file>
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 virtualbox 20240709.01 <file>
``` 

5. Release a version:
```
vagrant cloud version release luminositylabsllc/bento-almalinux-9 20240709.01
```

***NOTE:*** vagrant also has a "publish" command which combined all the steps above, but only allows a single provider
to be specified at a time.  The command may be invoked multiple times, once for each provider, but it's not certain
what happens if the non-provider related configuration properties change between invocations.
                                                                                             
```
vagrant cloud publish --box-version v20240709.01 \
                      -d "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --version-description "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" \
                      --no-private -r \
                      -s "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --no-direct-upload luminositylabsllc/bento-almalinux-9 20240709.01 parallels builds/almalinux-9.2-x86_64.parallels.box

vagrant cloud publish --box-version v20240709.01 \
                      -d "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --version-description "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" \
                      --no-private -r \
                      -s "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --no-direct-upload luminositylabsllc/bento-almalinux-9 20240709.01 virtualbox builds/almalinux-9.2-x86_64.virtualbox.box
```

### AlmaLinux

AlmaLinux8 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-almalinux-8  20240709.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-8 parallels   20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-8 parallels  20240709.01 builds/almalinux-8.8-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-almalinux-8 virtualbox  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-8 virtualbox  20240709.01 builds/almalinux-8.8-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-almalinux-8  20240709.01
```

AlmaLinux9 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-almalinux-9  20240709.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 parallels   20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 parallels  20240709.01 builds/almalinux-9.4-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 virtualbox  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 virtualbox  20240709.01 builds/almalinux-9.4-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-almalinux-9  20240709.01
```

AlmaLinux9 aarch64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1" luminositylabsllc/bento-almalinux-9-aarch64  20240709.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9-aarch64 parallels   20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9-aarch64 parallels  20240709.01 builds/almalinux-9.4-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-almalinux-9-aarch64  20240709.01
```

### RockyLinux

RockyLinux8 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-rockylinux-8 20240709.01
vagrant cloud provider create luminositylabsllc/bento-rockylinux-8 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-8 parallels  20240709.01 builds/rockylinux-8.8-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-rockylinux-8 virtualbox 20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-8 virtualbox 20240709.01 builds/rockylinux-8.8-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-rockylinux-8 20240709.01
```

RockyLinux9 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-rockylinux-9 20240709.01
vagrant cloud provider create luminositylabsllc/bento-rockylinux-9 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-9 parallels  20240709.01 builds/rockylinux-9.3-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-rockylinux-9 virtualbox 20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-9 virtualbox 20240709.01 builds/rockylinux-9.3-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-rockylinux-9 20240709.01
```

RockyLinux9 aarch64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1" luminositylabsllc/bento-rockylinux-9-aarch64 20240709.01
vagrant cloud provider create luminositylabsllc/bento-rockylinux-9-aarch64 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-9-aarch64 parallels  20240709.01 builds/rockylinux-9.3-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-rockylinux-9-aarch64 20240709.01
```

### Ubuntu

Ubuntu 20.04 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-ubuntu-20.04 20240709.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-20.04 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-20.04 parallels  20240709.01 builds/ubuntu-20.04-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-ubuntu-20.04 virtualbox 20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-20.04 virtualbox 20240709.01 builds/ubuntu-20.04-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-20.04 20240709.01
```

Ubuntu 20.04 aarch64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1" luminositylabsllc/bento-ubuntu-20.04-arm64 20240709.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-20.04-arm64 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-20.04-arm64 parallels  20240709.01 builds/ubuntu-20.04-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-20.04-arm64 20240709.01
```

Ubuntu 22.04 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-ubuntu-22.04 20240709.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-22.04 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-22.04 parallels  20240709.01 builds/ubuntu-22.04-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-ubuntu-22.04 virtualbox 20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-22.04 virtualbox 20240709.01 builds/ubuntu-22.04-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-22.04 20240709.01
```

Ubuntu 22.04 aarch64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1" luminositylabsllc/bento-ubuntu-22.04-arm64 20240709.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-22.04-arm64 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-22.04-arm64 parallels  20240709.01 builds/ubuntu-22.04-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-22.04-arm64 20240709.01
```

Ubuntu 24.04 amd64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1, virtualbox v7.0.18" luminositylabsllc/bento-ubuntu-24.04 20240709.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-24.04 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-24.04 parallels  20240709.01 builds/ubuntu-24.04-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-ubuntu-24.04 virtualbox 20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-24.04 virtualbox 20240709.01 builds/ubuntu-24.04-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-24.04 20240709.01
```

Ubuntu 24.04 aarch64
```
vagrant cloud version create -d "box v20240709.01, packer v1.11.0, parallels v19.4.1" luminositylabsllc/bento-ubuntu-24.04-arm64 20240709.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-24.04-arm64 parallels  20240709.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-24.04-arm64 parallels  20240709.01 builds/ubuntu-24.04-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-24.04-arm64 20240709.01
```

-------------------
# HashiCorp Cloud Platform (HCP)

Documentation https://developer.hashicorp.com/hcp/docs/hcp
  
## CLI tool installation

Install HCP CLI on MacOS via Homebrew:
```
brew tap hashicorp/tap
brew install hashicorp/tap/hcp
hcp
```

## Authentication

Authentication to HCP for Vagrant Registries migrated from VagrantCloud involves HCP organizations,
 projects, service principals, and keys.

### Service Principal Key Generation

To authenticate to HCP, a service principal needs to be setup and a key generated.
  The ClientID and ClientSecret from the generated key are then used to authenticate via the HCP CLI

1. In the hcp web portal, navigate to the Organizations list
2. Choose the organization containing the project and vagrant registries
3. Choose the project containing the vagrant registries
4. Choose the "Access control (IAM)" menu option
5. Choose the "Service principals" side-menu item
6. Choose an existing service principal or create a new one for the "Project" service with the appropriate role 
7. Choose the "Keys" side-menu item
8. Choose to "Generate+" a new key, then take note of the generated ClientID and ClientSecret

```
hcp auth login --client-id=<ClientID> --client-secret=<ClientSecret>
```

Upon successful login, the HCP CLI should then be able to list organizations and projects
```
hcp organizations list
hcp projects list
```

### Access Tokens

After the HCP CLI authenticates to HCP with the security principal key, it will be able to generate access tokens
```
hcp auth print-access-token
```
