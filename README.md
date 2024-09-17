# DNS-DummyServer
# Recursive DNS Server Project

## Overview

This project provides a guide for setting up a **recursive DNS server** using **BIND9**. The DNS server is configured to handle DNS queries recursively and forward unresolved queries to external DNS servers. This setup is ideal for providing local DNS resolution and offloading queries to well-known DNS providers.

## Features

- **Recursive DNS Resolution**: Handles DNS queries and performs recursive lookups.
- **Forwarding**: Forwards queries that cannot be resolved locally to external DNS servers.
- **Access Control**: Restricts DNS queries to trusted clients within a specific network.
- **DNSSEC Validation**: Ensures the integrity and authenticity of DNS data.

## Prerequisites

- **Linux Server**: Ubuntu, Debian, CentOS, or any other Linux distribution.
- **BIND9**: The DNS server software.

## Installation

### Step 1: Install BIND9

**On Debian/Ubuntu**:
```bash
sudo apt update
sudo apt install bind9
```

**On CentOS/RHEL**:
```bash
sudo yum install bind bind-utils
```

### Step 2: Configure BIND9

Edit the BIND9 configuration file (typically located at `/etc/bind/named.conf.options` on Debian/Ubuntu or `/etc/named.conf` on CentOS/RHEL) to include the following configuration:

```bash
acl MyNetworkTrustedClients {
    192.168.31.0/24;
    localhost;
    localnets;
};

options {
    directory "/var/cache/bind";

    recursion yes;
    allow-query { MyNetworkTrustedClients; };

    forwarders {
        8.8.8.8;    # Google DNS
        8.8.4.4;    # Google DNS
        1.1.1.1;    # Cloudflare DNS
    };
    
    dnssec-validation yes;
    auth-nxdomain no;   # conform to RFC1035
    listen-on-v6 { any; };
};
```

### Step 3: Set Up Logging (Optional)

To monitor DNS queries and server activity, configure logging by editing `/etc/bind/named.conf` to include:

```bash
logging {
    channel default_file {
        file "/var/log/bind/general.log" versions 3 size 10m;
        severity info;
        print-time yes;
        print-severity yes;
        print-category yes;
    };

    channel query_logging {
        file "/var/log/bind/query.log" versions 3 size 10m;
        severity info;
        print-time yes;
    };

    category default { default_file; };
    category queries { query_logging; };
};
```

### Step 4: Start and Enable BIND9

**On Debian/Ubuntu**:
```bash
sudo systemctl restart bind9
sudo systemctl enable bind9
```

**On CentOS/RHEL**:
```bash
sudo systemctl restart named
sudo systemctl enable named
```

### Step 5: Verify Your Setup

Check if BIND9 is running correctly:
```bash
sudo systemctl status bind9   # Debian/Ubuntu
sudo systemctl status named   # CentOS/RHEL
```

Use `dig` or `nslookup` to test DNS resolution:
```bash
dig @localhost example.com
```

## Usage

After setting up the server, configure client devices or other servers to use your DNS server for name resolution. Ensure that clients are within the allowed network range specified in the ACL.

## Troubleshooting

- **Logs**: Check `/var/log/bind/query.log` and `/var/log/bind/general.log` for detailed logs if issues arise.
- **Firewall**: Ensure that port 53 is open on your firewall for both UDP and TCP traffic.

## Contributing

If you have suggestions or improvements, please open an issue or submit a pull request. Contributions are welcome!
---

Feel free to adjust the configuration paths and details according to your specific setup. This README provides a clear guide for users to understand and implement a recursive DNS server using BIND9.
