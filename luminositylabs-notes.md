# Luminosity Labs Notes

## Building the boxes with packer

These notes were last updated to reflect building on macOS Sequoia v15.4.1, packer v1.12.0, parallels v20.3.0, and
 virtualbox v7.1.8 (2025-04-18).

### Current known issues

- [2025-04-18] MacOS v15.4.1 / Parallels v20.x

The build processes using packer seem to become problematic for parallels after an upgrade of parallels via homebrew.
 This may be caused by some combination of homebrew uninstalling parallels and reinstalling the newer version from
 scratch which may not properly take into account a need for shutting down parallels background processes or doing
 something special for handling MacOS security measures for 3rd party software.  If packer builds for parallels images
 stop building properly, a possible fix is to invoke "prlsrvctl shutdown" to stop parallels background processes, then
 start parallels from the MacOS launcher and follow any instructions for allowing/enabling parallels.
 

- [2025-04-18] MacOS v15.4.1 / virtualbox v7.1.6 / AlmaLinux & RockyLinux

The AlmaLinux and RockyLinux boxes build successfully for VirtualBox amd64 platform, but deploying the box with vagrant
 has problems with the VirtualBox Guest Additions.  Getting the additions to work involves installing RPMs after initial
 creation of the VM by vagrant and then reloading the VM.
```
vagrant up --provider virtualbox
vagrant ssh -c "sudo systemctl status vboxadd"
vagrant ssh -c "sudo dnf install -y kernel-headers kernel-devel"
vagrant reload
vagrant ssh -c "sudo systemctl status vboxadd"
```

- [2025-04-18] MacOS v15.3.1 / virtualbox v7.1.6 / Ubuntu

The Ubuntu boxes build successfully for Virtualbox, but the Virtualbox Guest Additions are not able to load kernel
 modules in some cases, due to situations where the guest additions setup needed packages which were not installed.
 The bento packer scripts may not install them or remove them before packaging the box in order to save space.  The
 following is an example of installing the packages and triggering a guest additions setup to rebuild and load the
 kernel modules so that the guest additions services run.

```
vagrant up --provider virtualbox
vagrant ssh -c "sudo systemctl status vboxadd"
vagrant ssh -c "sudo apt-get install -y linux-headers-generic gcc make"
vagrant reload
vagrant ssh -c "sudo systemctl status vboxadd"
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
time packer build -only=parallels-iso.vm -var-file=os_pkrvars/ubuntu/ubuntu-22.04-aarch64.pkrvars.hcl ./packer_templates
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
vagrant cloud version create -d "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" luminositylabsllc/bento-almalinux-9 20250418.01
```

3. Create a provider for the version of the box:
```
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 parallels  20250418.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 virtualbox 20250418.01
```

4. Upload the box file for the provider:
```
vagrant cloud provider upload luminositylabsllc/bento-almalinux-9 parallels  20250418.01 amd64 <file>
vagrant cloud provider upload luminositylabsllc/bento-almalinux-9 virtualbox 20250418.01 amd64 <file>
``` 

5. Release a version:
```
vagrant cloud version release luminositylabsllc/bento-almalinux-9 20250418.01
```

***NOTE:*** vagrant also has a "publish" command which combined all the steps above, but only allows a single provider
to be specified at a time.  The command may be invoked multiple times, once for each provider, but it's not certain
what happens if the non-provider related configuration properties change between invocations.
                                                                                             
```
vagrant cloud publish --architecture amd64 \
                      --description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
                      --no-private --no-release \
                      --short-description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      luminositylabsllc/bento-almalinux-9 20250418.01 parallels builds/almalinux-9.5-x86_64.parallels.box

vagrant cloud publish --architecture amd64 \ 
                      --description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
                      --no-private --no-release  \
                      --short-description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      luminositylabsllc/bento-almalinux-9 20250418.01 virtualbox builds/almalinux-9.5-x86_64.virtualbox.box
```

### AlmaLinux

AlmaLinux8 amd64
```
for P in parallels virtualbox; do
    time vagrant cloud publish \
        --architecture amd64 \
        --description "AlmaLinux 8 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "AlmaLinux 8 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-almalinux-8 20250418.01 ${P} builds/almalinux-8.10-x86_64.${P}.box   
done
vagrant cloud version release luminositylabsllc/bento-almalinux-8 20250418.01
```

AlmaLinux9 amd64
```
for P in parallels virtualbox; do
    time vagrant cloud publish \
        --architecture amd64 \
        --description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-almalinux-9 20250418.01 ${P} builds/almalinux-9.5-x86_64.${P}.box   
done
vagrant cloud version release luminositylabsllc/bento-almalinux-9 20250418.01
```

AlmaLinux9 aarch64
```
for P in parallels; do
    time vagrant cloud publish \
        --architecture arm64 \
        --description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-almalinux-9-aarch64 20250418.01 ${P} builds/almalinux-9.5-aarch64.${P}.box
done
vagrant cloud version release luminositylabsllc/bento-almalinux-9-aarch64 20250418.01
```

### RockyLinux

RockyLinux8 amd64
```
for P in parallels virtualbox; do
    time vagrant cloud publish \
        --architecture amd64 \
        --description "RockyLinux 8 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "RockyLinux 8 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-rockylinux-8 20250418.01 ${P} builds/rockylinux-8.10-x86_64.${P}.box   
done
vagrant cloud version release luminositylabsllc/bento-rockylinux-8 20250418.01
```

RockyLinux9 amd64
```
for P in parallels virtualbox; do
    time vagrant cloud publish \
        --architecture amd64 \
        --description "RockyLinux 9 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "RockyLinux 9 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-rockylinux-9 20250418.01 ${P} builds/rockylinux-9.5-x86_64.${P}.box   
done
vagrant cloud version release luminositylabsllc/bento-rockylinux-9 20250418.01
```

RockyLinux9 aarch64
```
for P in parallels; do
    time vagrant cloud publish \
        --architecture arm64 \
        --description "RockyLinux 9 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "RockyLinux 9 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-rockylinux-9-aarch64 20250418.01 ${P} builds/rockylinux-9.5-aarch64.${P}.box
done
vagrant cloud version release luminositylabsllc/bento-rockylinux-9-aarch64 20250418.01
```

### Ubuntu

Ubuntu 22.04 amd64
```
for P in parallels virtualbox; do
    time vagrant cloud publish \
        --architecture amd64 \
        --description "Ubuntu 22.04 amd64 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "Ubuntu 22.04 amd64 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-ubuntu-22.04 20250418.01 ${P} builds/ubuntu-22.04-x86_64.${P}.box   
done
vagrant cloud version release luminositylabsllc/bento-ubuntu-22.04 20250418.01
```

Ubuntu 22.04 aarch64
```
for P in parallels; do
    time vagrant cloud publish \
        --architecture arm64 \
        --description "Ubuntu 22.04 arm64 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "Ubuntu 22.04 arm64 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-ubuntu-22.04-arm64 20250418.01 ${P} builds/ubuntu-22.04-aarch64.${P}.box
done
vagrant cloud version release luminositylabsllc/bento-ubuntu-22.04-arm64 20250418.01
```

Ubuntu 24.04 amd64
```
for P in parallels virtualbox; do
    time vagrant cloud publish \
        --architecture amd64 \
        --description "Ubuntu 24.04 amd64 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "Ubuntu 24.04 amd64 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-ubuntu-24.04 20250418.01 ${P} builds/ubuntu-24.04-x86_64.${P}.box   
done
vagrant cloud version release luminositylabsllc/bento-ubuntu-24.04 20250418.01
```

Ubuntu 24.04 aarch64
```
for P in parallels; do
    time vagrant cloud publish \
        --architecture arm64 \
        --description "Ubuntu 24.04 arm64 prepared with packer templates from Chef Bento project" \
        --version-description "box v20250418.01, packer v1.12.0, parallels v20.3.0, virtualbox v7.1.8" \
        --no-private --no-release \
        --short-description "Ubuntu 24.04 arm64 prepared with packer templates from Chef Bento project" \
            luminositylabsllc/bento-ubuntu-24.04-arm64 20250418.01 ${P} builds/ubuntu-24.04-aarch64.${P}.box
done
vagrant cloud version release luminositylabsllc/bento-ubuntu-24.04-arm64 20250418.01
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

### Using vagrant to publish boxes to HCP Vagrant Registry

The Vagrant CLI can be used to publish vagrant box to the HCP Vagrant Registry using the `vagrant cloud` suites of
 commands.

```
export HCP_TOKEN=$(hcp auth print-access-token)
export VAGRANT_CLOUD_TOKEN=";$(hcp auth print-access-token)"
```
