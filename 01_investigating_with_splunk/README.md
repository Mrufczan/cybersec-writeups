Let's walkthrough TryHackMe's room called "Investigating with Splunk"



Scenario goes like this

<mark>SOC Analyst Johny has observed some anomalous behaviours in the logs of a few windows machines. It looks like the adversary has access to some of these machines and successfully created some backdoor. His manager has asked him to pull those logs from suspected hosts and ingest them into Splunk for quick investigation. Our task as SOC Analyst is to examine the logs and identify the anomalies. </mark>

**How many events were collected and Ingested in the index main?**

search index=main timeline alltime 12256

**On one of the infected hosts, the adversary was successful in creating a backdoor user. What is the new username?**

new user - logs collected from windows in windows logs event id for user creation 4720

index=main EventID="4720" one hit 

New Account:

&nbsp;    Security ID:        S-1-5-21-1969843730-2406867588-1543852148-1000

&nbsp;    Account Name:        A1berto

&nbsp;    Account Domain:        WORKSTATION6

**On the same host, a registry key was also updated regarding the new backdoor user. What is the full path of that registry key?**

index=main EventID="13" | search A1berto

HKLM\\SAM\\SAM\\Domains\\Account\\Users\\Names\\A1berto\\

**Examine the logs and identify the user that the adversary was trying to impersonate.**

Alberto

**What is the command used to add a backdoor user from a remote computer?**

Probably via procces creation ID 4688 with added command to create user which as we know is A1berto

index=main EventID="4688" | search A1berto

"C:\\windows\\System32\\Wbem\\WMIC.exe" /node:WORKSTATION6 process call create "net user /add A1berto paw0rd1"

**How many times was the login attempt from the backdoor user observed during the investigation?**

Windows Security Log Event ID 4624 for successful login and ID 4624 for Unsuccesful one

index=main EventID="4624" OR EventID="4625" | search A1berto

Got 0

**What is the name of the infected host on which suspicious Powershell commands were executed?**

we can go back to question 5 and it is clear that infected hostname is James.browne

**PowerShell logging is enabled on this device. How many events were logged for the malicious PowerShell execution?**

Hard one found powershell activities on https://www.iblue.team/incident-response-1/logging-powershell-activities

Using index=main | search EventID="4103" OR EventID="4104"

Got 79

**An encoded Powershell script from the infected host initiated a web request. What is the full URL?**

Hard one... from the last search wee can clarly ssee under ContextInfo in field Host application execution of powershell.exe witn -enc, which means that message will be encoded in Base64.  

Tried to extract only Host Application value, bo with no avail. But after a bit of digging found solution on https://community.splunk.com/t5/Splunk-Search/How-to-extract-a-text-from-a-field/m-p/261303#M78414 and with usage of https://regexr.com/ made this index=main | search EventID="4103" OR EventID="4104" | rex field=ContextInfo "Host Application = (?<Powershell>\[^\\n]+)" | dedup Powershell | table Powershell

With it we get one uniqe command pushed through powershell. 

Try cyber chef doesn't look good. lets remove null bytes and look whats in the code

got some resemblance of URL FroMBASe64StRInG('aAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADAALgA1AA==')));$t='/news.php' so lets try decode once more full address shoud be hxxp\[://]10\[.]10\[.]10\[.]5/news\[.]php
