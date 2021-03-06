# Contributing

Fork this repo, create a branch, and submit a PR. CI is done using CircleCI by using a Docker container to build the Vagrant box. You can build your fork to verify your changes pass.

## Update Puppet Modules

Update puppet modules from the `Puppetfile`:
```shell
$ librarian-puppet install --path=environments/dev/modules --strip-dot-git
```

If the `zanloy-vim` module is updated, the URL to vim-pathogen needs to be changed to a github.com URL to make it past some filtering proxies. Change `https://tpo.pe/pathogen.vim` to `https://raw.githubusercontent.com/tpope/vim-pathogen/v2.4/autoload/pathogen.vim` in the file `environments/dev/modules/vim/manifests/pathogen.pp`.

If the `paulosuzart-sdkman` module is updated, the `su` commands in the file `environments/dev/modules/sdkman/manifests/package.pp` need to be fixed to have a space after the ` - ` to enable a login shell.

## Packer Builds

Packer builds are available for the following providers:

* VirtualBox
* VMware
* Parallels
* AWS
* qemu
* Azure (VHD only, Vagrant box is not supported)

The VMs are large, 10-12GB uncompressed. You'll likely need to build them individually.

* packer build -only=virtualbox-iso centos.json
* packer build -only=vmware-iso     centos.json
* packer build -only=parallels-iso  centos.json
* packer build -only=amazon-ebs     centos.json
* packer build -only=qemu           centos.json
* packer build -only=azure-arm      centos.json

There are environment variables needed for building. If you aren't using a specific build, the variable is required, but a dummy value will do.

* `VAGRANT_CLOUD_TOKEN` - vagrantcloud.com token for publishing Vagrant boxes
* `AWS_ACCESS_KEY` - Access key for AWS EC2 allowing read/write access (not admin) to EC2
* `AWS_SECRET_KEY` - Secret key for AWS
* `AZURE_CLIENT_ID` - Client ID for Azure
* `AZURE_CLIENT_SECRET` - Secret for Azure

## Building in Azure

If you'd like to use a VM in the cloud to build the boxes, Azure supports nested virtualization. The `builder` directory has a Packer `builder.json` file to build the VM with Packer, VirtualBox and QEMU.

### Configure your Azure Account

Azure needs several configurations before you can run the Packer build. Run the `azure-setup.sh` script in this directory. The output is suitable for a Packer variables file. We'll assume you've created a `vars.json` with the output.

### Run Packer

```shell
$ packer build -var-file=vars.json builder.json
```

### Create the VM

You'll need to follow the instructions provided in this [Packer GitHub issue](https://github.com/Azure/packer-azure/issues/201) to create a VM based on the VHD image.
