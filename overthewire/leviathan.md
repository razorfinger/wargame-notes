Leviathan

---

Level 0

Poking around we see a `.backup` folder in the user directory, which contains a file `bookmarks.html`. Poking around in it with a search for "leviathan" and sure enough, there's `leviathan.labs.overthewire.org`, with a password for `leviathan1`.

---

Level 1

So there's an executable named "check" in the user directory. It's looking for a password. Who knows what the password is?

I tried `strings` and got a few strings, then eventually `strace`, which didn't give me much in the way of library calls. So I learned about `ltrace`.

`strings` isn't super useful here, but `ltrace` is. `ltrace ./check` shows the `strcmp` for `sex`, which is the password. It drops us to a shell with uid `leviathan2`. Then we can `cat /etc/leviathan_pass/leviathan2` and get the password.

--- 

Level 2*

So we have a file named `printfile` in the home directory. What is it? It appears that it gives me files, like "cat" but owned by `leviathan3`. So will it give me `/etc/leviathan_pass/leviathan3`? Nope. We need to know what "file" it lets us have, so back into `ltrace` to see if we can get a strcmp or something like the last level.

So if we `ltrace` we notice that we have an `access` call. This dereferences symbolic links and the sort, and gives the `real` user ID. What if we feed it two files? Nope, seems OK. Files with spaces in the names?

There we go. `cat` only gets called on the first word, but the `access()` call uses the whole path. Nice.

```
$ touch "fake file.txt"
$ ln -s /etc/leviathan_pass/leviathan3 fake
$ ~/printfile "fake file.txt"
```

and we should get the file. Yep!

---

Level 3

Back to the password game. Let's see if we can pull it out in `strings`. We have a ton of strings. `secret` is a cute red herring; but what about `snlprintf`, right before "You've got shell!" 

Well, that works and we have a shell. `cat /etc/leviathan_pass/leviathan4` to continue.

---

Level 4

Level 4 starts out with nothing but a hidden directory `.trash` in the home directory, which contains a binary named `bin`. What does `bin` do? Hand us binary digits. Decoding the binary gives us something that looks like a password, and is the same length as the other passwords, so...

---

Level 5

We have a binary `leviathan5` as soon as we log in. Running it with no arguments, nothing, gives us the error `Cannot find /tmp/file.log` so I assume we need to make it. `touch /tmp/file.log` gets us a file we control; I then `chmod a+rw /tmp/file.log` to head off any permissions weirdness. And what's in the file?

The file's gone. OK, so let's `ltrace ./leviathan5` to see what's going on. It appears to open the file, get something out of it, and then destroy it with `unlink`. So now, uh, let's put something in the file and see what happens.

So it's just reading the file and then destroying it... what if we make this a symlink to /etc/leviathan_pass/leviathan6? `ltrace` shows no `access` call or similar, and it has read permissions on that file...

```
$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
```

Well, that was easy. There's your password.

---

Level 6

Yet another binary. What's this one do?

```
$ usage: ./leviathan6 <4 digit code>
```

A quick look with `strings` doesn't show any integer string; it just shows it using `atoi` so it's stopping us from just stringing it. Back to `ltrace`, which gives me nothing useful.

Fuck it, let's just bruteforce it. It's only 10000 things.

```
#!/usr/bin/python
import os, sys
try:
	for i in xrange(10000):
		code = str(i).zfill(4)
		print "Trying %s" % code
		os.system('~/leviathan6 ' + code)
except KeyboardInterrupt:
	sys.exit(0)
```

At `7123` we get a shell. `cat /etc/leviathan_pass/leviathan7`.

---

Level 7

Log in and get your congratulations. Nice and encouraging.

`Well done! You seem to have used a *nix system before, now try something more serious.`

