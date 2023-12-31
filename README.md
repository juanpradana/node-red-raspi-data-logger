# node-red-raspi-data-logger
This project built for temporary data storage from eddy covariance before send to server because gas monitoring device sensor sensing in high frequency (10 - 20 Hz). Data transmission used ethernet cable for ethernet port.

## Setup Device
### Setting up Raspi
[raspi setup tutorial](https://www.raspberrypi.com/documentation/computers/getting-started.html)

### Install Node-RED
[install Node-RED](https://nodered.org/docs/getting-started/raspberrypi)

### Setup node-red and Raspi
#### Setup node-red
- Open node-red ```http://localhost:1880/```
- Install Pallete ```node-red-contrib-filesystem```
- Import flow from [logger.json](https://github.com/juanpradana/node-red-raspi-data-logger/blob/main/logger.json)

  ![image](https://github.com/juanpradana/node-red-raspi-data-logger/assets/30497994/cb9f3b44-3bdb-4eb5-bcb3-af8214367967)

- information:
  - HTTP Method
    | Methods | Address | Details |
    | --- | --- | --- |
    | GET | ip:port/data | We will receive html |
    | GET | ip:port/zipData?node={Your_Node} | We will receive zip file |
    | GET | ip:port/csvFile?node={Your_Node} | We will receive csv file |
    | POST | ip:port/csvDataString?node={Your_Node} | Body data is csv format (content-type: text/plain) |
    | POST | ip:port/csvFile?node={Your_Node} | Body data is form-data (content-type: multipart/form-data; boundary=<value>) |
    | POST | ip:port/jsonData?node={Your_Node} | Body data is json format (content-type: application/json), use thingsboard ip telemetry body data format |
  - TCP Method
    - port 2001 for Eddy Cov1, send data json formated
    - port 2002 for Eddy Cov2, send data json formated

#### Enable FTP
- Open terminal
- ```sudo raspi-config```
- Choose __Interfacing Options__
- Choose __SSH__
- Would you like the SSH server to be enabled? __Yes__
- Close and open terminal
- ```sudo apt update```
- ```sudo apt full-upgrade```
- ```sudo apt install vsftpd```
- ```sudo nano /etc/vsftpd.conf```
- Uncomment or replace

```text
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
chroot_local_user=YES
user_sub_token=$USER
local_root=/home/$USER/FTP
```

then save and exit
- Open terminal
- Create FTP directory ```mkdir -p /home/<user>/FTP/files```, change <user> to your user (default "pi")
- Modify permission of the directory ```chmod a-w /home/<user>/FTP```, change <user> to your user (default "pi")
- ```sudo service vsftpd restart```

#### Test connect FTP Using FileZilla (Windows)
- Open FileZilla
- Fill Host: sftp://<your raspi ip>, Username: pi, Password: raspberry, Port: , then quick connect

![image](https://github.com/juanpradana/node-red-raspi-data-logger/assets/30497994/d0a96b0b-c49c-4274-ada7-e43f1d9c6ff6)

#### Setting up Zerotier
We will use zerotier for access raspi aplication from internet without vnc.
- Install zerotier ```curl -s https://install.zerotier.com | sudo bash```
- Route between ZeroTier and Physical Networks >> [article](https://zerotier.atlassian.net/wiki/spaces/SD/pages/224395274/Route+between+ZeroTier+and+Physical+Networks) , don't forget to set route on zerotier.com after join network.
- Finish and we can access to node-red or FTP from another place using internet.

#### Setting up Firewall using UFW
- Open terminal
- ```sudo apt update```
- ```sudo apt full-upgrade```
- ```sudo apt install ufw```
- Enable port what we need before:
  - ```sudo ufw allow 1880``` for node-red
  - ```sudo ufw allow 22``` for SSH and SFTP
  - ```sudo ufw allow 2001``` for eddy cov1
  - ```sudo ufw allow 2002``` for eddy cov2
  - ```sudo ufw enable```
  - When get a warning ```Command may disrupt existing ssh connections. Proceed with operation (y|n)?``` choose __y__
  - ```sudo ufw status``` to check UFW status
 
## Result for example data logger
![image](https://github.com/juanpradana/node-red-raspi-data-logger/assets/30497994/b30cd2ea-5da0-4719-bdbf-949a8ddd2218)

## Why use TCP connection?
### HTTP Connection Latency Test
![WhatsApp Image 2023-07-03 at 09 26 20](https://github.com/juanpradana/node-red-raspi-data-logger/assets/30497994/38b8d45b-43ba-46ed-94da-1788ff47a2d9)
| No | Latency |
| -- | --- |
| 1 | 64 ms |
| 2 | 65 ms |
| 3 | 68 ms |
| 4 | 69 ms |
| 5 | 72 ms |
| 6 | 73 ms |
| 7 | 74 ms |
| 8 | 76 ms |
| 9 | 77 ms |

At 9 data Sended, latency ranges between 64 ms until 77 ms, with average of latency is 70.89 ms.

### MQTT Connection Latecy test
![WhatsApp Image 2023-07-03 at 09 26 21](https://github.com/juanpradana/node-red-raspi-data-logger/assets/30497994/3cf924b3-f860-41fc-977a-575454ec3969)

| No | Sended | Delay Set | Latency |
| --- | --- | --- | --- |
| 1 | 11:01:52.691 | 100 ms | - |
| 2 | 11:01:52.801 | 100 ms | 10 ms |
| 3 | 11:01:52.908 | 100 ms | 7 ms |
| 4 | 11:01:53.016 | 100 ms | 8 ms |
| 5 | 11:01:53.126 | 100 ms | 10 ms |
| 6 | 11:01:53.244 | 100 ms | 18 ms |
| 7 | 11:01:53.345 | 100 ms | 1 ms |
| 8 | 11:01:53.453 | 100 ms | 8 ms |
| 9 | 11:01:53.561 | 100 ms | 8 ms |
| 10 | 11:01:53.671 | 100 ms | 10 ms |

At 10 data Sended, latency ranges between 1 ms until 18 ms, with average of latency is 8.89 ms.

### TCP Connection Latecy test
![WhatsApp Image 2023-07-03 at 09 27 16](https://github.com/juanpradana/node-red-raspi-data-logger/assets/30497994/459dcff7-6fc0-459f-8d45-75eb201bc469)

| No | Sended | Delay Set | Latency |
| --- | --- | --- | --- |
| 1 | 02:25:23.952 | 0 ms | - |
| 2 | 02:25:23.955 | 0 ms | 3 ms |
| 3 | 02:25:23.956 | 0 ms | 1 ms |
| 4 | 02:25:23.957 | 0 ms | 1 ms |
| 5 | 02:25:23.958 | 0 ms | 1 ms |
| 6 | 02:25:23.959 | 0 ms | 1 ms |
| 7 | 02:25:23.960 | 0 ms | 1 ms |
| 8 | 02:25:23.962 | 0 ms | 2 ms |
| 9 | 02:25:23.963 | 0 ms | 1 ms |
| 10 | 02:25:23.966 | 0 ms | 3 ms |

At 10 data Sended, latency ranges between 1 ms until 3 ms, with average of latency is 1.56 ms.

### Conclusion
| Method | Latency Range | Average of Latency |
| --- | --- | --- |
| HTTP | 64 ms - 77 ms | 70.89 ms |
| MQTT | 1 ms - 18 ms | 8.89 ms |
| TCP | 1 ms - 3 ms | 1.56 ms |
