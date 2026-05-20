# Controlled Multi-Threaded HTTP Request Lab

## Overview

This project is a Python-based networking lab designed to demonstrate how sockets, threading, HTTP requests, and logging work in a controlled local environment.

The script creates multiple threads, where each thread sends a fixed number of HTTP GET requests to a local test server running inside a virtual machine. The goal of this lab is to understand how concurrent requests behave, how traffic can be observed in a lab network, and how logging can be used to record program activity.

This project was completed for educational purposes in a private lab environment.

---
## Lab Purpose

The purpose of this lab was to learn:

- How Python sockets create TCP connections
- How HTTP GET requests are built and sent manually
- How threading allows multiple tasks to run at the same time
- How to log events to a file with timestamps
- How to monitor traffic between a host machine and a virtual machine
- How high request volume can affect a test server in a controlled environment

---

## Lab Environment

The test was performed using a Windows host machine and a Kali Linux virtual machine.

| Device | Role | IP Address | Notes |
|---|---|---|---|
| Windows Host | Runs the Python script | Host machine | Sends controlled HTTP requests |
| Kali Linux VM | Runs test web server | `192.168.74.131` | Receives requests |
| Python HTTP Server | Test service | Port `8080` | Local lab web server |

The Kali VM was running a basic Python web server using:

```bash
python -m http.server 8080
```

```python
from asyncio import threads
import socket
import threading
import struct  
from datetime import datetime  


# define the target IP address and port 
ip_target = "192.168.74.131"
target_port = 8080

thread_count = 5
request_count = 10

# log file setup
FILE_LOG = 'ATTACK_LOG.txt'
LOG_LOCK = threading.Lock()
def write_log(message):
    """
    Writes a log message to the log file with a timestamp.
    Args:
        message (str): The log message to write.
    """
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with LOG_LOCK:
        with open(FILE_LOG, 'a') as f:
            f.write(f"{timestamp} - {message}\n")

# function to perform the attack
def attack(thread_id):
    for _ in range(request_count):
        # create a socket object
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # AF_INET is for IPv4 and SOCK_STREAM is for TCP
        s.settimeout(5)  # set a timeout for the socket operations to prevent hanging indefinitely
        try:
            # connect to the target
            s.connect((ip_target, target_port))
            # send a large number of requests to the target , this is the line that actually sends the attack to the target IP address and port. It constructs a simple HTTP GET request and sends it to the target using the socket connection. The request includes the target IP address in the Host header, which is a standard part of an HTTP request.
            s.sendto(b"GET / HTTP/1.1\r\nHost: " + ip_target.encode() + b"\r\n\r\n", (ip_target, target_port)) # this sends a simple HTTP GET request to the target IP address and port
            write_log(f"Thread {thread_id}: Attack successful!")
        except Exception as e:
            write_log(f"Thread {thread_id}: Attack failed: {e}")
        finally:
            s.close()   

threads = []
# create multiple threads to perform the attack
# this will create 100 threads that will continuously perform the attack function, which will send a large number of requests to the target IP address and port, potentially overwhelming the target and causing a denial of service (DoS) attack.
for i in range(thread_count):
    thread = threading.Thread(target=attack, args=(i,))
    threads.append(thread)
    thread.start()
for thread in threads:
    thread.join()
write_log("Attack completed.")
print("Attack completed. Check ATTACK_LOG.txt for details.")


```
