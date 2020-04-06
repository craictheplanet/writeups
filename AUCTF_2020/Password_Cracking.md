# Password Cracking Challenges
## Crack Me
```
Here's an easy one.

Hash:
33f966f258879f252d582d45cef37e5e
```
Hashcat with rockyou wordlist cracks this easily enough.

## Crack Me 2
```
Here's another one.

Hash:
b1ee3fbc44b4ba721273699ac4511fa1631257f37da7bede3d5ba7bda5e7f96f1bab30e206caf47a5ce8c6587d0fbd6306e70b08a3a7e7233bb707bf21752c33
```
This looks like SHA-512 but it's actually SHA3-512.
Cracks with rockyou wordlist when you use the right hash mode ( 17600 )

## Big Mac
```
You might need this: thisisasecret

Hash:
5ee9fafd697e40593d66bef8427d40f8beca6921
```
I knew Big Mac was a hint at something and I saw that the hash was the right length for sha1 and had to be salted or something.
I scrolled through https://hashcat.net/wiki/doku.php?id=example_hashes and saw the mode ```HMAC-SHA1``` and it was all clear.
Cracks with rockyou and hashcat mode 160


## Manager
```
There might be a flag inside this file.
The password is all digits.
```
For this challenge we are given a password protected KeePass file.
I used keepass2john from [John The Ripper](https://www.openwall.com/john/) to extract the hash.
Then I used a mask attack to bruteforce the password as we know the password is all digits.
```hashcat -m 13400 --username manager.hash -a 3 ?d?d?d?d?d?d?d?d?d?d --increment```

## Mental
```
Mental
The password format is Color-Country-Fruit.

Hash:
17fbf5b2585f6aab45023af5f5250ac3
```
For this we need to get three wordlists: colours, countries and fruits.
I made a script to join the three wordlists together and generate every combination.
```
from sys import argv

script,color_f,country_f,fruit_f = argv

def load_words(fname):
        a = set()
        with open(fname) as f:
                for line in f.readlines():
                        a.add(line.rstrip())
        return a

colors = load_words(color_f)
countries = load_words(country_f)
fruits = load_words(fruit_f)

for color in colors:
        for country in countries:
                for fruit in fruits:
                        print(f"{color}-{country}-{fruit}")
                        print(f"{color.lower()}-{country.lower()}-{fruit.lower()}")
```
My attempt at this didn't work. Later 3mm4h3ff built got it with a similar wordlist but with the first letter of each word capitalised.


## Salty
```
You might need this: 1337

Hash:
5eaff45e09bec5222a9cfa9502a4740d
```
This was a salted md5 hash that cracks if you use hashcat mode 20 and rockyou.

## Zippy
```
Have fun!
```
This was a series of passworded zip files nested within eachother.
I used zip2john to extract the hashes and then ran them with hashcat and rockyou.
Rinse and repeat that until you get to the flag.

## Keanu
```
My friend Keanu Reeves forgot his password. Can you help him out?

Hash:
a32480e4c78df0fccfd921d42d2adf03
```
This one was frustrating but eventually we got hints.
First hint: ```There is a *cool* tool that can help you solve this challenge.```
I went to the [Kali tools page](https://tools.kali.org/tools-listing) to see what this was hinting at.
CeWL is on the Password Attacks tool list. Its a wordlist building tool that spiders webpages to build its list.
We ran CeWL using the Keanu Reeves wiki page as a starting point to build a wordlist.
The second hint for the challenge told us we needed to apply rules.
I ran the list with all the rules and couldn't figure out what was wrong but 12thRockYou managed to crack it by running rules simultaneously:
```hashcat -m 0 a32480e4c78df0fccfd921d42d2adf03 keanu_wordlist.txt  -r rules/leetspeak.rule -r rules/d3ad0ne.rule ```
