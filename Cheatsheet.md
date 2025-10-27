# Run shell script
sh filename.sh

./script.sh

# DNS / host discovery
ping example.com

nslookup example.com

whois example.com

# WhatWeb
whatweb https://example.com

whatweb https://example.com -v

whatweb 192.168.1.1-192.168.1.225 -v --no-errors --log-verbose=whatweb_verbose.txt


# theHarvester
theHarvester -d example.com -b linkedin -l 1000

# Clone from GitHub
git clone https://github.com/owner/repo.git

# Sherlock (example workflow)
git clone https://github.com/sherlock-project/sherlock.git

cd sherlock

python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt

python3 -m sherlock username

# Local discovery
arp -a

sudo netdiscover

# Nmap examples
nmap 192.168.10.5

sudo nmap -sS 192.168.10.5

nmap -sT 192.168.10.5

sudo nmap -sU 192.168.10.5

sudo nmap -O 192.168.10.5

sudo nmap -sV --version-intensity 6 192.168.10.5

sudo nmap -A 192.168.10.5

nmap -sn 192.168.10.0/24

nmap -p 22,80,443 192.168.10.5

nmap -p 1-100 192.168.10.5

sudo nmap -oN saved_scan.txt -sS 192.168.10.5
