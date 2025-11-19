# installation steps :

## **1. Ubuntu / Debian**

### **Install Squid**

```bash
sudo apt update
sudo apt install squid -y
```

### **Start & Enable Squid**

```bash
sudo systemctl start squid
sudo systemctl enable squid
sudo systemctl status squid
```

### **Config file path**

```
/etc/squid/squid.conf
```

# **2. CentOS / RHEL / Rocky / AlmaLinux**

### **Install Squid**

```bash
sudo yum install squid -y
```

or on newer versions:

```bash
sudo dnf install squid -y
```

### **Start & Enable Squid**

```bash
sudo systemctl start squid
sudo systemctl enable squid
sudo systemctl status squid
```

### **Config file path**

```
/etc/squid/squid.conf
```

# **3. Amazon Linux 2 / Amazon Linux 2023**

### **Install Squid**

```bash
sudo yum install squid -y
```

### **Start & Enable**

```bash
sudo systemctl start squid
sudo systemctl enable squid
```

### **Config file path**

```
/etc/squid/squid.conf
```

# **4. Fedora**

```bash
sudo dnf install squid -y
sudo systemctl start squid
sudo systemctl enable squid
```

# **5. OpenSUSE**

```bash
sudo zypper install squid
sudo systemctl start squid
sudo systemctl enable squid
```

# Squid-server configuration

```
mv /etc/squid/squid.conf  /etc/squid/squid.conf_backup
vi  /etc/squid/squid.conf
```

```
############################################
# PARENT SQUID SERVER CONFIGURATION
############################################

http_port 3128
visible_hostname parent-squid

# Allow local network / allow VM clients
acl localnet src 10.0.0.0/8
acl localhost src 127.0.0.1/32

http_access allow localhost
http_access allow localnet
http_access deny all

# Basic allowed ports
acl Safe_ports port 80      # HTTP
acl Safe_ports port 443     # HTTPS
acl Safe_ports port 21      # FTP (optional)
acl CONNECT method CONNECT

http_access allow CONNECT !Safe_ports
http_access deny !Safe_ports

# Logging
access_log /var/log/squid/access.log squid

# Performance tuning (optional)
cache_mem 256 MB
maximum_object_size_in_memory 512 KB
maximum_object_size 50 MB
```

**restart the squid proxy service

```
sudo service squid restart
```

# Squid-client configuration

** update the proxy server ip 

10.0.0.2

** update parent_domains which is required for routing

.example.com .mycorp.com .secureapp.net .ifconfig.me

** edit the file

```
mv /etc/squid/squid.conf  /etc/squid/squid.conf_backup
vi  /etc/squid/squid.conf
```

```
############################################
# CLIENT SQUID CONFIGURATION (VM SIDE)
############################################

http_port 3128
visible_hostname client-squid

# ACL: networks allowed to use this proxy
acl localnet src 10.0.0.0/8
acl localhost src 127.0.0.1/32

http_access allow localhost
http_access allow localnet
http_access deny all

############################################
# DOMAINS TO ROUTE THROUGH PARENT SQUID
############################################
# Add all domains you want to force via parent
acl via_parent_domains dstdomain .example.com .mycorp.com .secureapp.net .ifconfig.me

############################################
# DEFINE PARENT SQUID
############################################
# Format: cache_peer <IP> parent <port> <icp-port> [options]
cache_peer 10.0.0.2 parent 3128 0 no-query default

############################################
# SEND ONLY THESE DOMAINS TO PARENT
############################################
cache_peer_access 10.0.0.2 allow via_parent_domains
cache_peer_access 10.0.0.2 deny all

# Never connect directly for those domains
never_direct allow via_parent_domains
# Everything else can go direct
never_direct deny all

############################################
# PORT RULES
############################################
acl SSL_ports port 443
acl Safe_ports port 80
acl Safe_ports port 443
acl Safe_ports port 21

# HTTPS CONNECT
acl CONNECT method CONNECT

http_access allow CONNECT !Safe_ports
http_access deny !Safe_ports

############################################
# LOGGING
############################################
access_log /var/log/squid/access.log squid

############################################
# PERFORMANCE TUNING
############################################
cache_mem 256 MB
maximum_object_size_in_memory 512 KB
maximum_object_size 20 MB

```

```
sudo service squid restart
```
