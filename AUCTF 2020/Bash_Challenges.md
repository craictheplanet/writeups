# Bash Challenges
## Bash 1
For this one you just had to ssh in ```ssh challenges.auctf.com -p 30040 -l level1``` with the given password ```aubie```.
When you login there's a README in the home directory and you just have to run ```cat README``` to get the flag.

## Bash 2
To login to bash 2 we use the flag from the previous challenge as our password.
This time there's a file called flag.txt that is owned by the level3 user and we dont have permissions to read it.
There's also a bash script called random_dirs.sh.
```
#!/bin/bash

x=$RANDOM

base64 flag.txt > /tmp/$x
function finish {
	rm  /tmp/$x
}
trap finish EXIT

sleep 15
```
When I run ```sudo -l``` it tells me that I can run random_dirs.sh as the level3 user without a password.
To solve this I open up a second ssh session, run the script on the first one (```sudo -u level3 ./random_dirs.sh```) and cat the files in the /tmp directory to get the flag.

## Bash 3
Same scenario as previous but this time the script is:
```
#!/bin/bash

x=$RANDOM
echo "Input the random number."
read input

if [[ "$input" -eq "$x" ]]
then
	echo "AWESOME sauce"
	cat flag.txt
else
	echo "$input"
	echo "$x try again"
fi
```
I was scratching my head about this one for a while but then I realised if I put in "x" as my guess then ```if [[ "$input" -eq "$x" ]]``` becomes ```if [[ "$x" -eq "$x" ]]``` which would get me the flag.
```echo 'x' | sudo -l level4 ./passcodes.sh```
As it turns 

## Bash 4
print_flag.sh:
```
#!/bin/bash

if [ ! -z "$@" ]
then
	cat $@ # 2>/dev/null
	# if [ ! $? -eq 0 ]
	# then
	# 	echo "Printing error. Check file permissions"
	# fi
else
	echo "Please enter a file."
	echo "./print_file FILENAME"
fi
```
To get the flag run: ```sudo -l level4 ./print_flag.sh flag.txt```

## Bash 5
portforce.sh:
```
#!/bin/bash

x=$(shuf -i 1024-65500 -n 1)
echo "Guess the listening port"
input=$(nc -lp $x)
echo "That was easy right? :)"
cat flag.txt
```

For this one I found an unintended solution. The intended solution was to bruteforce ports with netcat but what I did was I opened a second ssh session.
I ran ```ps aux | grep nc``` and I could see what port it was listening on. Then I just had to run ```nc -z localhost 1234``` to stop the portforce.sh script listening and print the flag.