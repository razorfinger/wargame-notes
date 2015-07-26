0. Level 0 is ssh.

```
apt-get install proxychains tor
/etc/init.d/tor start
proxychains ssh bandit0@bandit.labs.overthewire.org
bandit0
```

The flag is in the README.

`$ cat readme`

---

Level 1

I got a laugh out of how much this annoyed me. Just give a full path.

`$ cat ./-`

---

Level 2

Easy.

`$ cat "spaces in this filename"`

---

Level 3

This is a hidden file in a directory.

```
$ ls
$ cd inhere
$ ls -la
$ cat .hidden
```

---

Level 4

```
$ ls
$ cd inhere
$ file ./-*
```

Note what you get back from file. `-file02`, `-file07`, and `-file09` all come back as an ASCII test, with `-file07` seeming the most likely since it doesn't have a warning attached.

```
$ cat ./-file07
```

Done.

---

Level 5

This could be anywhere. The goal says it's 1033 bytes in size and human readable and not executable.

```
$ cd inhere
$ ls -Ra .
```

It's a lot of stuff, and they are using both the dash hack from one level and spaces hack from another. So let's see if we can filter files that are 1033 bytes in size:

```
$ du -ab | grep 1033
```

There's only one file, `./maybehere07/.file2` that fits. Is it human-readable? Let's just `cat` it and see.

```
$ cat ./maybehere07/.file2
```

Yes.

---

Level 6

It's not in the home directory anymore. Searching for the user first.

```
$ ls -laR / | grep "bandit7"
```

Pretty bad, but I do see a `bandit7.password` in a line this way. My command filtered the location, but:

```
$ find / -name "bandit7.password"
```

And there it is, at `/var/lib/dpkg/info/bandit7.password`. You can pipe out the `Permission denied` shit but it doesn't matter in the interest of the next flag. Do what you need to, don't be clever for the sake of it if it's not wasting too much time.

---

Level 7

Big file.

```
$ du -h data.txt
4.0M	data.txt
```

So searching it manually won't work. Let's use `strings`.

```
strings data.txt | grep "millionth"
```

Well, there it is.

---

Level 8

So we need the line that only occurs once.

```
$ cat data.txt | sort | uniq -u
```

There's my flag.

---

Level 9

Among a few lines starting with =. First I tried to `cat data.txt`, but it appears it's a binary file. So `strings data.txt | grep ^=` and I have a pretty obvious line.

---

Level 10

`cat data.txt` gives me a line that is pretty obviously Base64 encoded due to the `==` on the end of the line.

`$ base64 -d data.txt` gives the "The password is" line.

---

Level 11

`$ cat data.txt`, then off to [rot13.com](http://rot13.com) with the data.

---

Level 12

We first need to stage an environment to poke around in.

```
$ mkdir /tmp/b12test
```

From there, `xxd` will revert the hexdump with the `-r` flag.

```
$ cp ~/data.txt .
$ xxd -r data.txt > compressed_file
```

Now, using `file compressed_file`, we can see it was `data2.bin` and is gzip compressed. So:

```
$ mv compressed_file data2.bin.gz
$ gunzip data2.bin.gz
$ file data2.bin
```

And now we have "bzip2 compressed data". So bunzip2 time. `bunzip2 data2.bin` gives another file at `data2.bin.out`. Running `file data2.bin.out` gives us MORE gzip compressed data, called `data4.bin`.

```
$ mv data2.bin.out data4.bin.gz
$ gunzip data4.bin.gz
$ file data4.bin
```

Finally, a `POSIX tar archive`. `tar xvf data4.bin` yields `data5.bin`, another tar archive, which yields `data6.bin`, which yields a bzip archive, which yields another tar archive in `data6.bin.out`, which yields `data8.bin`, which is a GZIPped `data9.bin`, which finally yields the text.

Yes, this is a lot of iteration. You could re-pipe the file over and over again into `tar xvf`, but the long way works, too.

----

Level 13

Level 13 is using SSH keys. `sshkey.private` exists in the home directory. Try to use it to log in as `bandit14`:

```
$ ssh -i sshkey.private bandit14@localhost
```

You'll log in as bandit14. Now, `cat /etc/bandit_pass/bandit14` for the flag.

---

Level 14

We need to send the password on port 30000 to localhost.

```
$ telnet localhost 30000
<PASSWORD><ENTER>
```

And you'll get the next password back.

---

Level 15

We need to send the password on port 30001 using SSL. This requires the use of openssl's `s_client`.

```
$ openssl s_client -connect 127.0.0.1:30001
```

Gave me a cute "HEARTBEATING" message, so I needed to add `-quiet` to the options.

```
$ openssl s_client -connect 127.0.0.1:30001
```

And all is well.

---

Level 16

We will need to use `nmap` to scan this range and check for SSL support across all 1000 ports.


`$ nmap -sV -sC -p31000-32000 localhost` gives us what's listening. We have ports `31000`, `31046`, `31518`, `31691`, `31790`, and `31960` listening, and something about them. A few are listed as "echo" servers, and one as ssh. I went after the ones nmap called "Microsoft Distributed Transaction Coordinator" since that was an odd request and made the most sense.

`openssl s_client -connect localhost:31518 -quiet` and we're connected via SSL, but it just echoes things back.
`openssl s_client -connect localhost:31790 -quiet` and we get an RSA private key as the next way to log in.

---

Level 17

Logging in with the private key, we have passwords.old and passwords.new. The password is the differing line, so I just used `diff passwords.old passwords.new` to find the password.

---

Level 18

Well, we can't log in with our password because our shell's fucked up. But we can probably scp out the file `readme`.

`$ proxychains scp bandit18@bandit.labs.overthewire.org:readme ~/bandit18_readme`

And yep, there's the password.

---

Level 19

We need to execute something as another user. `./bandit20-do` allows us to setuid of another user. If you run it without arguments it gives you a helpful hint:

`./bandit20-do id` shows the euid as `bandit20`. So `./bandit20-do cat /etc/bandit_pass/bandit20` and you have your flag.

---

Level 20

Log in twice: once to run the client, once to run the "server" we use to listen.

On the **server**, run netcat:

```
$ nc -l 32740
```

On the **client**, run the `suconnect` binary:

```
./suconnect 32740
```

Back on the server, paste the password into netcat and hit enter. The client will send the password to `nc`.

---

Level 21

So in `/etc/cron.d` there are cronjobs for ALL the next levels. `cat /cronjob/bandit_22` and we see that there's a binary `/usr/bin/cronjob_bandit22.sh`. Then, `cat /usr/bin/cronjob_bandit22.sh` will show what the hell this thing is doing. It dumps the password into a long-named file in `/tmp/`. So we `cat` that file, and boom. Flag.

---

Level 22

This is very similar to the other level. What's in `/etc/cron.d/cronjob_bandit23`? Why, a cronjob that runs `/usr/bin/cronjob_bandit23.sh`. So what's in that file?

It's making a filename from `I am user `whoami` | md5sum | cut -d ' ' -f 1`. `whoami` returns the user executing this script, which in this case we assume is `bandit23`. So the target name is `echo "I am user bandit23" | md5sum | cut -d ' ' -f 1`, a hash that begins with `8ca`. `cat /tmp/$TARGET_MD5` gives us the password.

---

Level 23

So what do we have here? Yay, time to actually write some code.

So basically we need to `cat /etc/cron.d/cronjob_bandit24.sh` to see what it's executing, no surprise it's executing `/usr/bin/cronjob_bandit24.sh`. This script executes and deletes a script in `/var/spool/$myname`, where `$myname` is returned from `whoami`. So make a directory to work in so you don't lose your script.

```
$ mkdir /tmp/croncrap
$ cd /tmp/croncrap
```

Note the weak permissions on `/var/spool/bandit24` -- this is why we can put a script into it as bandit23. The cronjob doesn't check to make sure that the script is even owned by `bandit24` as long as `bandit24` can execute it. So let's create a script that sends the password from `/etc/bandit_pass/bandit24` to a file for us:

```
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/croncrap/passwd
```

Let's make our directory writable by everyone `chmod a+w /tmp/croncrap` and script `/tmp/croncrap/script.sh` executable by everyone: `chmod a+x script.sh` and then copy it into `/var/spool/bandit24` with `cp /tmp/croncrap/script.sh /var/spool/bandit24`. Then wait a minute and see what we get.

Note: If someone else is playing the level when you are, you can probably steal their script out of `/var/spool/bandit24` as well, or `cat` it and see where they are spitting out the password.

---

Level 24 -> 25

At this moment, Level 25 does not exist, so you're done. Wargame complete.

