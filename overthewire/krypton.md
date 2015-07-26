Krypton

---

Level 0

OK, so we just have a base64 encoded string here. Opening up a quick Python interpreter and using `base64.b64decode`, and we're on our way to level 1 with the password `KRYPTONISGREAT`.

--- 

Level 1

So where the shit is the file `krypton2`? `find / -name krypton2` finds it for us; there's a file at `/games/krypton/krypton1/krypton2` with the ciphertext.

This is a Caesar cipher. I'll try rot13 first since that's most common these days, and well shit, there's the password.

---

Level 2

I'm not messing around with the binary. There are tons of tools to crack these codes on the internet, so I'm just using a thing that converts it to all keys and then looking for the password in it. `n = 14` gives me the password `CAESARISEASY`.

Doing it the way they want you to, they want you to use the binary, create arbitrary plaintext, and figure out the key from there. But it's a waste of time in that simple of a cipher.

---

Level 3



A simple substitution cipher again, and we have known ciphertexts. We don't know the key this time either. The thing is that these damn substitution ciphers are so simple that it's best to just keep poking around the easy way vs. taking the hints.

What I assume they want here is frequency analysis. Since we have three known plaintexts, all encrypted with the same key, and they're in English, we can attempt to see the frequencies of commonly-used letters across the three plaintexts to make an educated guess at the key. They are also massive. Using [a frequency analysis tool](http://rumkin.com/tools/cipher/frequency.php) we can paste in ALL the plaintexts and get something interesting back that hopefully lines up with what we expect from the english language. S, Q shows up a lot more than E, T, so let's try there. What if S is E? or S is T?

What we see in terms of popularity is:

s,q,j,u,b,n,c,d,g,d,z,v

versus english: 

e,t,a,o,i,n,s,h,r,d,l,u

... so what if we swap those around? Meh, not quite. But let's poke at trigrams.

"JDS" comes up. That's likely "THE", the most common english trigram. Oddly enough this fits our frequency analysios quite well so we'll roll with it.

J = T
D = H
S = E

Now we just poke around until we start getting something that makes sense. I was able to decrypt enough to start getting phrases that might allow me to reverse a bit deeper, and found that some of the plaintext is from a biography on William Shakespeare. Having real plaintext, I can reverse my errors.

The key for the substitution cipher is: `boihgknqvtwyurxzajemsldfpc`

`Well done. The level 4 password is BRUTE`

---

Level 4

Yay Vigenere, with a key length of 6. This means we don't need to statistically go hunting around for the key length with a Kasiski examination or the like; we can start straight on the cryptanalysis part.

I just grabbed an off-the-shelf Vigenere cracker, [PyGenere](http://smurfoncrack.com/pygenere/pygenere.py).

```
$ mkdir /tmp/kryp4
$ cat /games/krypton/krypton4/found1 > ciphertext
```

Then I made a small Python script:

```
from pygenere import *
with open('ciphertext', 'r') as f:
	ciphertext = f.read()
	print VigCrack(ciphertext).set_language('EN').crack_codeword(6)
```

gives us `FREKEY`.

So let's just crack this now:

```
from pygenere import *
with open('/games/krypton/krypton4/krypton5', 'r') as f:
	ciphertext = f.read()
print Vigenere(ciphertext).decipher('FREKEY')
```

And we get a password of `CLEARTEXT`. I `chmod 777 /tmp/kryp4` so I can use the same directory again if I need to.

---

Level 5

Moving on. We'll continue to use PyGenere. We could use a Kasiski examination, but let's just use computer processing power instead using the PyGenere script from the last one.

```
$ cd /tmp/kryp4
$ cp /games/krypton/krypton5/found* .
$ cat found1 found2 found3 > big_ciphertext
$ python
```

Now we'll just use the python console.

```
from pygenere import *
with open('big_ciphertext', 'r') as f:
	c = f.read().replace(' ', '')
possible_codeword = VigCrack(c).set_language('EN').crack_codeword()
print possible_codeword
```

Well that makes it seem it's "YLEHTGKE" which is obviously false. So we'll try only using one plaintext, like worked previously. Using `found1` as the ciphertext and `crack_codeword(6, 30)`, we get "KEYLENGTHKEYLENGTH". So our key is probably `KEYLENGTH`.

`Vigenere(c).decipher('KEYLENGTH')` gives us the beginning of Dickens's *A Tale of Two Cities*. There's our key.

```
with open('/games/krypton/krypton5/krypton6', 'r') as f:
	c = f.read()
print Vigenere(c).decipher('KEYLENGTH')
```

and the password, decoded, is `RANDOM`.

---

Level 6

We're done with classical crypto now, and shit is getting real. We're dealing with a stream cipher with a broken random number generator. It's like Level 2 but real.

To break this, we will need to find out what the random number generator is. Somehow we've gone from newspaper cryptograms and 100-level math to graduate-level information security in a single level, but OK here we go.

I just went out and read the hints immediately, since this is now out of my normal range of shit I've done.

First I created a file full of 1024 As (0x41) and then ran it through the system. This gave back a pattern that repeated itself every 30 characters. This means we have some sense of the periodicity of the random number generator in the stream cipher.

We will need to code up something that handles an ASCII character into binary first, since it appears to work on those blocks. Feeding it a bunch of \x00 or something doesn't actually yield any output at all so it is expecting ASCII A-Z input, which it is likely translating I suspect, and then exporting back to ASCII A-Z output.

Then, hopefully, I can recreate the RNG that gives the same answer.

Having done more research on this, it might be easier to attempt a reused key attack.

Note: I ended up solving this but do not have the script anymore. It uses a LFSR as the RNG with a specific period. Reversing the binary gives you the algorithm and the key schedule.
