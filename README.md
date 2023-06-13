## FTP Server
### Initial setup of PROFTPD
Setting our hostname
```shell
sudo hostnamectl set-hostname ftp1
```

Updating our repositories
```shell
sudo apt update
```

Now we can begin the installation
```shell
sudo apt -y install proftpd
```

If we want to implement tls/sftp we'll also install the crypto mod
```shell
sudo apt -y install proftpd-mod-crypto
```

Now we can check if the service is running through systemctl
```shell
sudo systemctl status proftpd
```
![](https://hackmd.io/_uploads/HJuZNumPn.png)

Logs with journalctl
```shell
journalctl -u proftpd
```

Correct configuration syntax with proftpd -t
```shell
proftpd -t
```

### User management
PROFTPD will use the regular UNIX users as the method of authentication.
We can create a new user with this command:
```shell
sudo adduser ftp1
```
![](https://hackmd.io/_uploads/B1jUQO7Dh.png)

We'll log with the user
```shell
su ftp1
```

Place a few things to play with in it's home directory
![](https://hackmd.io/_uploads/r15Em_Xvh.png)

You can quit this user's session with the command:
```shell
exit
```
![](https://hackmd.io/_uploads/Hyx0X_QP3.png)

We can also modify their home directory with this command where **/var/www/** is the path to the new home.
```shell
sudo usermod -m -d /var/www/ username
```

### Use of FTP
We can log from our network using the command:
```shell
ftp ftp1@192.168.174.30
```
You'll be prompted for the password
![](https://hackmd.io/_uploads/SktI4_Xw2.png)

Once inside you can type **help** to get the commands
![](https://hackmd.io/_uploads/Sy_0EuQPn.png)

**dir** or **ls** is used to see what's inside our directory
![](https://hackmd.io/_uploads/BJM3VFXv2.png)

**delete* erases a file
![](https://hackmd.io/_uploads/SyceSY7v2.png)

**cd** to change directories
![](https://hackmd.io/_uploads/r174HK7v2.png)

**put** uploads a file from the host directory
![](https://hackmd.io/_uploads/SJaFHY7w3.png)

**get** downloads a file
![](https://hackmd.io/_uploads/SyPhBK7D3.png)

**exit** to leave ftp

### Auto-signed certificates
To enable FTPS and SFTP we need encryption, we can get an auto-signed certificates through one of these commands:
1. Install SSL in case we don't have it
```shell
sudo apt-get install openssl -y
```
2. **Gencert**
```shell
sudo proftpd-gencert
```
![](https://hackmd.io/_uploads/rydYLK7Ph.png)

or

2. **OpenSSL**
```shell
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/proftpd.key -out /etc/ssl/certs/proftpd.crt
```

Change the permissions on these files
```shell
sudo chmod 0600 /etc/ssl/private/proftpd.key
sudo chmod 0600 /etc/ssl/certs/proftpd.crt
```

### Configuration files
#### proftpd.conf
This is the general configuration file, we'll modify it using the following command:
```shell
sudo nano /etc/proftpd/proftpd.conf
```

##### Enable TLS
Uncomment this line
```shell
Include /etc/proftpd/tls.conf
```
![](https://hackmd.io/_uploads/H1L0vFQv2.png)

##### Enable SFTP
Uncomment this to enable SFTP:
```shell
Include /etc/proftpd/sftp.conf
```
![](https://hackmd.io/_uploads/By-yuFXD3.png)

##### Anonymous users
We could activate anonymous users uncommenting the following lines:
```shell
 <Anonymous ~ftp>
   User ftp
   Group nogroup
   # We want clients to be able to login with "anonymous" as well as "ftp"
   UserAlias anonymous ftp
   # Cosmetic changes, all files belongs to ftp user
   DirFakeUser on ftp
   DirFakeGroup on ftp

   RequireValidShell off

   # Limit the maximum number of anonymous logins
   MaxClients 10

   # We want 'welcome.msg' displayed at login, and '.message' displayed
   # in each newly chdired directory.
#   DisplayLogin welcome.msg
#   DisplayChdir .message
#
#   # Limit WRITE everywhere in the anonymous chroot
#   <Directory *>
#     <Limit WRITE>
#       DenyAll
#     </Limit>
#   </Directory>
#
#   # Uncomment this if you're brave.
#   # <Directory incoming>
#   #   # Umask 022 is a good standard umask to prevent new files and dirs
#   #   # (second parm) from being group and world writable.
#   #   Umask022  022
#   #   <Limit READ WRITE>
#   #     DenyAll
#   #     </Limit>
#   #       <Limit STOR>
#   #         AllowAll
#   #     </Limit>
#   # </Directory>
#
 </Anonymous>
```
#### sftp.conf
Modify the file
```shell
sudo nano /etc/proftpd/sftp.conf
```

Uncomment these lines
```shell
<IfModule mod_sftp.c>
SFTPEngine     on
Port           2222
SFTPLog        /var/log/proftpd/sftp.log
SFTPHostKey /etc/ssh/ssh_host_rsa_key
</IfModule>
```
#### tls.conf
Modify the file
```shell
sudo nano /etc/proftpd/sftp.conf
```

To get TLS working we have to uncomment these lines
```shell
<IfModule mod_tls.c>
TLSEngine                               on
TLSLog                                  /var/log/proftpd/tls.log
TLSProtocol                             SSLv23
TLSRSACertificateFile                   /etc/ssl/certs/proftpd.crt
TLSRSACertificateKeyFile                /etc/ssl/private/proftpd.key
TLSOptions                              AllowClientRenegotiations
TLSRequired                             on
</IfModule>
```
#### modules.conf
##### TLS Activation
Uncomment these lines
```shell
# Install proftpd-mod-crypto to use this module for TLS/SSL support.
LoadModule mod_tls.c
# Even these modules depend on the previous one
LoadModule mod_tls_fscache.c
LoadModule mod_tls_shmcache.c
```
![](https://hackmd.io/_uploads/B14dcKmPh.png)
##### SFTP Activation
Uncomment these lines
```shell
# Install proftpd-mod-crypto to use this module for SFTP support.
LoadModule mod_sftp.c
LoadModule mod_sftp_pam.c
```

### TLS testing
Once the required configuration files of proftpd.conf, tls.conf and modules.conf have been modified we can restart and check our service
```shell
sudo systemctl restart proftpd.service
sudo systemctl status proftpd.service
```
![](https://hackmd.io/_uploads/rJym3YQP2.png)

Now since we enabled TLSRequired On we won't be able to connect through normal ftp
![](https://hackmd.io/_uploads/BkFeatXw2.png)

We can install ftp-ssl to connect securely 
```shell
sudo apt install ftp-ssl
```

Now we can connect using the command ftp-ssl **server name**
```shell
ftp-ssl ftp1
```

We'll be prompted by the user name and password
![](https://hackmd.io/_uploads/BJrH15Qvh.png)

### SFTP Testing
We'll test our connection with the sftp command:
```shell
sftp ftp1@192.168.174.30
```

We'll accept the key by typing **yes** and enter the password
![](https://hackmd.io/_uploads/rkrPOqXDh.png)
