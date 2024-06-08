# Luminosity Labs Notes

## Building the boxes with packer

These notes were last updated to reflect building on macOS Sonoma v14.5, packer v1.11.0, parallels v19.4.0, and
virtualbox v7.0.18 (2024-06-08).

### Current known issues

- rockylinux 9.4 aarch64 currently does not build due to some OS-level package provisioning issue

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

## Using the boxes with vagrant

1. Create a new box
```
vagrant cloud box create -s "AlmaLinux 9 prepared with packer templates from Chef Bento project" --no-private luminositylabsllc/bento-almalinux-9
```

2. Create a new version of the box
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-almalinux-9 20240509.01
```

3. Create a provider for the version of the box:
```
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 parallels  20240509.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 virtualbox 20240509.01
```

4. Upload the box file for the provider:
```
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 parallels  20240509.01 <file>
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 virtualbox 20240509.01 <file>
``` 

5. Release a version:
```
vagrant cloud version release luminositylabsllc/bento-almalinux-9 20240509.01
```

***NOTE:*** vagrant also has a "publish" command which combined all the steps above, but only allows a single provider
to be specified at a time.  The command may be invoked multiple times, once for each provider, but it's not certain
what happens if the non-provider related configuration properties change between invocations.
                                                                                             
```
vagrant cloud publish --box-version v20240509.01 \
                      -d "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --version-description "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" \
                      --no-private -r \
                      -s "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --no-direct-upload luminositylabsllc/bento-almalinux-9 20240509.01 parallels builds/almalinux-9.2-x86_64.parallels.box

vagrant cloud publish --box-version v20240509.01 \
                      -d "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --version-description "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" \
                      --no-private -r \
                      -s "AlmaLinux 9 prepared with packer templates from Chef Bento project" \
                      --no-direct-upload luminositylabsllc/bento-almalinux-9 20240509.01 virtualbox builds/almalinux-9.2-x86_64.virtualbox.box
```

### AlmaLinux

AlmaLinux8 amd64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-almalinux-8  20240509.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-8 parallels   20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-8 parallels  20240509.01 builds/almalinux-8.8-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-almalinux-8 virtualbox  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-8 virtualbox  20240509.01 builds/almalinux-8.8-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-almalinux-8  20240509.01
```

AlmaLinux9 amd64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-almalinux-9  20240509.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 parallels   20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 parallels  20240509.01 builds/almalinux-9.4-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-almalinux-9 virtualbox  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9 virtualbox  20240509.01 builds/almalinux-9.4-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-almalinux-9  20240509.01
```

AlmaLinux9 aarch64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1" luminositylabsllc/bento-almalinux-9-aarch64  20240509.01
vagrant cloud provider create luminositylabsllc/bento-almalinux-9-aarch64 parallels   20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-almalinux-9-aarch64 parallels  20240509.01 builds/almalinux-9.4-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-almalinux-9-aarch64  20240509.01
```

### RockyLinux

RockyLinux8 amd64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-rockylinux-8 20240509.01
vagrant cloud provider create luminositylabsllc/bento-rockylinux-8 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-8 parallels  20240509.01 builds/rockylinux-8.8-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-rockylinux-8 virtualbox 20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-8 virtualbox 20240509.01 builds/rockylinux-8.8-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-rockylinux-8 20240509.01
```

RockyLinux9 amd64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-rockylinux-9 20240509.01
vagrant cloud provider create luminositylabsllc/bento-rockylinux-9 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-9 parallels  20240509.01 builds/rockylinux-9.3-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-rockylinux-9 virtualbox 20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-9 virtualbox 20240509.01 builds/rockylinux-9.3-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-rockylinux-9 20240509.01
```

RockyLinux9 aarch64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1" luminositylabsllc/bento-rockylinux-9-aarch64 20240509.01
vagrant cloud provider create luminositylabsllc/bento-rockylinux-9-aarch64 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-rockylinux-9-aarch64 parallels  20240509.01 builds/rockylinux-9.3-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-rockylinux-9-aarch64 20240509.01
```

### Ubuntu

Ubuntu 20.04 amd64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-ubuntu-20.04 20240509.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-20.04 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-20.04 parallels  20240509.01 builds/ubuntu-20.04-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-ubuntu-20.04 virtualbox 20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-20.04 virtualbox 20240509.01 builds/ubuntu-20.04-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-20.04 20240509.01
```

Ubuntu 20.04 aarch64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1" luminositylabsllc/bento-ubuntu-20.04-arm64 20240509.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-20.04-arm64 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-20.04-arm64 parallels  20240509.01 builds/ubuntu-20.04-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-20.04-arm64 20240509.01
```

Ubuntu 22.04 amd64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1, virtualbox v7.0.18" luminositylabsllc/bento-ubuntu-22.04 20240509.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-22.04 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-22.04 parallels  20240509.01 builds/ubuntu-22.04-x86_64.parallels.box
vagrant cloud provider create luminositylabsllc/bento-ubuntu-22.04 virtualbox 20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-22.04 virtualbox 20240509.01 builds/ubuntu-22.04-x86_64.virtualbox.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-22.04 20240509.01
```

Ubuntu 22.04 aarch64
```
vagrant cloud version create -d "box v20240509.01, packer v1.10.3, parallels v19.3.1" luminositylabsllc/bento-ubuntu-22.04-arm64 20240509.01
vagrant cloud provider create luminositylabsllc/bento-ubuntu-22.04-arm64 parallels  20240509.01
vagrant cloud provider upload --no-direct luminositylabsllc/bento-ubuntu-22.04-arm64 parallels  20240509.01 builds/ubuntu-22.04-aarch64.parallels.box
vagrant cloud version release luminositylabsllc/bento-ubuntu-22.04-arm64 20240509.01
```