# Initial Ubuntu 20.04 Server ToDos

## Update/Upgrade All Packages (run with sudo)

```
apt update
apt dist-upgrade
```

## Create Another User w/ Root Privileges (run with sudo)

```
adduser x
adduser x sudo
```

## On Your PC/macOS/Linux run the following command

```
ssh-keygen -o -a 512 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')"
cat "$(hostname)-$(date +'%d-%m-%Y')".pub
```

now copy evertyhing from the public and paste it into the authorized_keys

```
su - x
mkdir .ssh
cd .ssh
vim authorized_keys
```

## Now change SSH Configs to the following (if you like, you can backup the old one)

```
sudo vim /etc/ssh/sshd_config
```

remove everything (backup first) and paste the following

```
Protocol 2                                  	                #Protocol 1 is fundamentally broken
Port 22                                                         #Listening port. Normal 22
StrictModes yes                                                 #Protects from misconfiguration
AuthenticationMethods publickey                                 #Only public key authentication allowed
PubkeyAuthentication yes                                        #Allow public key authentication
HostKey /etc/ssh/ssh_host_ed25519_key                           #Only allow ECDSA pubic key authentication
HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-ed25519  #Host keys the client should accepts
KexAlgorithms curve25519-sha256                                 #Specifies the available KEX (Key Exchange) algorithms
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com    #Specifies the ciphers allowed
MACs hmac-sha2-512,hmac-sha2-512-etm@openssh.com                #Specifies the available MAC alg.
#Only allow incoming ECDSA and ed25519 sessions:
CASignatureAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-ed25519
HostbasedAcceptedKeyTypes ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-ed25519
PubkeyAcceptedKeyTypes sk-ecdsa-sha2-nistp256@openssh.com,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,sk-ssh-ed25519@openssh.com,ssh-ed25519
PermitRootLogin no                                              #Disable root login
MaxAuthTries 5                                                  #Maximum allowed authentication attempts
MaxSessions 2                                                   #Maximum allowed sessions by the user

PasswordAuthentication no                                       #No username password authentication
PermitEmptyPasswords no                                         #No empty password authentcation allowed
IgnoreRhosts yes                                                #Dont read users rhost files
HostbasedAuthentication no                                      #Disable host-based authentication
ChallengeResponseAuthentication no                              #Unused authentication scheme
X11Forwarding no                                                #Disable X11 forwarding
LogLevel VERBOSE                                                #Fingerprint details of failed login attempts
SyslogFacility AUTH                                             #Logging authentication and authorization related commands
UseDNS no                                                       #Client from a location without proper DNS generate a warning in the logs

PermitTunnel no                                                 #Only SSH connection and nothing else
AllowTcpForwarding no                                           #Disablow tunneling out via SSH
AllowStreamLocalForwarding no                                   #Disablow tunneling out via SSH
GatewayPorts no                                                 #Disablow tunneling out via SSH
AllowAgentForwarding no                                         #Do not allow agent forwardng

Banner /etc/issue.net                                           #Show legal login banner
PrintLastLog yes                                                #Show last login

ClientAliveInterval 900                                         #Client timeout (15 minutes)
ClientAliveCountMax 0                                           #This way enforces timeouts on the server side
LoginGraceTime 30                                               #Authenticatin must happen within 30 seconds
MaxStartups 2                                                   #Max concurrent SSH sessions
TCPKeepAlive yes                                                #Do not use TCP keep alive

AcceptEnv LANG LC_*                                             #Allow client to pass locale environment variables
Subsystem sftp /usr/lib/ssh/sftp-server -f AUTHPRIV -l INFO     #Enable sFTP subsystem over SSH
```

Now restart SSHD Service

```
sudo systemctl restart sshd.service
```

check the status

```
sudo systemctl status sshd.service
```

if everything is green (Active: active (running)), the try to connect to the server using your SSH Key

DO NOT CLOSE THIS WINDOW, UNTIL YOU ARE SURE THAT YOU CAN SUCCESSFULLY CONNECT TO YOUR SERVER

If you the following Warning:


```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:....
Please contact your system administrator.
Add correct host key in /Users/[USER_NAME]/.ssh/known_hosts to get rid of this message.
Offending [KEY_TYPE] key in /Users/[USER_NAME]/.ssh/known_hosts:[LINE_NUMBER]
ED25519 host key for [SERVER_PUBLIC_IP_ADDRESS] has changed and you have requested strict checking.
Host key verification failed.
```

that's because we have changed the SSHD config, this can be easily fixed by going to the following the directory:


```
cd ~./ssh
vim authorized_keys
```

and remove the keys, which the [LINE_NUMBER] is mentioned above in the WARNING.

and try to connect to your server again.

## Now we are going to install some packages

### I'm not comfortable using ufw, hence I remove UFW and instead we are going to use firwalld

```
sudo apt -y --purge autoremove ufw
```

### Install useful packages among others firewalld

```
sudo apt update
sudo apt -y dist-upgrade
sudo apt -y install \
build-essential \
curl \
software-properties-common \
vim \
bash-completion \
firewalld \
```

### Checking FirewallD Services

```
sudo firewall-cmd --list-services
```

now, because we have NOT changed the default SSH Port (22), there shouldn't be any problem connecting to server, BUT, if you changed SSH PORT to anything other than 22, then you have to explicitly allow in firewalld, to do this, simply run the following commands:

```
sudo firewalld-cmd --add-port=[PORT_NUMBER]/tcp
sudo firewalld-cmd --runtime-to-permanent
sudo firewalld-cmd --reload
```
