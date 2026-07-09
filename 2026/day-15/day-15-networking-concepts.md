# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## Objective

Today I learned the fundamental networking concepts that every DevOps Engineer should understand. I explored how DNS resolves domain names, how IP addresses work, the basics of CIDR and subnetting, and why ports are important for network communication.

---

# Task 1 – DNS (How Names Become IPs)

## What happens when you type google.com in a browser?

When we type `google.com` in a browser, the browser first asks a DNS server to convert the domain name into an IP address. Once the IP address is received, the browser connects to that server using the IP address and sends an HTTP/HTTPS request. The server then responds with the requested webpage.

---

## DNS Record Types

### A Record
Maps a domain name to an IPv4 address.

Example:

```
google.com → 142.251.142.238
```

---

### AAAA Record

Maps a domain name to an IPv6 address.

---

### CNAME Record

Points one domain name to another domain name.

Example:

```
www.example.com → example.com
```

---

### MX Record

Specifies the mail server responsible for receiving emails.

---

### NS Record

Specifies which DNS servers are authoritative for a domain.

---

## Command

```bash
dig google.com
```

### Output

```text
google.com. 20 IN A 142.251.142.238
```

### Observation

- A Record IP Address: **142.251.142.238**
- TTL: **20 seconds**

---

# Task 2 – IP Addressing

## What is an IPv4 Address?

An IPv4 address is a unique address assigned to every device connected to a network. It consists of four numbers separated by dots.

Example:

```
192.168.1.10
```

---

## Public IP vs Private IP

### Public IP

A Public IP is accessible over the internet.

Example:

```
13.48.xx.xx
```

---

### Private IP

A Private IP is used only inside private networks.

Example:

```
172.31.36.59
```

---

## Private IP Ranges

```
10.0.0.0 – 10.255.255.255

172.16.0.0 – 172.31.255.255

192.168.0.0 – 192.168.255.255
```

---

## Command

```bash
ip addr show
```

### Observation

```
172.31.36.59/20
```

This belongs to the **172.16.0.0 – 172.31.255.255** range, so it is a **Private IP Address**.

---

# Task 3 – CIDR & Subnetting

## What does /24 mean?

`/24` means the first **24 bits** represent the network portion and the remaining **8 bits** represent host addresses.

---

## Why do we subnet?

Subnetting divides a large network into smaller networks.

Benefits:

- Better network management
- Improved security
- Reduced unnecessary network traffic
- Efficient use of IP addresses

---

## CIDR Table

| CIDR | Subnet Mask | Total IPs | Usable Hosts |
|------|-------------|-----------|--------------|
| /24 | 255.255.255.0 | 256 | 254 |
| /16 | 255.255.0.0 | 65,536 | 65,534 |
| /28 | 255.255.255.240 | 16 | 14 |

---

# Task 4 – Ports (The Doors to Services)

## What is a Port?

A port is a communication endpoint on a computer.

An IP address identifies the server, while a port identifies the service running on that server.

Example:

```
13.60.xx.xx:22
```

Here,

- IP identifies the server.
- Port 22 identifies the SSH service.

---

## Common Ports

| Port | Service |
|------|----------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 53 | DNS |
| 3306 | MySQL |
| 6379 | Redis |
| 27017 | MongoDB |

---

## Command

```bash
ss -tulpn
```

### Observation

From the output:

| Port | Service |
|------|----------|
| 22 | SSH (sshd) |
| 53 | DNS (systemd-resolved) |

---

# Task 5 – Putting It Together

## Question 1

### You run:

```bash
curl http://myapp.com:8080
```

### What networking concepts are involved?

1. DNS resolves the domain name into an IP address.
2. The client connects to the server using the IP address.
3. TCP establishes the connection.
4. Port **8080** identifies the application.
5. The server returns an HTTP response.

---

## Question 2

### Your application can't reach a database at:

```
10.0.1.50:3306
```

### What would you check first?

- Verify whether the database IP is reachable.
- Check whether port **3306** is open.
- Verify the MySQL service is running.
- Check firewall or security group rules if required.

---

# Commands Used

```bash
dig google.com

ip addr show

ss -tulpn

ping google.com

tracepath google.com
curl -I https://google.com

netstat -an | head
```

---

# Screenshots

| Screenshot | Description |
|------------|-------------|
| 01-dns-lookup.png | DNS lookup using dig |
| 02-ip-address-information.png | IP address information |
| 03-ping-and-tracepath.png | Connectivity test |
| 04-listening-ports.png | Listening ports using ss |
| 05-http-check.png | HTTP response using curl |

---

# What I Learned

- DNS converts domain names into IP addresses.
- Public and Private IP addresses have different purposes.
- CIDR notation defines network and host portions of an IP address.
- Subnetting improves network organization and security.
- Ports identify specific services running on a server.
- Commands like `dig`, `ip addr`, `ss`, and `curl` are useful for troubleshooting networking issues.

---

# Conclusion

Today's session helped me understand the basic networking concepts used in real DevOps environments. Instead of only memorizing commands, I learned how DNS, IP addresses, subnetting, and ports work together whenever a client communicates with a server. This practical understanding will help me troubleshoot networking issues more confidently.
