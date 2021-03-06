Your development box has been created. Likely you will not have a graphical desktop at this point. Restart the box using the following command and you should see a graphical desktop. (Cloud providers such as AWS see below for VNC instructions.)

  vagrant reload

There are several ways to configure the box for your environment. To apply configuration changes, run the following command:

  vagrant provision

The `config.yaml` file has several machine size configurations. The default is medium, you can change that by changing the `use` entry in `config.yaml` or setting the environment variable `DEV_PROFILE` to the configuration name. Using the environment variable will allow you to commit the `config.yaml` file to a shared repository.

```yaml
---
configs:
    use: 'medium'
    small:
        memory: 2048
        cores: 2
    medium:
        memory: 4096
        cores: 2
    large:
        memory: 8192
        cores: 4
    # settings in 'default' are applied to all configurations
    default:
        # Proxy URL that will be configured in various places
        proxy_url: http://proxy:8123
        # Domains excluded from the proxy, comma separated list
        proxy_excludes: .internal.net,.dmz.net
        # Limit DNS resolution to IPV4
        ipv4only: true
        # Force search domain in /etc/resolv.conf
        search_domain: company.com
        # Limit DNS resolution to IPV4
        ipv4only: true
        # Populates git config
        user_name: Droopy Dog
        user_email: droppy@dogpound.nil
        # Set the number of monitors, defaults to let the provider determine it
        monitors: 1
        # Optional Timezone, pulled from host machine if not specified
        timezone: America/Chicago
```

## Proxies
The variables `HTTP_PROXY`, `HTTPS_PROXY` and `NO_PROXY` are recognized and applied throughout the box. If you want to use something different, the `config.yaml` file can be used to override the environment variables.

## SSH Keys
Any SSH public/private key pairs found in the `/vagrant` guest directory, which is usually mapped to the directory containing the `Vagrantfile`, will be searched for SSH keys. Any files ending with `.pub` are found and if there is a matching file without `.pub`, the key will be added as an identity in SSH. The SSH config points to the `/vagrant` directory, so if this directory is unmounted the keys will not be found. SSH usually handles missing files gracefully. Important exclusions to the search path are the `environments` directory and any directory beginning with a `.`.

## SSH User
The `vagrant` user is the primary user on the box. SSH will be configured to use the host machine user account. The environment variables `USER` and `USERNAME` are checked. The profile in `config.yaml` may include a `username` value to specify the user name.

## Git User
If the host computer has defined a name and email for git commits, git will be configured in the box the same way. The name and email for commits can be specified in `config.yaml` using `user_name` and `user_email` keys. See example above.

## CA Certificates
CA certificates can be added to the system wide trust store by placing the files similarly to the above SSH keys. The files must be in PEM format and have an extension of `.pem`, `.crt` or `.cer`.

## Source Code Repositories
You can configure source code repositories to be checked out as part of provisioning. The puppet module at https://forge.puppet.com/puppetlabs/vcsrepo is used for checking out and it supports several versioning systems. The repo information is stored in `repos.yaml` in the root of this repo. Each repo has a name which is checked out into `/home/vagrant/Workspace`, and the keys under it specify the source URL, branch, etc.

```yaml
---
spring-boot:
  provider: git
  source: https://github.com/spring-projects/spring-boot.git
  revision: 1.5.x

qgroundcontrol:
  provider: git
  source: https://github.com/mavlink/qgroundcontrol.git
```

## AWS, Azure
Running the box on the cloud provides a VNC server. You need to use SSH tunneling to access the server. Assuming you have `vncviewer` installed with either the TightVNC or TigerVNC package:

```shell
$ ssh -t -L 5910:localhost:5900 centos@ec2-NNN.amazonaws.com
$ vncviewer localhost:5910
```

If you want to use a different VNC client, point it to `localhost:5900`.

_DO NOT_ expose port 5900 by adjusting firewall rules. The VNC server has no password. If you want direct access to VNC then you must change the VNC configuration. SSH tunneling is preferred.

Sometimes after starting a new EC2 instance, the IP and hostname will be changed while it is running. It seems the VNC systemctl service doesn't handle this well and VNC won't be started. Restart the EC2 instance to fix it.
