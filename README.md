# Linux Server Configuration Project

## To the Reviewer:
* Public IP address: 34.227.31.137

# Steps Taken (In Order)
1. Setup AWS account and Lightsail Server according to Udacity [link](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/ce268cfe-99ec-49be-9326-876375f89a22)
2. Add custom TCP 2200 port on Amazon Lightsaild Networking tab
3. Go to instance page and click "Connect using SSH" to open terminal

### In AWS Terminal
##### Set up firewall
4. sudo apt-get update
5. sudo apt-get upgrade
6. sudo ufw allow 2200/tcp
7. sudo ufw allow 80/tcp
8. sudo ufw allow 123/udp
9. sudo nano /etc/ssh/sshd_config, add ‘Port 2200’ below ‘Port 22’ for now
10. sudo ufw default allow outgoing
11. sudo ufw default deny incoming
12. sudo ufw allow ssh
13. sudo ufw enable

##### Set up user 'grader'
14. sudo apt-get install finger
15. sudo adduser grader
16. sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
17. sudo nano /etc/sudoers.d/
18. added grader ALL=(ALL) NOPASSWD:ALL to /etc/sudoers.d/
19. chmod 700 .ssh
20. Back on the main Lightsail Instance page click on the 'Account page' link toward the bottom
21. Click on the "SSH Keys" tab and download the default private key

### In a separate, local terminal shell
22. ls -ltr /Users/username/Downloads/LightsailDefaultPrivateKey-us-east-1.pem (this was to check the file permissions)
23. cp /Users/username/Downloads/LightsailDefaultPrivateKey-us-east-1.pem ~/.ssh/ (this is just copying the default key and moving it to /.ssh)
24. chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem (change the permissions)
25. ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem ubuntu@34.227.31.137 -p 22 (check to make sure I can ssh into the server as ubuntu)
26. sudo login grader (switch to user grader)

##### Set up new keypair
### Open a new terminal
27. ssh-keygen (this creates a new keypair, place it in ~/.ssh/ and name it)
28. ls -ltr ~/.ssh/ (just to make sure it's there)
29. ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem ubuntu@34.227.31.137 -p 2200 (test ssh into -p 2200)
30. In the ssh terminal, enter 'sudo service ssh restart'


