---
title: setup sftp account and two differences authentication logins
date: 2016-06-17 15:18:15
tags:
---

# sftp is a way to use ftp on a linux server. it use the ssh account to connect the server, we can setup the sftp account and provide differences authentication


## create sftp account without shell permission

```
sudo adduser --shell /bin/false sftpuser
```

then set the password for the user
```
sudo passwd sftpuser
```

## create sftp directory for the user, and give the permission
/var/sftp/files here
```
sudo mkdir -p /var/sftp/files
sudo chown sftpuser:sftpuser /var/sftp/files

sudo chown root:root /var/sftp
sudo chmod 755 /var/sftp

```


## create public key

we need to create the files in the user's home directory, and generate the authentication keys

```
mkdir /home/sftpuser/.ssh
```

generate keys by `ssh-keygen -t rsa` and point the file location  to `/home/sftpuser/.ssh/id_rsa`


set authorized keys and permission

```
cd .ssh
touch authorized_keys
cat id_rsa.pub >> authorized_keys
cd ..
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

save the generate private key to your compucter and give 600 permission

```
cat /home/sftp/.ssh/id_rsa > sftp.pem
chmod 600 sftp.pem
```


## update sftp user config to enable password and public key authentication

edit file at /etc/ssh/sshd_config, append below lines

```
Match User sftpuser
	ForceCommand internal-sftp
	PasswordAuthentication yes
	ChrootDirectory /var/sftp
	PermitTunnel no
	AllowAgentForwarding no
	AllowTcpForwarding no
	X11Forwarding no
	RSAAuthentication yes
	PubkeyAuthentication yes
	AuthorizedKeysFile /home/sftpuser/.ssh/authorized_keys
```


set PasswordAuthentication to yes to enable password authentication,
set PubkeyAuthentication to yes to enable public key authentication,


and then restart sshd services
`sudo service sshd restart`


## test sftp connection by command line 

password authentication
```
sftp  sftpuser@<server_host>
```


public key  authentication
```
sftp -i sftp.pem sftpuser@<server_host>
```


