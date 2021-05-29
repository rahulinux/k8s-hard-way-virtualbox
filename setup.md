## Step 1# Create virtual machines

1. lb-1
   - 192.168.2.100 (public) (this will be different based on your local network)
   - 10.0.0.100    (private)

2. controller-1
   - 10.0.0.111

3. worker-1
   - 10.0.0.222

### Download Vagrantfile and spin up virtual machines 

```
git clone https://github.com/rahulinux/k8s-hard-way-virtualbox
cd k8s-hard-way-virtualbox
# Edit Vagrantfile
# - Change 192.168.2. to your network range of your lan network
# - Change 192.168.2.254 to your router ip, e.g. 192.168.1.1
# - Change "en0: Wi-Fi (Wireless)" to your network adapter 
#   You can find it using: VBoxManage list bridgedifs, Windows: .\VBoxManage.exe list bridgedifs
vagrant up
```


## Step 2# Password less authentication from controller to all nodes


Loging to controller-1

```
ssh-keygen
```

Copy content of : `/home/vagrant/.ssh/id_rsa.pub` 


Add following entry in /etc/hosts

```
10.0.0.100 lb-1
10.0.0.111 controller-1
10.0.0.222 worker-1
```

Login to all nodes include controller and run:

Append copied content to `~/.ssh/authorized_keys` 


## Step 3# Install require tools 

https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md



## Step 4# Install 


```shell
vagrant ssh lb-1
sudo -i
apt update
apt-get install haproxy -y
cat <<"EOF" > /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Configure HAProxy for Kubernetes API Server
#---------------------------------------------------------------------
listen stats
  bind    *:9000
  mode    http
  stats   enable
  stats   hide-version
  stats   uri       /stats
  stats   refresh   30s
  stats   realm     Haproxy\ Statistics
  stats   auth      Admin:Password

############## Configure HAProxy Secure Frontend #############
frontend k8s-api-https-proxy
    bind :6443
    mode tcp
    tcp-request inspect-delay 5s
    tcp-request content accept if { req.ssl_hello_type 1 }
    default_backend k8s-api-https

############## Configure HAProxy SecureBackend #############
backend k8s-api-https
    balance roundrobin
    mode tcp
    option tcplog
    option tcp-check
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server controller-1 10.0.0.111:6443 check
EOF
service haproxy restart
```





