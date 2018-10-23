---
layout: post
title:  "Archery - Vulnerability Assessment and Management Tool"
author: anand
categories: [ tutorial, introduction ]
image: assets/images/archeryblog/how_arc_work.png
featured: true
hidden: false
comments: false
---

<em> "Archery is an opensource vulnerability assessment and management tool which helps developers and pentesters to perform scans and manage vulnerabilities. Archery uses popular opensource tools to perform comprehensive scanning for web application and network. It also performs web application dynamic authenticated scanning and covers the whole applications by using selenium. The developers can also utilize the tool for implementation of their DevOps CI/CD environment."
</em>


Organisation continue to struggle with implementing a robust, scalable, effective and user-friendly Vulnerability Assessment and Management program that is capable of showing a quantifiable reduction in risk to the business despite the existence and need for these programs for the last decade or more. In the blog we’ll look how Archery will help you to organize your scanners and manage vulnerabilities in a automated way. Also we’ll get to know the features of Archery and how to use & utilize them.

 <blockquote ><em>A vulnerability assessment is the process of identifying, quantifying, and prioritizing the vulnerabilities in a system. — </em><a href="https://en.wikipedia.org/wiki/Vulnerability_assessment" data-href="https://en.wikipedia.org/wiki/Vulnerability_assessment" target="_blank">Wikipedia</a>
 </blockquote>


| <img src="/assets/images/archeryblog/vuln_cycle.png">| 
|:--:| 
| *Vulnerability Assessment and Management Cycle.* |

<br>

<blockquote>Vulnerability management is the “cyclical practice of identifying, classifying, remediating, and mitigating vulnerabilities”, particularly in software. Vulnerability management is integral to computer security and network security, and must not be confused with Vulnerability assessment. <a href="https://en.wikipedia.org/wiki/Vulnerability_management" data-href="https://en.wikipedia.org/wiki/Vulnerability_management" class="markup--anchor markup--pullquote-anchor" rel="nofollow noopener" target="_blank">Wikipedia</a></blockquote>

#### Challenges in Vulnerability Assessment and Management

Let’s explore those challenges which we commonly face while doing vulnerability assessment and management.

1. **Multiple Scanners**: We have multiple open source and commercial tools in our organization to perform vulnerability scanning. It is pain full task to manage multiple scanners.

2. **False Positive Removal**: Most profound challenge is to remove multiple false positive vulnerabilities from scanners and remove from future periodic scans.

3. **Vulnerability Prioritization**: Today, many organizations prioritize based on CVSS score and perform some level of asset importance classification within the process. However, this is still generating too much data for remediation teams to take targeted and informed action. In a larger organization, this process can result in tens of thousands — or even millions — of vulnerabilities detected.

4. **Remediation and Tracking**: Vulnerability remediation and tracking is the one of the most challenging part of vulnerability assessment and management. There is no technology that can easily and economically solve the problem, there are ways to enable better management through automation that can improve the process and influence user behavior. In other words, synchronizing communication between internal teams through workflow automation can help accelerate the remediation process. From simple ticket and task management to notifications and patch deployment, the ability to track the remediation process within a single unified view can eliminate the need to navigate and update multiple systems and potentially result in significant time savings.

## How Archery try to solve these challenges.

**1. Vulnerability Assessment and Management Tool**: Archery is an open source tool that helps you to plug vulnerability scanners like ZAP Scanner, Burp Scanner, OpenVAS etc. Once you configure your scanners Archery triggers scan into multiple scanners and correlates, collaborate all raw scans data, show them in a consolidated manner. Its also automate your scanners and do web dynamic scanning.

| <img src="/assets/images/archeryblog/how_arc_work.png">| 
|:--:| 
| *High-level Archery Architecture..* |

<br>
**2. Web Dynamic Authenticated Scanning**: Archery help you to perform web application dynamic authentication using inbuilt selenium integrated solution. Archery storing application authenticated cookies value into cookies_db and using ZAP Replacer its replace cookies value from headers. It’s also using selenium based authentication and grab cookies value runtime.

| <img src="/assets/images/archeryblog/arcyery_dynacmi.png">| 
|:--:| 
| *Web application Dynamic Authentication Solution by Archery.* |

<br>
**3. Manage False Positives Vulnerabilities:** Archery help you on removing false positive vulnerabilities after scan completion. Its very important to remove false positives vulnerabilities and work on new discovered vulnerabilities from all future scans.

| <img src="/assets/images/archeryblog/false_positive.png">| 
|:--:| 
| *False Positive marked vulnerability.* |

<br>
**4. JIRA Ticketing:** Archery has JIRA Ticketing system plugin that you can plug with Archery tool and raise tickets. 

| <img src="/assets/images/archeryblog/jiraticket.png">| 
|:--:| 
| *JIRA Ticket Integrated and raise vulnerability ticket..* |

<br>

| <img src="/assets/images/archeryblog/jira2.png">| 
|:--:| 
| *After JIRA Ticket raised issue tracking..* |

<br>

**5. Web Scanners Plugin & Parser:** We are working on to writing plugins and parsers for open source and commercial scanners.

Currently Archery supports below Web Scanners plugin and parser —

- ZAP Scanner (Plugin and XML Parser) Open Source.
- Burp Scanner (Plugin and XML Parser) Commercial or Pro Version.
- arachni scanner (XML Parser) Open Source.
- Netsparker (XML Parser) Commercial Scanner.
- Webinspect (XML Parser) Commercial Scanner.
- Acunetix (XML Parser) Commercial Scanner.

<br>

**6. Network Scanners Plugin & Parser:** There are multiple open source and commercial tools & scanners are available to perform network vulnerability assessment. We also working on to integrate those scanners plugin with Archery.

Currently we supports below Network scanners —

- OpenVAS (Plugin and XML Parser) Open Source.
- Nessus Scanner (nessus file Parser) Commercial Scanner.

<br>

**7. Tools integrated:** We are doing multiple experiment with multiple available open source security tool to integrate with Archery. We have integrated some of popular tool which helps on doing Vulnerability Assessment.

- SSLScan

| <img src="/assets/images/archeryblog/archery_ssl.png">| 
|:--:| 
| *SSLScan tool integrated into Archery tool.* |

<br>

- Nikto

| <img src="/assets/images/archeryblog/nikto.png">| 
|:--:| 
| *Nikto tool Integrated into Archery Tool.* |

<br>

- Nmap + Vulners

| <img src="/assets/images/archeryblog/nmap_vulns.png">| 
|:--:| 
| *Nmap and Vulners Integrated into Archery tool.* |

<br>

**8. Integrated OSINT:** Archery has integrated some OSINT tool like Sublist3r to extract sub-domains which is useful to have run vulnerability assessment. Currently we are experimenting with various OSINT tools and trying to integrate with Archery tool.

| <img src="/assets/images/archeryblog/domainlist.png">| 
|:--:| 
| *OSINT way subdomain Sublist3r integrated.* |

<br>

**9. Dashboards:** The dashboard provides a summarised view of vulnerability assessment results. Vulnerability analytics has monthly trends of High, Medium and Low findings. Also it has web and network pie & bar charts which gives overall summary of findings.

<br>

| <img src="/assets/images/archeryblog/overall_dashboard.png">| 
|:--:| 
| *Archery Dashboard.* |

<br>

| <img src="/assets/images/archeryblog/pie&bar.png">| 
|:--:| 
| *Archery Dashboard..* |

## References

- [https://github.com/archerysec/archerysec](https://github.com/archerysec/archerysec)
- [https://archerysec.info/](https://archerysec.info/)
- [https://www.computerworld.ph/print-article/86303/](https://www.computerworld.ph/print-article/86303/)
- [https://www.networkworld.com/article/2980303/security/three-key-challenges-in-vulnerability-risk-management.html](https://www.networkworld.com/article/2980303/security/three-key-challenges-in-vulnerability-risk-management.html)


