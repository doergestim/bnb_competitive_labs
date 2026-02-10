![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Haschat

# Ubuntu VM


In this lab we will be getting started with the fundamentals of password cracking.  We will be using **Hashcat** to do this.

After you open a terminal, we need to gain root access by running the following:

Now, let's delete any old leftover pot files

```bash
rm /root/.local/share/hashcat/hashcat.potfile
```

If you get an error that the file does not exist, that is fine.  It just means the file does not exist.  Carry on.

We need to navigate to the appropriate directory. Run the following:

```bash
cd ~/Intro_To_Security/Password_Cracking
```

Lets begin by attempting to crack some **MD5 hashes**. 

Run the following command:

```bash
hashcat -a 0 -m 0 -r /usr/share/hashcat/rules/Incisive-leetspeak.rule MD5.txt password.lst
```

The result will look like this:

![](attachments/md5run.png)

Running this command will not show us the cracked hashes. As seen above, in order to see cracked hashes, we need to run our command again and add the **--show** option onto the end.

After running the command again with the **--show** option, you should see something like this:

![](attachments/md5hashes.png)

Lets crack some NT hashes.  These are the hashes that almost all modern **Windows** systems store these days.  Older systems may store **LANMAN**, but that is very rare.

Lets run the following command:

```bash
hashcat -a 0 -m 1000 -r/usr/share/hashcat/rules/Incisive-leetspeak.rule sam.txt password.lst
```

When this command is complete, it should look like this:

![](attachments/nthashrun.png)

We will not see the cracked hashes unless we append **--show** onto the end of the command. Lets do that.  Run it again to see the cracked hashes:

![](attachments/ntcracked.png)


---

# Finished?

[Back to Card's Main Page](/Cards/PE/Kerberoasting.md)
