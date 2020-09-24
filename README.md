# Deploy Django on Debian server

1. Connect to server via root and create new user
2. Setup SSH
3. Install must have packages
4. Install python
5. Clone the project
6. Install and configure PostgreSQL
7. Install packages from requirements.txt
8. Configure nginx, gunicorn, supervisor
9. Configre static and media
10. (optional) SSL for domain

Simplifications:
1. Remote server host: example.com
2. Remote user: a1


1. Connect to server through SSH. Then
```
sudo adduser a1
sudo usermod -aG sudo a1
```

Local machine:
```
ssh-copy-id a1@example.com
sudo mcedit /ssh/ssh_config
Host example.com
Hostname example
User a1
Port 2022
```

2. Connect to server via key and then:
```
sudo mcedit /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
Port 2022
sudo service ssh restart
```
Now we may connect to server via `ssh example` withoud password and we are ready to step3. 

