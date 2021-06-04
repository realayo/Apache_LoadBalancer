# Load Balancer Solution With Apache
We will be using servers we configured for our [tooling website](https://github.com/realayo/Tooling_Website_Solution) which consists of:

1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server

## Configure Apache As A Load Balancer
1. Set up a new Ubuntu server on EC2.
1. Install Apache Load Balancer on the server and configure it to point traffic coming to Load Balancer to both Web Servers:
```
# Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

# Enable the following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```
3. Restart apache2 service
```
sudo systemctl restart apache2
```
4. Make sure apache is running
```
sudo systemctl status apache2
```
![](https://user-images.githubusercontent.com/18899718/120818835-98e58e00-c518-11eb-91d0-edd367c9f55a.png)
5. Configure load balancing
```
sudo vi /etc/apache2/sites-available/000-default.conf
```
6. Add this configuration between section  `<VirtualHost *:80>` and `</VirtualHost>` section:
```
<Proxy "balancer://mycluster">
        BalancerMember <WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
        BalancerMember <WebServer2-Private-IP-Address> loadfactor=5 timeout=1
          roxySet lbmethod=bytraffic
        # ProxySet lbmethod=byrequests
 </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/ 
```
7. Verify that our configuration works by accessing our LB’s public IP address or Public DNS name from the web browser
![](https://user-images.githubusercontent.com/18899718/120820451-2e355200-c51a-11eb-9fa9-a9bca1cda8eb.png)

## Unmount log directory
1. Remember we mounted /var/log/httpd/ from our Web Servers to the NFS server  in our [tooling project](https://github.com/realayo/Tooling_Website_Solution) - unmount them and make sure that each Web Server has its own log directory.
```
sudo umount -l <WebServer-Private-IP-Address>:/mnt/logs
```
2. Open two ssh consoles for both Web Servers and run following command:
```
sudo tail -f /var/log/httpd/access_log
```
3. Refresh your browser page `http://<Load-Balancer-Public-IP-Address>/index.php` several times and make sure that both servers receive HTTP GET requests from your LB - new records must appear in each server’s log file.
![](https://user-images.githubusercontent.com/18899718/120821600-3e016600-c51b-11eb-97e3-b82ce4b4fb1a.png)