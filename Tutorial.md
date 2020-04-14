# Introduction
In this python3 coding exercise you will learn how to connect and authenticate to a `http`, `ssh` and `ftp` service in python3. 

```
                            +--------------+--------------+
                            |              |              |
                            |              |    http      |
                            |              |              |
+------------------+        |              |              |
|  python3 solver  +------->+  fail2ban    |    ssh       |
+------------------+        |              |              |
                            |              |              |
                            |              |    ftp       |
                            |              |              |
                            +--------------+--------------+
```

## Goal 
Please perform a password spraying attack against the server

## Password Spraying
Password spraying is a fling-mud-against-the-wall type of brute-force attack in which a malicious actor uses a single password against targeted user accounts before trying other passwords until one works. The tactic enables the hacker to remain undetected by avoiding rapid or frequent account lockouts. Attacks are typically launched against businesses and other organizations.

## Resource
Please connect to the vulnerable server from `RESOURCES`. 
* connect to port https://{PASSWORD SPRAYING SERVER}/
* password changes every hour (60 minutes)

## Fail2Ban
Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks. Fail2Ban operates by monitoring log files for `failed login attempts` and blocks selected IP addresses that may belong to hosts that are trying to breach the system's security. 

* The intrusion detection tool `fail2ban` is activated on the host. 
* lockout period is set to 10 minutes
* Range of users = 500 accounts

## Services
* ssh
* ftp
* http

##  Your Task
Please write a python3 program that is performing the password spraying attack. The client ip of the tool must change regurarely, otherwise fail2ban will lockout the password attempts. 

