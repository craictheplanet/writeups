

# Programming - 150 - Web Cipher

This challenge requests that you connect to ```neverlanctf-challenges-elb-1477748152.us-west-2.elb.amazonaws.com``` on port ```1180```. When you do  you are met with a challenge to decrypt a string and you are given ten seconds to do it multiple times.

```
$ nc neverlanctf-challenges-elb-1477748152.us-west-2.elb.amazonaws.com 1180
Welcome to the NeverLAN CTF.
You have 10 seconds to answer these questions.
decrypt: YXJlbHl0ZHRobA==
That's not it. Restart!
decrypt: cG55c2xqZmV4cw==TEST
That's not it. Restart!
decrypt: ZXRsZW11dGFhcw==etlemutaas
To Late. You're obviously not a bot
```

The strings can immediately be identified as base64 and there is no further encoding, the main problem will be sending the data quick enough to meet the time constraints.

The following program can connect to the service and supply the answers automatically for you, it simply receives the buffer of data and extracts the data , base64 decodes it and sends it back. It's easy enough to grab the encrypted data because it's always the last bit of data that arrives after the characters ": "

Note: This only works in python 2 because socket buffers are strings and not bytes streams.

```
!/usr/bin/python

import socket
import base64
import time

BUFFER_SIZE = 1024

print("leeto challenge crack0r by ack_")

# Create Socket and Connect to host
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("neverlanctf-challenges-elb-1477748152.us-west-2.elb.amazonaws.com", 1180))

# Create a loop for sending and receiving data
for i in range(10):

        # Accept the first buffer of data
        data = s.recv(BUFFER_SIZE)
        # Print it to the screen because, why not.
        print ("received data:" + data)
        # Split the screen with the separator ": " , base64 the answer and send it back
        s.send(base64.b64decode(data.split(": ")[-1]))
        # little snooze
        time.sleep(1)

# Rejoice with your flag and close connection
s.close()
```

Now run the code and grab the flag.

```
$ ./web.py 
leeto challenge crack0r
received data: Welcome to the NeverLAN CTF.
You have 10 seconds to answer these questions.
decrypt: dG1kbmhveHhwbQ==
received data: Awesome, continuing.
decrypt: d3VkcnVzbGVmZw==
received data: Awesome, continuing.
decrypt: aHVsaGttZWFpdw==
received data: Awesome, continuing.
decrypt: a3FwcGV4bmRqbw==
received data: Awesome, continuing.
decrypt: dmxka2ZtZXp2cQ==
received data: Awesome, continuing.
flag{Ant1_hum4n}
````
