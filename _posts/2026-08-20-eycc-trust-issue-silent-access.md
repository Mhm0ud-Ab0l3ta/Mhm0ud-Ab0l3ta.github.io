---
title: "Two DFIR Challenges I Created for EYCC CTF {Trust Issue-Silent Access}"
date: 2026-08-20
categories: [challenge-authoring, dfir, ctf]
description: "Recently, I had the opportunity to create two DFIR challenges for the EYCC CTF."
image:
  path: https://cdn-images-1.medium.com/max/1024/1*J3FSVatkytH_ZCsgbxAstw.png
  alt: "EYCC Trust Issue and Silent Access DFIR Challenges"
toc: true
---

Recently, I had the opportunity to create two DFIR challenges for the EYCC CTF.

At first, I wanted to make them super easy, but then I changed my mind. I wanted participants to actually search, investigate, and learn new things along the way — hopefully enough to make them even more interested in DFIR.

For the qualification phase, I created a Windows Disk Forensics challenge that was rated as Hard by the participants. That was actually close to what I wanted: a challenge that would push them to investigate, research, and learn along the way.

What impressed me the most was that many participants still managed to answer 8 out of the 10 questions correctly.

Seeing how well they performed showed me that they were able to do exactly what I had hoped for, which was one of my main goals when designing the challenge.

## **First challenge: Trust Issue**

![](https://cdn-images-1.medium.com/max/828/1*kuNWl6DppGqLRu2Ios5LDA.png)

Here the provided artifact is the victim’s **C:** logical folder so lets start our analysis based on the questions.

## Q1 → The victim interacted with the threat actor on a specific chat platform to receive a requested document. What is the name of this communication application?

While navigating through the provided files,we do not immediately find an installed application that clearly answers the question.

So, instead, we can look at the victim’s recent browsing activity. we can see that from the **History DB** located at:

```text
C\Users\mm370\AppData\Local\Google\Chrome\User Data\Default
```

*we can open it using DB browser for SQLite.*

![](https://cdn-images-1.medium.com/max/1024/1*FUhwqi5wQjuG42pVezvJFg.png)

Looking through the visited URLs, there we can see the user was using **Discord through the browser**.

→ Answer: **discord**

## Q2 → What is the ID of the conversation between the attacker and the victim?

- We can identify that directly from the visited URL we saw before.

```text
https://discord.com/channels/@me/1518625267487346844
```

- Since this was the only Discord channel URL found in the browser history database, it directly reveals the conversation ID

→ Answer: **1518625267487346844**

## Q3 → The attacker mentioned the password used to unzip the folder containing that malicious pdf, what is the used password?

For this question, we need to recover the conversation between the attacker and Mahmoud.

Since Mahmoud was using **Discord through Google Chrome**, the browser cache is a useful artifact to examine. Chrome stores cached web resources under:

```text
C:\Users\mm370\AppData\Local\Google\Chrome\User Data\Default\Cache\Cache_Data
```

To be able to view the chrome cache files in a readable way is using Chromecacheview.exe tool so by opening it and loading the Cache folder there and searching by the channel ID to specify the conversation we can see those files

![](https://cdn-images-1.medium.com/max/1024/1*AMZHBbYnCQEQd89iOAo_Uw.png)

so by inspecting the **10.json** We can see:

![](https://cdn-images-1.medium.com/max/1024/1*NIlrZDG9jgtbfo3B_YmFbg.png)

→ Here we got our answer: **1233m7SeNshE7at3**

There we can find too that the user downloaded **Nexora_Document.7z** from the same channel which seems to be the malicious document.

```text
https://cdn.discordapp.com/attachments/1518625267487346844/1519351408330276935/Nexora_Documents.7z?ex=6a3d3dc3&is=6a3bec43&hm=7a1005bc0820bbf97dd7c600461b9cb3f874c2e7fd7c127b4cbc2e8975b1cbec&
```

## Q4 → The victim reported that upon double-clicking the supposed HR document a Command Prompt window briefly flashed on the screen before the document opened. What is the exact name of the hidden batch script that was executed via the command line arguments?

In the user Downloads we can see the downloaded document which seems to contain a .lnk file not pdf as seemed to be so let’s analyze it using LECmd.exe

![](https://cdn-images-1.medium.com/max/1024/1*7ePQbJtwcPhVpsQzmXWPYA.png)

Here, we can identify the hidden batch script, **ECCY.bat**, which is referenced in the shortcut’s command-line arguments and executed when the malicious .lnk is opened.

The shortcut first launches cmd.exe, which then runs the batch script through the specified command-line arguments.

We can also see that the shortcut uses **document.ico** as its icon. Combined with the filename resembling a PDF document, this was likely intended to make the .lnk file appear legitimate and persuade the victim to open it as if it were a normal PDF.

- Answer: ECCY.bat

## Q5 → What is the name of the legitimate binary (LOLBin) that the attacker utilized to deliver the malicious payload onto the compromised system?

We can inspect the Windows Prefetch directory, which records evidence of recently executed programs.

Using PECmd.exe, we can parse the Prefetch files:

```text
PECmd.exe -d "C:\Users\mm370\Downloads\Trust_Issuee\Trust_Issuee\Trust_Issue2\2026-06-25T024027_Trust_Issue\C\Windows\prefetch" --csv "C:\Users\mm370\OneDrive\Desktop\Solver" --csvf "prefetch.csv"
```

![](https://cdn-images-1.medium.com/max/1024/1*WjKUghoKY7r8SXMhDHPv7A.png)

curl.exe is a legitimate Windows binary commonly used to transfer data over protocols such as HTTP and HTTPS.

Because it can download remote files directly from the command line, attackers can abuse it as a **LOLBin** to retrieve malicious payloads while relying on a trusted binary already present on the system.

By inspecting the files loaded we can get

![](https://cdn-images-1.medium.com/max/999/1*U-yULJ0mgiGg-lI1-I6nXw.png)

suspected .exe installed in the system32 called **Admin.exe**

- Answer: **curl.exe**

## Q6 → What is the full path of the delivered malicious Payload?

We knew that from the previous question already which is:

```text
C:\Windows\System32\ADMIN.exe
```

## Q7 → The attacker established a persistence mechanism to automatically trigger the delivered payload. What is the name of the component responsible for this execution?

Again from the prefetch the execution of the **sc.exe** just after installing the **Admin.exe** using **curl.exe** this is typically indicates that the adversary is attempting to **manipulate Windows services to execute the downloaded payload** Since sc.exe is commonly used to create and manage Windows services.

Windows stores service configuration beneath the Services registry tree. By examining the offline SYSTEM hive and searching for references to ADMIN.exe, we can identify a suspicious service associated with the payload Under

HKLM\SYSTEM\CurrentControlSet\Services

![](https://cdn-images-1.medium.com/max/1024/1*MiEfOog9lwPUAq9wky7Spw.png)

→ Answer: **WinCloudSyncManager**

## Q8: The attacker aimed to establish persistence what is the MITRE ATT&amp;CK ID of the used technique? Format (T\*\*\*\*.\*\*\*)

Since the attacker established persistence by creating a Windows service that executes the malicious payload, the behavior maps to:

**T1543.003 — Create or Modify System Process: Windows Service**

![](https://cdn-images-1.medium.com/max/1024/1*REM3Q4eUvf7M-hExNx3spg.png)

Answer: **T1543.003**

## Q9: The attacker left a message for the victim trolling him, Get the main content of that txt file.

On the user’s Desktop we can identify a .txt file named **St3Y_H3MbLe** which suggests that it contains the left message lets ensure that by seeing the creation date of that file from the **$J so after parsing using MFTECmd.exe and searching by the file name we got:**

```text
MFTECmd.exe -f C:\Users\mm370\Downloads\Trust_Issuee\Trust_Issuee\Trust_Issue2\2026-06-25T024027_Trust_Issue\C\$Extend\$J --csv "C:\Users\mm370\OneDrive\Desktop\Solver" --csvf "usnjrnl.csv"
```

![](https://cdn-images-1.medium.com/max/1024/1*axdKUbY7oiX8m9d5yRqJEQ.png)

we can observe activity associated with the file between **2026-06-25 02:31:34** and **2026-06-25 02:36:26**, placing it within the attack timeline.

We can further correlate this with the corresponding .lnk artifact, which indicates that the victim opened the text file.

Since the file was too large to fit within the available resident space of its MFT record, its $DATA attribute became **non-resident**.

This means the file content was stored in separate disk clusters, while the MFT record only kept metadata and references to where that data was located.

Since the original content cannot be recovered directly from the MFT record, the **Windows Search index** becomes another valuable artifact to examine.

On this system, the search index is stored in Windows.edb at:

```text
C:\ProgramData\Microsoft\Search\Data\Applications\Windows\Windows.edb
```

Windows Search can index both file metadata and, for supported file types, their contents. This makes Windows.edb a valuable forensic artifact, as indexed data may still remain even after the original file has been deleted.

Since St3Y_H3MbLe.txt existed during the attack timeline, checking Windows.edb may allow us to recover the attacker’s message.

During the analysis of Windows.edb using WinSearchDBAnalyzer, the tool located the entry for St3Y_H3MbLe.txt but failed to parse its indexed content, returning Parsing Error : lV pointer. In the ESE database architecture, larger text or binary values may be stored separately from the main record as **Long Values (LVs)** and referenced through a **Long Value ID (LID)**.

→ You can read more about that here : [**Long Value Columns**](https://learn.microsoft.com/en-us/windows/win32/extensible-storage-engine/long-value-columns)

In this case, the error indicates that WinSearchDBAnalyzer was unable to resolve the referenced Long Value rather than proving that the database itself was corrupted. Switching to **SIDR (Search Index Database Reporter)** allowed us to successfully recover the indexed text content.

You can download the tool form here: [**SIDR**](https://github.com/strozfriedberg/sidr)

So by using it:

```text
./sidr -r to-stdout /mnt/c/Users/mm370/OneDrive/Desktop/Solver/Windows | grep -i "st3y"
```

![](https://cdn-images-1.medium.com/max/1024/1*QdcbwAcg0r5etCEr57iqAg.png)

```text
A wise main once  said:
        ----D0n't_Trust_Just_V3rify-------
Said by Ab0L3TA
```

→ Answer : **D0n’t_Trust_Just_V3rify**

## Q10: What is the exact creation time of this txt file on the victim machine? Format (YYY-MM-DD HH:MM:SS)

We already got that answer form the usnjrnl.csv we got from parsing the **$J**

![](https://cdn-images-1.medium.com/proxy/1*axdKUbY7oiX8m9d5yRqJEQ.png)

→ Answer: **2026–06–25 02:31:34**

From there we got our flag :

![](https://cdn-images-1.medium.com/max/1024/1*0lkpz5qvWAMlsmRGmK1goA.png)

That was the one for qualifications.

It may look like a fairly long investigation, but participants had 48 hours for the qualification round, and the challenge was intentionally designed to make them explore multiple Windows forensic artifacts along the way.

The EYCC Finals DFIR Challenge → [**Silent Access**](https://medium.com/@MAb0EL3TA/eycc-simple-access-dfir-finals-challenge-9072d49352f2)

**Here we go -_- feel free to reach out to me for any clarification.**

……………..

**I hope you enjoyed this write-up and found everything easy to follow.**

**Let me know if you have any feedback!**

………………

{% include toc-image-shift-fix.html %}
