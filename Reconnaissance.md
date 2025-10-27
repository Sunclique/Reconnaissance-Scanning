# 1. Overview and legal/ethical notice
Reconnaissance is the early phase of any penetration test or security assessment. Typical objectives include enumerating network hosts, discovering exposed services and versions, identifying technologies that power a target’s public assets, and collecting public contact information. While the techniques in this document are widely used by security professionals, they are intrusive if used without permission.

_Legal & ethical reminder:_ Only perform scans and data collection on systems you own or on which you have written authorization. Unauthorized scanning, enumeration, or data harvesting may violate laws and terms of service.

# 2. Running executable files on Linux
Linux executes shell scripts and programs differently from Windows. Common usage patterns:

•	Make a script executable and run it:

chmod +x ./script.sh

./script.sh

•	Run a shell script with sh:

sh filename.sh

•	Execute Python modules or scripts:

python3 script.py

python3 -m modulename

Always prefer ./filename (with a proper executable bit) or invoking an interpreter explicitly (sh, bash, python3) to ensure predictable behavior.

# 3. High-level reconnaissance: objectives and common data points
During information gathering your goal is to map the target’s external footprint and gather data that informs deeper testing. Typical items of interest:

•	Public IP addresses and IP ranges

•	Domain names and DNS records

•	Public-facing services and their versions (HTTP, SSH, SMTP, etc.)

•	Exposed web technologies (CMS, frameworks, server software, plugins)

•	Public contact details (corporate e-mails, employee contacts)

•	Subdomains and related domains

•	Hosts on the same local network (when in-scope)

These data points are used to plan focused scans, reduce noise, and prioritize targets.

# 4. Basic network and domain discovery
Useful commands for initial discovery:

•	Check host reachability:

ping example.com

•	Query DNS records (A, MX, TXT, NS, etc.):

nslookup example.com

or

dig example.com any

•	Get domain registration and registrar information:

whois example.com

These tools provide initial evidence of ownership, name servers, and sometimes contact details for administrators.

# 5. Website fingerprinting with WhatWeb
_Purpose:_ WhatWeb identifies technologies used by a website (web server, frameworks, CMS, analytics, plugins). It is a common tool for passive fingerprinting and quick tech enumeration.

_Basic usage examples:_

•	Simple scan:

whatweb https://example.com

•	Verbose output (more detected items & plugin details):

whatweb https://example.com -v

•	Scan an IP range (note: this will iterate every address in the range):

whatweb 192.168.1.1-192.168.1.225

•	Add verbosity and control error output:

whatweb 192.168.1.1-192.168.1.225 -v --no-errors

•	Save verbose (detected plugins / verbose log) to a file:

whatweb https://example.com -v --log-verbose=whatweb_verbose_output.txt

_Aggression / intensity:_ Many fingerprinting tools include an “aggression” or intensity parameter that increases the number or intrusiveness of checks. Use higher aggression values cautiously and only when authorized, because increased aggression can be more detectable and may trigger defensive systems. Consult the tool’s help output for supported ranges and semantics:

whatweb –help

_Operational note:_ Scanning an entire address range will attempt every IP in that range; many addresses may be unused. Use --no-errors or similar options if you wish to suppress noisy error output, but remember suppressed errors do not mean the hosts are responsive.

# 6. Email harvesting with theHarvester
_Purpose:_ theHarvester collects publicly available email addresses and subdomains from multiple public sources; useful for building social-engineering safe lists and mapping external contacts.

_Example usage:_

theHarvester -d example.com -b linkedin -l 1000

•	-d specifies the domain to search.

•	-b chooses the data source (e.g., google, bing, linkedin, all).

•	-l limits the number of results.

_Operational notes:_

•	Use -b all to aggregate multiple sources, but expect duplicate results.

•	Respect rate limits and the terms of the data sources.

•	Verify harvested addresses manually before use — not all automatically collected addresses are valid or current.

# 7. Finding and installing tools from GitHub
Open-source reconnaissance tools are commonly hosted on GitHub. A reproducible workflow:

1.	Search with your browser: tool-name github (e.g., information gathering tool github).

2.	Choose the official or well-maintained repository.

3.	Clone locally:

git clone https://github.com/owner/repository.git

4.	Follow the project’s README for installation and usage. Typical steps might include:

cd repository

_install dependencies (examples)_

pip3 install -r requirements.txt

_or if the repo provides an installer_

./install.sh

_Notes:_ Some repositories use different invocation methods (a CLI entrypoint, python3 -m modulename, or a wrapper script). Always prefer the instructions provided by the repository.

# 8. Example: running Sherlock (module example and dependency management)
Sherlock is a common OSINT tool used to find usernames across many sites. When installing or running third-party tools from GitHub, you may face dependency issues.

_Common workflows:_

•	Clone then run (when repository provides a module entrypoint):

git clone https://github.com/sherlock-project/sherlock.git

cd sherlock

_recommended: create and use a virtual environment_

python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt

python3 -m sherlock username_to_search

_Troubleshooting dependency errors:_
•	If Python module X is reported missing (e.g., network/requests futures packages), you can:

o	Use pip to install into an active virtual environment:

o	pip install requests-futures

o	If you encounter messages about a "managed environment" (where pip to user site is restricted), install the missing system package using the OS package manager (only when you have privilege and it’s appropriate):

o	sudo apt install python3-requests-futures

o	Prefer virtual environments (venv) to avoid system-wide changes.


_Important:_ Do not install arbitrary system packages without understanding their provenance. When in doubt, consult the project’s documentation or issue tracker.

# 9. Local network discovery: arp, netdiscover, and limitations
•	View current ARP table:
arp -a

arp shows IP/MAC mappings learned by your system; it relies on prior interaction with hosts (e.g., ARP requests during network traffic). Thus, ARP may not list devices that have not recently communicated with your host.

•	Actively discover hosts on a local network:

sudo netdiscover

netdiscover actively scans the local network and reports IP, MAC and vendor if available. It is useful for quickly enumerating devices on the same subnet.

_Operational caveat:_ Active discovery is more intrusive than passive methods; perform it only in-scope.

10. Port & service scanning with Nmap
Nmap is a fundamental tool for enumerating open ports, determining service versions, and performing OS detection. Below are commonly used flags and workflows with explanation.

_Basic host scan:_

nmap 192.168.10.5

_Scan a range or CIDR:_

nmap 192.168.10.1-225

or by CIDR

nmap 192.168.10.0/24

_SYN scan (stealthy, requires root):_


sudo nmap -sS 192.168.10.5

•	Sends TCP SYN and awaits response; does not complete full TCP handshake. More stealthy than a full connect scan but still detectable.

_TCP connect scan (full TCP handshake, does not require root):_

nmap -sT 192.168.10.5

•	Performs full TCP connection; more likely to be logged on the target.

_UDP scan:_

sudo nmap -sU 192.168.10.5

•	UDP scanning is slower and often less reliable due to lack of response and ICMP rate limiting; patience required.

_OS detection:_

sudo nmap -O 192.168.10.5

•	Capital -O triggers OS detection; accuracy depends on host fingerprintability and network conditions.

_Service/version detection:_

sudo nmap -sV 192.168.10.5

•	-sV probes open ports to determine service names and versions.

_Version intensity control (0–9):_

sudo nmap -sV --version-intensity 6 192.168.10.5

•	Lower values reduce probing; higher values increase the number/depth of probes. Default is typically moderate (for many Nmap builds it is 7). Use higher intensity only when justified and authorized.

_Aggressive scan (OS + version + scripts):_

sudo nmap -A 192.168.10.5
•	Equivalent to enabling OS detection, version detection and default NSE script scans; very noisy — use with caution.
_Host discovery only (to see which hosts are up):_

nmap -sn 192.168.10.0/24
•	-sn (no port scan) performs host discovery only.

_Port selection examples:_

•	Single port:

nmap -p 22 192.168.10.5

•	Multiple specific ports:

nmap -p 22,80,443 192.168.10.5

•	Range of ports:

nmap -p 1-1000 192.168.10.5

•	Scan top 100 most common ports:

nmap -F 192.168.10.5

_Saving output:_
•	Redirect stdout to a file (simple):

sudo nmap -sS 192.168.10.5 >> output.txt

•	Use Nmap native output options to save structured results:

sudo nmap -oN output_normal.txt -sS 192.168.10.5   _normal_

sudo nmap -oG output_grepable.txt -sS 192.168.10.5  _greppable_

sudo nmap -oX output.xml -sS 192.168.10.5           _XML (machine readable)_

•	Example: show on terminal and save:

sudo nmap -oN saved_scan.txt -sS 192.168.10.5

_Interactive progress:_

For long scans, Nmap displays progress; you can scroll output or use the up arrow in an interactive terminal to retrieve previous commands.

_Manual / help:_

man nmap

or

nmap --help

# 11. Saving outputs and logging results
•	Prefer tool-native logging flags (-oN, --log-verbose, -oX) when available — these formats are easier to parse and retain metadata.

•	Use a consistent naming convention for outputs (target, date, scan type), e.g.:

targetname_YYYYMMDD_nmap_syn_normal.txt

•	Keep logs in an organized directory structure and ensure sensitive outputs are stored securely and access-controlled.

