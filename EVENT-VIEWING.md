**Title**: Forensics - Event-Viewing  
**CTF Title**: EVENT-VIEWING  
**Category**: Forensics  
**Difficulty**: HARD  
**Description**: Given a `.evtx` file, and three different questions to use to find the pieces of the flag.

***

**Approach**:  
Having some past experience with searching and examining event logs. I knew I wanted to get the .evtx file into Event Viewer to be able to effectively extract the necessary parts for the flag.

***

**Solution**:

Step 1: Load the .evtx File into Event Viewer  
The first step was to load the `.evtx` file into Event Viewer to start analyzing the events. 

Step 2: Examine the hints/context given in the CTF

Before jumping into the log files, I wanted to examine each hint/piece of context to get an idea of the situation I would be investigating. The hints included:

```
1. They installed software using an installer they downloaded online.
2. They ran the installed software but it seemed to do nothing.
3. Now every time they boot up and log in to their computer, a black command prompt screen quickly opens and closes and their computer shuts down instantly.
```

Step 3: Find Event IDs and Investigate Logs

For hint one, I instantly think of events involving successful software installation, completed installation, etc. After a bit of research figured that Event ID: 11707 would be the most likely ID for the event I would be searching for.

Using the filter log feature in Event Viewer, I filtered for only logs with the 11707 event ID. After searching the first log showed the successful installation of a "Totally_Legit_Software" software. But, after further examination the event didn't get any parts to the flag so I decided to look at the logs around the timestamp. 

![image](https://github.com/user-attachments/assets/78d34fe2-9329-4beb-8367-3e6dece11bc0)

Once I got to the 11:55 timestamp I was able to find the log that contained the first part of the flag. 

![image](https://github.com/user-attachments/assets/066066da-ccb2-45ca-b27e-d8e5a8ffa2a1)

For hint two, I went straight to looking for any logs that involved a software being run. After a while of searching I wasn't able to find any logs that contained any of the information that I was looking for so I decided to move onto hint three.

Hint three, I had two different ideas of what I had to look for. First was how was the file maintain persistence and two how was the file shutting the computer down. After some research I narrowed down the two events to Event IDs: 1074 and 4657. 1074 is logged when a shutdown is initiated and 4657 is logged when a registry path is edited. With this information I went and searched through the logs and found the last two parts of the flag.

![image](https://github.com/user-attachments/assets/34837a1b-95c4-4482-a5d8-386875f01693)

![image](https://github.com/user-attachments/assets/e511c30d-ad85-4491-ab86-bfa29f82d8c0)

***

**Conclusion**:  
By using Event Viewer combined with some research, was able to find the Event IDs that correlated with the logs that provided the necessary information to piece together the flag.
