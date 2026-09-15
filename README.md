# Drive-By Download Network Traffic Analysis
Nadia Rapheal, _NSF SecKnitKit Exercise_

![Wireshark Screenshot](<screencaptures/D - Copy.png>)

## Process
In this report, I analyzed a PCAP of a simulated drive-by download incident. I utilized Wireshark to analyze the redirect chain from the legitimate website, passing through a third party domain, and being redirected before installing a malicious Java file. After I determined it was reasonably suspicious, I used VirusTotal to analyze it further. I then researched the associated factors, and found that this is consistent with the Blackhole Exploit Kit.


## Findings
- The user accessed www.psicologia-online.com.
- The user then was redirected to seris.biz.
- The website seris.biz responded with a 302 Found redirect to kanon-finale.com.
- The website kanon-finale.com returned obfuscated JavaScript archives that were x-java-archive applications.
- While there was no final executable payload found within the PCAP, it was enough information to escalate to security tools like VirusTotal and research into malicious software.
