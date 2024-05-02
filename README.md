# Home Lab

## Objective
This project is a breakdown of steps and instructions to build and test a home lab based on VirtualBox sandboxing to test and analyze malwares, observe traffic and perform future projects. The goal is to build two separate ambient, one based on Kali Linux and the other on Windows10, in a way that they can communicate with each other in a separate private network. After doing this I simulated a scenario where the attacker based on Kali performs an attack on the target Windows10, scanning ports, crafting a malware and serving it. On the other hand, to simulate the work of a SOC, I used Splunk and Sysmon to monitor and flag the malicious actions of the target. 

### Skills Learned

- Advanced understanding of VirtualBox, expecially the configuration of private network.
- Proficency in the basic toolbox of ethical hacking tools in Kali linux: nmap, msfvenom and msfconsole. 
- Basic knowledge of the installation and interface of event-driven analysis based on Splunk and sysmon. 
- Enhanced knowledge of network protocols and security vulnerabilities.

### Tools Used
[Bullet Points - Remove this afterwards]

- VirtualBox sandboxing system. 
- Windows10 and Kali linux operative systems installation process and advanced use. 
- Ethical hacking basic toolbox: msfvenom, nmap, msfconsole, reverse-TCP meterpreter.
- System monitor tools: Splunk and sysmon to monitor events. 

## Steps
1. I first download and install both windows 10 and kali, making them two virtualmachines ready to be used. 
2. Then I create a snapshot on both of them in a safe state that i call baseline. This way I'm sure that if something wrong would occurr, it won't cause too much trouble. 
	1. Both machines have about 4GB of RAM and 50-ish GB of static memory to use and download tools
	2. The idea is to later transform these two machines in toolbox for malware analysis, one static and one dynamic
	3. Just to be sure I'm writing credentials here:
		1. Kali: kali/********
		2. Windows: sally/*********
3. There are two possible network configurations that I intend to use in these two machines:
	1. **NAT**: open to the internet, useful to download and install tools, to access internet and so on.
	2. **Private Network**: the actual configuration for testing purposes. Basically the two systems are located in an isolated private network where they can only communicate between them.![Pasted image 20240430112625](https://github.com/RBraga-droid/project_home_lab/assets/62329743/dc283024-812d-48b1-831b-be6c8eac06af)
4. I will configure the static IP and configuration for private network **internal-1**.
	1. On Windows10 machine the configuration is as follows on the image:

    ![Pasted image 20240430113054](https://github.com/RBraga-droid/project_home_lab/assets/62329743/ed4ea885-964c-4b91-9c41-06a35bdd4c64)

    creating a snapshot called **internal-1-privateconfig** to go back to this configuration when needed.
  2. For the kali machine I do the same, giving the following configuration:

     ![Pasted image 20240430113844](https://github.com/RBraga-droid/project_home_lab/assets/62329743/f5d8a812-6244-4ada-b5f7-4d45ccfa0e98)
     
  3. Now for effectively test the configuration I'm hitting a ping from windows10 to kali machine: _note: running both machines slows a lot my computer. windows10 can be very heavy._

     ![Pasted image 20240430114807](https://github.com/RBraga-droid/project_home_lab/assets/62329743/ecac982d-97f1-484a-bab9-aee4b342e521)

     Note: Windows10 loses configuration when rebooted. It is a better practice to reboot it always to latest configuration.

5. Now the goal of the lab is to fire a sample malware attack from kali (atacker) to windows10 (target) trying to generate telemetry and start detecting evil actions. Note that this phase of the project serves only to show the connectivity of both machines that at this point are ready to be used as labs. The idea is to used nmap to map the target and then craft a sample malware using kali to be downloaded on target machine. Then use [[Splunk installation]] to analyze the traffic and start identifying indicator of compromise. To do so I also need [[Sysmon installation]].
6. After doing this, on the kali machine I run nmap with the following options:
	1. -A to scan os detection version ad everything similar
	2. -Pn to skip pings
7. The goal is to target the windows machine, but this can't happen because of the automted windows firewlal blocking ICMP. I shut it down. It is in fact possible to ping windows machine from kali machine now.
8. It takes a while to perform complete nmap on the target.

   ![Pasted image 20240430130444](https://github.com/RBraga-droid/project_home_lab/assets/62329743/fd100606-546f-490c-82c8-7d0806640d9c)

   Here we see the main services, TCP services attract my attention.
9. Now I use msfvenom to crafdt an example meterpreter reverse shell. The goal is to hack the windows target into making shell contact with the attacker machine, and then analyze the telemetry of the attack.
10. By hitting msfvenom -l payloads it is possible to list every payload that it gives. I'll use windows/x64/meterpreter_reverse_tcp since I've noticed there are TCP service ports on the target. I span the payload hitting the following command:

    ![Pasted image 20240430131148](https://github.com/RBraga-droid/project_home_lab/assets/62329743/e6b85dfa-c757-4263-b461-b62017d4f4f4)

    Specifying the ip of the attacker that the target has to connect, the port and the name and format of the output file.

11. Now using msfconsole it is possible to study and customize the payload itself, here is the view:

  ![Pasted image 20240430131411](https://github.com/RBraga-droid/project_home_lab/assets/62329743/dfd95f84-03a7-4e59-a2dc-5ea0a828ccff)

The option, for example, is set to generi/shell_reverse_tcp, i have to change it to the speciofic payload option I downloaded before using the command ``set payload windows...`` with the path of the specific payload I want to use. 

12.  I also need to configure lhost and port, typing command ``set lhost 192.168.20.11`` that is the kali ip.
13.  I run ``exploit`` to run the listener now that everything is set up.

  ![Pasted image 20240430131951](https://github.com/RBraga-droid/project_home_lab/assets/62329743/29a350ac-52a3-417e-a869-79d34b8a8925)

14. Now that the attacker is listening I need to simulate the serving of the malware to the target by simulating a python serving service using on another tab the command ``python3 -m http.server 9999`` with an unused port, basically serving the same directory I'm in, where the exploit is located.
15. I now shift to Windows machine and turn off the real time protection so that the exploit is not flagged as malicious. Being it produced with an automated tool it is pretty easy to detect by any antivirus, but in this case it is only for testing purposes.
16. By using the browser I can clearly see the serving file as intended:
    
  ![Pasted image 20240430132443](https://github.com/RBraga-droid/project_home_lab/assets/62329743/00a589e0-fda5-4f0e-a077-579b8418095c)

Even the browser will flag this as a potential threat, meaning that at least with automated generated malware, Windows is defended as it should. Quick note: the file extension is not shown by default on windows so using the trick in the name we can hide the malware so it looks like a normal .pdf file:

  ![Pasted image 20240430132712](https://github.com/RBraga-droid/project_home_lab/assets/62329743/986b258e-185b-44d0-8a4b-7ee9c4613609)

17. I fire the file and encounter another screen that flags the file as potentially malicious, I choose to execute anyway. After doing so, running it with compatibility, we can use the command ``etstat -anob`` to see that there is an established connection with the attacker machine:

    ![Pasted image 20240430140524](https://github.com/RBraga-droid/project_home_lab/assets/62329743/12728e38-4007-49ff-b4ad-d2a0bcb4586a)

    It is possible to see that in the specified PID there is in fact the expected file:

    ![Pasted image 20240430140719](https://github.com/RBraga-droid/project_home_lab/assets/62329743/4650f798-ebc3-4e8a-bad0-6ee1e6b71a18)

18. At this point using meterpreter I type the command ``shell`` to run a shell on the target. Mimicking a real attacker I then type
    1. ``net user`` to know about user
	  2. ``net localgroup`` to know about localgroup
	  3. ``ipconfig`` to know about net IP and similar
19. Now I head back to the target machine to see what kind of telemetry it generated. I filter with index=endpoint 192.168.20.11 the address of kali attacker machine we can see a lot opf informations:
    1. The destination port

       ![Pasted image 20240430150739](https://github.com/RBraga-droid/project_home_lab/assets/62329743/0e6d6863-344a-483a-a249-fb28caca0d4d)

    2.  Then in the same endpoint index I filter the name of the malware totally_secure_file.pdf.exe seein a lsit of events

       ![Pasted image 20240430151143](https://github.com/RBraga-droid/project_home_lab/assets/62329743/2572e613-213f-4694-adbe-74a4ebcbef45)

       I want to focus on event code 1: process creation seeing that it spawned cmd.exe with a specific PID. It is possible to look up these PID or the GUID. 
    3. I look the GUID up on the search bar and I can see the command that were called by the reverse shell:
   
       ![Pasted image 20240430151705](https://github.com/RBraga-droid/project_home_lab/assets/62329743/b05fb7c9-2010-4c1a-9bc8-581760b69489)


## Conclusions

I now have configured and tested two virtual machines that can communicate between them, mimicking a real world scenario where an attacker wants to compromise the target, an ethical hacker wants to scan a potential client or a vulnerable system can receive and scan for incoming attacks or hidden malwares. Both the machines where snapshotted to a read state so that it would take a few seconds to implement specific tools for malware analysis, traffic monitoring and similar. This would probably be my home lab for future projects. 

    






