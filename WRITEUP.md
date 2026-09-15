## Network Traffic Analysis of a Drive-By Download
Analyzing a potential drive-by download incident consistent with the Blackhole Exploit Kit.

### Summary
This report begins with analyzing a PCAP capture of a drive-by download incident procured from Malware-Traffic-Analysis inspired by the methodology used in the NSF-funded SecKnitKit Cybersecurity Education initiative, provided to me by Dr. K. I utilized Wireshark to analyze the redirect chain from the legitimate website (psicologia-online.com), passing through a third party domain (seris.biz), and being redirected (kanon-finale.com) before installing a malicious Java file (Zova44.class). After I determined it was reasonably suspicious, I used VirusTotal to analyze it further. I then researched the associated factors, and foudn that this is consistent with the Blackhole Exploit Kit. 

### Environment
I used Oracle VirtualBox for my virtualization, Kali Linux for the operating system of my VM, Wireshark for my packet analysis, and VirusTotal for malware verification.

### Background
A drive-by download occurs when a file, such as an executable, is downloaded onto a user's machine without their authorization. Many online downloads ask for the user to provide consent via popup, and once it is affirmed, the download begins. In a drive-by scenario, however, it is downloaded and installed in the background. The file may be malware, spyware, a computer virus, or even crimeware. Many reputable security organizations have put together databases such as VirusTotal that can be used to easily detect whether files are malicious or not.

### Process
The timeline would be that a user would look up the website using DNS, would visit the website, would be redirected to a place where they would be connected to downloading it, then we would see the payload, and then view after the infection. 

The victim in this instance is Address A, 192.168.1.101. They communicated with the other external addresses, as can be seen under the collected IPv4 conversations.

In order to see what addresses they accessed, I used http.request in the filters. I see the list of GET requests, which tell me when they request information. 

At 0.179 seconds, I see when the initial page was loaded. It was clear they wished to visit http://www.psicologia-online.com/. 

At 1.288 seconds, I see a request to deliver a banner of some sort, which does not seem too suspicious. This is typcical traffic, and comes from the same host as the website with the same request URI.

However, at 1.878 seconds, I see a request to get information from a completely different URI than the typical request, from http://seris.bin. The HTTP field of this certain packet is interesting:

Request URI: /20a958bc.js?cp=www.psicologia
Referer: http://www.psicologia-online.com\r\n
Host: seris.biz\r\n

Because this is such a strange change initiated roughly 0.6 seconds after the previous loading of other elements of the page, this leads me to believe it is suspicious. I will now look into the corresponding response by following the HTTP Stream.

As seen by the HTTP stream, it is clear the "302 Found" means the HTTP was redirected. The location, kanon-finale.com, tells us where the final part was redirected. This may be the location of the harmful payload. I will now look into the following domain when the user connects to the kanon-finale.com website.

The very next HTTP request after the redirect, we can see the host is kanon-finale.com. We will follow this HTTP stream and look for where this leads. Another thing to notice is that the request is being made to seris.biz, however the original website seems to be a parameter. This makes me wonder what kind of information they are collecting about the website in this manner. 

After following the packet stream, we see that the response is a gzip encoded JavaScript response in text, and that it is heavily obfuscated, or intentionally complex and obscured. The obfuscated text continues for many, many lines. I will continue to see what happens to the packets after the JavaScript is interpreted.

After looking through the packets, I saw in packet 92 a packet with (application/x-java-archive) as the identified response. This looks very suspicious to me, so I will look into the HTTP  stream.

In analyzing this HTTP stream, it is clear that the content-type is a Java archive. The User-Agent states Mozilla, but it also states Java/1.6.0_25. The response acknowledges the Java archive file, and lets out a string of text. Upon doing research, "PK" is the beginning of a ZIP file or a compressed Java file package (Kessler). When you continue to study the text, you can see "Zova44.class," which strongly leads me to believe that this is a Java file.

Through observing this PCAP file and looking into the user's traffic, I was able to observe many redirects, different URIs than the typical requests, redirections from websites kanon-finjale as well as seris.bin, and obfuscated JavaScript, and the almost instant download of a Java file, it is clear to me that this is not ordinary activity from the user and should be investigated further. I will use a tool, VirusTotal, to analyze it further. While I was unable to find a payload, the activity was strange enough to analyze this. 

I utilized VirusTotal to analyze the packet further. I uploaded the PCAP file to the multi-engine antivirus solution submission. As I suspected, because it doesn't have an executable file within the Wireshark files, it would not seem to be easily detected as malicious by 55 of the security vendors. The most popular threat label would be a generic trojan utilizing Java. 7 of the vendors, including Google and Nano-Antivirus, detected threat labels of 'trojan,' using labels like 'Java' which is what my analysis also concluded.

Looking further into the details that VirusTotal provided me about the PCAP file, I see that the kanon-finale.com requests were flagged as interesting. There were many Snort Alerts and Suricata Alerts that I did not catch, as attempted information leaks, attempted user privlege gains, web application attack, and a network trojan being detected. While I was able to determine that this file was malicious and the stream throughout the PCAP file was strange, I am glad I had access to this tool to analyze the packet further based on databases of past similar threats.

### Further Research + Conclusion
After further research, this seems like the pattern that may be used in a Blackhole Exploit page, with information stored as HTML, using Javascript to decode the payload, perhaps being a malicious Java applet. This is appropriate with the date of this file as well as the information found in the VirusTotal analysis of the packet, being a popular kind of crimeware around 2012 (Sophos). Certain Blackhole Exploits would obtain IPs, countries, browsers, and exploited targets like Java and compromised and legitimate webpages. This is something of a past artefact, but it is important to be familiar with how to identify the beginning of threats, and how to utilize resources and industry research to get a full picture of the security and insecurity landscape.

### Sources
https://www.malware-traffic-analysis.net/training-exercises.html
https://www.garykessler.net/library/file_sigs_GCK_latest.html
https://web.archive.org/web/20120906071340/http://sophosnews.files.wordpress.com/2012/03/blackhole_paper_mar2012.pdf
https://www.virustotal.com/
