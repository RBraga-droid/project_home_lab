## Steps

1. As first, I set windows10 machine in NAT mode to enable nework connection. 
2. From there I create Sapienza account for free trial Splunk. I'm starting to think that this is not free at all, in fact I have a trial of 60 days before having to pay. It should be enough for this project. 
3. Installing the software I create an account. This account will be local for the administrator of splunk I think. 
	1. username: sally
	2. pwd: **********
4. Splunk GUI fires upon browser. 
5. Now I need to ling splunk with [[Sysmon installation]] so I go to the installation folder under ProgramFiles, etc then system then default to take the inputs.conf file and paste it into local. inputs.conf is the file to be imported, it makes it pssible for splunk to read sysmon telemetry. 
6. Then restart Splunkd from services. 
7. Now I need to create endpoint for splunk to interact with sysmon. To do so i navigate to localhost:8000 that is the GUI of splunk. 
8. I log in and then go to settings, indexes and create a basic index called endpoint:

   ![Pasted image 20240430144555](https://github.com/RBraga-droid/project_home_lab/assets/62329743/d8bc53ac-b20f-4d91-ba59-ef6e89af0495)

10. Now in apps->search and reporting I filter idnex=endpoint to start collecting sysmon telemetry. 
11. Then I need to download the parser from sysmon to splunk so I search up on "more apps" the sysmon add-on for splunk. 
