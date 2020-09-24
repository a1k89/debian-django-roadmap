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

1. Connect to server through SSH. Then:
```
sudo adduser a1
sudo usermod -aG sudo a1

```
Logout from server. And then copy ssh-key to remote server:
```
ssh-copy-id a1@domain 
```
2. Connect to server via key. Then:
```
sudo mcedit /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
Port 2022

sudo service ssh restart

```
Now we may connect to server withoud password. We change port number at 2022.
Now we may create alias for ssh connection. Please exit from server. Then:
```
exit();
sudo mcedit /ssh/ssh_config
Host server_host
Hosname demo
User a1
Port 2022

```

Finally, connect to server again via:
```
ssh demo
```
