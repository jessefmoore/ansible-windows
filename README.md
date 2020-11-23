## [![Nabla](http://albandrieu.com/nabla/index/assets/nabla/nabla-4.png)](https://github.com/AlbanAndrieu) roles/alban_andrieu_windows

This file was generated by Ansigenome. Do not edit this file directly but instead have a look at the files in the ./meta/ directory.

[![License](http://img.shields.io/:license-apache-blue.svg?style=flat-square)](http://www.apache.org/licenses/LICENSE-2.0.html)
[![Branch](http://img.shields.io/github/tag/AlbanAndrieu/ansible-windows.svg?style=flat-square)](https://github.com/AlbanAndrieu/ansible-windows/tree/master)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-alban.andrieu.windows-660198.svg?style=flat)](https://galaxy.ansible.com/alban.andrieu/windows)
[![Platforms](http://img.shields.io/badge/platforms-windows-lightgrey.svg?style=flat)](#)


Goal of this project is to launch ansible script using [pywinrm](https://pypi.python.org/pypi/pywinrm).
A VagrantFile is downloading a windows 2012 server VM that will be hosted on VirtualBox.
Then we are launching Ansible script in order to set up this VM.

VM was taken from
------------------

https://vagrantcloud.com/opentable/boxes/win-2012r2-standard-amd64-nocm/versions/1.0.0/providers/virtualbox.box

# Table of contents

<!-- toc -->

<!-- tocstop -->

## Actions

- Ensures that windows is installed

WARNING : In inventory file, please use ansible_ssh_user and ansible_ssh_pass instead of ansible_user ansible_password, because of vault overriden values

Usage example
-------------

    - name: Install windows
      connection: local
      hosts: windows

      roles:
        - role: windows

### Requirements

On Ubuntu, where VirtualBox and Vagrant are installed, do not forge to do the following :
sudo pip install https://github.com/diyan/pywinrm/archive/df049454a9309280866e0156805ccda12d71c93a.zip --upgrade

It is working with the following version :

# Os is an Ubuntu 18.04

$ python -V
Python 2.7.3
$ pip -V
pip 1.4.1 from /usr/local/lib/python2.7/dist-packages (python 2.7)

$ VBoxManage --version
4.3.28r100309

$ vagrant --version
Vagrant 2.3.1.0

$ vagrant plugin list
winrm (1.1.3)
vagrant-login (1.0.1, system)
vagrant-share (1.1.0, system)

$ ansible --version
ansible 1.7.2

Ansible 2.5.0 is required on order to have win_copy working on Windows 7 and Windows Server 2016

Check winrm in target host

```
winrm id
winrm get winrm/config
```

For older version of Windows, please do

```
Set-Item WSMan:\localhost\Shell\MaxMemoryPerShellMB 5000
Set-Item WSMan:\localhost\Plugin\Microsoft.PowerShell\Quotas\MaxMemoryPerShellMB 5000
```

Restart-Service winrm

```
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="0"}'
```

On the windows VM :

```
powershell -File upgrade_to_ps3.ps1
```

```
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%systemdrive%\chocolatey\bin
cinst powershell
```

```
choco install powershell
choco upgrade powershell
```

```
powershell -File ConfigureRemotingForAnsible.ps1
```


Test winrm
-----------

See [windows_winrm](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html)

See [common-winrm-issues](https://docs.ansible.com/ansible/devel/user_guide/windows_setup.html#common-winrm-issues)

## From windows

### Local user

$ winrm identify -u:myuser -p:Mypass123! -r:http://targetHost:5985

### Domain user

$ winrm identify -u:MISYSROOT\aandrieu -p:Mypass123! -r:http://targetHost:5985

### Test out HTTP

winrs -r:http://server:5985/wsman -u:Username -p:Password ipconfig

### Test out HTTPS (will fail if the cert is not verifiable)

winrs -r:http://server:5985/wsman -u:Username -p:Password -ssl ipconfig

## From unix

### User prompted for REALM name and KDC for Kerberos

$ sudo apt-get install python-dev libkrb5-dev krb5-user

### Python WinRM module

$ sudo pip install pyOpenSSL --upgrade
$ sudo pip install "pywinrm>=0.2.2"

### Ignore warnings about maj_stat

$ sudo pip install kerberos

### Kerberos and CredSSP

$ sudo pip install "pywinrm[kerberos]"
$ sudo pip install "pywinrm[credssp]"
$ sudo pip install "requests-credssp" "requests-kerberos"

### Get xmllint for pretty print of SOAP response

$ sudo apt-get install libxml2-utils -y

### Replace 'targetHost' with the target Windows host

$ curl --header "Content-Type: application/soap+xml;charset=UTF-8" --header "WSMANIDENTIFY: unauthenticated" http://targetHost:5985/wsman --data '&lt;s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:wsmid="http://schemas.dmtf.org/wbem/wsman/identity/1/wsmanidentity.xsd"&gt;&lt;s:Header/&gt;&lt;s:Body&gt;&lt;wsmid:Identify/&gt;&lt;/s:Body&gt;&lt;/s:Envelope&gt;' | xmllint --format -

### Basic authentication is not enabled by default on a Windows host but can be enabled by running the following in PowerShell

$ Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true

$ Set-Item -Path WSMan:\localhost\Service\Auth\Certificate -Value $true
$ New-SelfSignedCertificate
$ (Get-Service -Name winrm).Status

$ .\ConfigureRemotingForAnsible.ps1 -ForceNewSSLCert

On the windows VM :

[Install PowerShell](https://github.com/AlbanAndrieu/ansible-windows/blob/master/files/upgrade_to_ps3.ps1)
[Configure remoting](https://github.com/AlbanAndrieu/ansible-windows/blob/master/files/ConfigureRemotingForAnsible.ps1)
[Disable password](http://www.tenniswood.co.uk/technology/windows/how-to-disable-password-expiration-for-windows-server-2012/)


Automate jenkins service
------------------------

See [jenkins-agent.ps1](https://github.com/jenkinsci/docker-inbound-agent/blob/master/jenkins-agent.ps1)

Change default JDK if wrong
Replace Path Environment variable from C:\ProgramData\Oracle\Java\javapath by %JAVA_HOME%\bin
Replace regedit Registry key 'Software\JavaSoft\Java Runtime Environment'\CurrentVersion' but 1.7
Replace regedit Registry key 'Software\JavaSoft\Java Development Kit'\CurrentVersion' but 1.7

### Check the java web start default JDK

$ javaws -viewer

### Run the java web start by hand if the JDK is not right

$ javaws  "slave-agent.jnlp"

### Add -noCertificateCheck to the jenkins-slave.xml in the jenkins directory if missing

### Generate id_rsa from MSYS2

Copy it from C:\msys64\home\mysuser or C:\tools\msys64\home\mysuser to the user
Add the key to Bitbucket
Test doing git clone ssh://stash:7999/test/repo.git

### Log on

Make sure in the jenkins you have Log on -> Log on as -> Local System account
BUT NOT Change jenkins service to start as Log on as -> This account and use my user, otherwise is do not reconnect automatically after reboot

### Recovery

Reset fail count after: 1 days
Restart service after: 60 minutes

### Documentation

More information about `alban.andrieu.windows` can be found in the
TODO [official alban.andrieu.windows documentation](https://docs.debops.org/en/latest/ansible/roles/ansible-windows/docs/).


### Role variables

List of default variables available in the inventory:

```YAML
windows_enabled: yes                       # Enable module

#ansible_ssh_user: vagrant
#ansible_ssh_pass: vagrant
#target port
#ansible_ssh_port: 5986
#local port
#ansible_ssh_port: 55985
ansible_connection: winrm
```


### Detailed usage guide

Run the following command :

`ansible-playbook -i hosts -c local -v windows.yml -vvvv --ask-sudo-pass | tee setup.log`

### Testing
```shell
$ ansible-galaxy install alban.andrieu.windows
$ vagrant up
```

Ansible lint
------------

```shell
$ git add tasks/pacman.yml # First add your file, then
$ pre-commit run ansible-lint
```

### Contributing

The [issue tracker](https://github.com/AlbanAndrieu/ansible-windows/issues) is the preferred channel for bug reports, features requests and submitting pull requests.

For pull requests, editor preferences are available in the [editor config](.editorconfig) for easy use in common text editors. Read more and download plugins at <http://editorconfig.org>.

In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests and examples for any new or changed functionality.

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

### Authors and license

`roles/alban_andrieu_windows` role was written by:

- [Alban Andrieu](nabla.mobi) | [e-mail](mailto:alban.andrieu@free.fr) | [Twitter](https://twitter.com/AlbanAndrieu)

License
-------

- License: [GPLv3](https://tldrlegal.com/license/gnu-general-public-license-v3-%28gpl-3%29)

### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/AlbanAndrieu/ansible-windows/issues)!

***

This role is part of the [Nabla](https://github.com/AlbanAndrieu) project.
README generated by [Ansigenome](https://github.com/nickjj/ansigenome/).

***

Alban Andrieu

[linkedin](fr.linkedin.com/in/nabla/)
