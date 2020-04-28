# Introduction
In this python3 coding exercise you will learn how to connect and authenticate to an `http` service in python3 using proxies.

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
* crawl and use free proxies for HTTP requests
* perform a password spraying attack on an HTTP service

## Tasks

* Task 1: Analyze the behavior of Fail2Ban
* Task 2: Analyze the Challenge 
* Task 3: Proxy Crawling
* Task 4: Password Spraying using Python

## Preparation

```bash
mkdir -p /opt/git
cd /opt/git
git clone https://github.com/ibuetler/p3s-dynamic-client-ip-password-spraying.git
cd /opt/git/p3s-dynamic-client-ip-password-spraying/HTTP-Proxy
pipenv --python 3 sync
pipenv --python 3 shell
```

# Analyzing Fail2Ban

![Fail2Ban-Service](../media/challenge/png/fail2ban.png)

## Step 1

### Resource

* Get the canonical name from `RESOURCES`
* The HTTP service is running on port `80`

## Step 2

### Analysis of Fail2Ban

Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks. Fail2Ban operates by monitoring log files for `failed login attempts` and blocks selected IP addresses that may belong to hosts that are trying to breach the system's security. 

* The intrusion detection tool `fail2ban` is activated on the host. 
* lockout period is set to 10 minutes

Send some requests to the service trying to authenticate with some random username and password. Make sure to count each of your requests so you know how many failed login attempts are possible before your IP gets banned.

You can use curl with basic authentication as follows:

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

Your challenge isn't to crack a password to a given user as modern services are mostly protected by blocking a users login for some time if the password was entered wrong a couple of times. Therefore you'll make use of an attack called **password spraying**.

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

![Password Spraying Server](../media/challenge/png/password-spraying-server.png)

**Hint: ** The server runs two different websites your *attack target* will be running on port `80` not on port `443` and isn't using SSL! 

### Clarification

Your challenge is to find the correct username to the given password in an automated manner. The range of users is limited to 500 users to speed things up.

# Proxy Scrape

To prevent your IP from getting blocked by intrusion prevention systems you can use `Proxies` to change your public IP now and then. There are many free proxies available out there but if you are planning to use them in a "professional" manner you should consider buying dedicated proxies or rotating proxy services.

**Warning:** Free proxies aren't reliable, it's very likely that only a few dozens of thousands of proxies even work.

## Step 1

### Make yourself familiar with *proxyscrape*

Proxyscrape is a small python library written to scrape free proxies from various sites. You can either use proxyscrape or write your own crawler which isn't covered in this Tutorial.

[Proxy Scrape Library](https://github.com/jaredlgillespie/proxyscrape)

## Step 2

### Using Proxyscrape

To start using proxyscrape simply import *create_collector* from proxyscrape.

```python
from proxyscrape import create_collector
```

## Step 3

### Define a collector

Proxyscrape uses collectors as an interface to retrieve proxies. Think about them as a filter for different proxy types. 

As *requests* can use **http** and **socks5** proxies a collector to retrieve http and socks5 proxies.

``` python
collector = create_collector('http-collector', ['http', 'socks5'])
```

## Step 4

### Retrieve proxies

Use the collector you've just created to retrieve the proxies

``` python
proxies = collector.get_proxies()
```

## Step 5

### Write a function

Now write a function to retrieve the proxies and return them as a list. 

**Hint:** The *requests* library expects the proxy in the following format `{'http': 'protocol://IPAddress:Port'}` whereas *protocol* has to be replaced with the proxy type either http or socks5h.

```python
def get_proxies():
    # your code goes here
```

# Password Spraying HTTP Basic Authentication

## Step 1

### Write a login function

Write a function *try_login()* that takes the four parameters *url, proxy, user* and *password* as an argument.

 ```python
import requests

def try_login(url, proxy, user, password):
    # your code goes here
    
 ```

## Step 2

### Write a runner function

As you'll be going to parallelize the requests later you should extract the logic of issuing a new request into a separate function.

Write a *runner()* function that takes a dictionary as an argument. The dictionary should contain the current username (number) and the proxies as a list.

**Explanation:** You cannot pass multiple arguments to the function you are calling from within *ThreadPoolExecutor*.

```python
def runner(params):
    # unpacking parameters
    user = params['user']
    proxies = params['proxies']
    
    # Your code goes here
```

### Picking a random proxy

Your runner function is responsible to pick a random proxy from your proxy list. You can create a random integer within a given range using:

```python
import random

random_index = random.randint(0, len(proxy_list) - 1)
```

Access the entry in the list with:

```python
proxy = proxy_list[random_index]
```

### Handling errors

As your request might fail due to a blocked IP-Address you have to handle that case within your *runner()* function. Pack your logic into a `while True:` block and retry your request until it's successful.

**Hint**: Typically you'll receive an HTTP error 429 when your IP-Address gets blocked. You should remove the proxy that has been blocked from your list to avoid unnecessary requests.

```python
def runner(params):
    ...
    while True:
        # your code goes here
    
```

### Calling the runner function

The *runner()* function will be called as follows:

```python
params = {
    'user': 123 # integer
    'proxies': proxies # list
}

runner(params)
```

## Step 3

### Parallelize your requests

You can speed up your *password spraying program* by parallelizing your requests using [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html).

Fiddle around with the *max_workers* parameter as this depends from system to system.

```python
with ThreadPoolExecutor(max_workers=50) as executor:
    print("--------- Start spraying -----------")
    tasks = {executor.submit(runner, user) for user in users}
    
    for task in as_completed(tasks):
        print(task.result())
```

## Step 4

### Password Spraying

Now as you have all your needed code wrap the parts together and try to capture the flag. Make sure to update the **password** according to https://pwspray.vm.vuln.land/.

When you add some `print` statements to your code it could look somehow like this.

LOG:

```bash
(('user_140000', 'password'), 401)
(('user_140001', 'password'), 401)
(('user_140002', 'password'), 401)
(('user_140003', 'password'), 401)
(('user_140004', 'password'), 401)
(('user_140005', 'password'), 401)
(('user_140006', 'password'), 401)
(('user_140007', 'password'), 401)
(('user_140008', 'password'), 401)
(('user_140009', 'password'), 401)
...
(('user_140500', 'password'), 401)

-- Succesful logins:
(('user_140237', 'password'), 200)

Total time needed: 0:04:54.442807
```

When you've successfully grabbed valid credentials head over to http://pwspray.vm.vuln.land/ and login manually. You should see the following output.

![Succesfull password spray](../media/challenge/png/successful-spray.png)
