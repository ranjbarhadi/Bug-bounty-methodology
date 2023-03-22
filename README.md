# introduction:
What you see here is an overview of how i approach reacon.

## Roadmap:
```mermaid
graph LR
A((Project setup)) --> B(recon-ng)
B --> C(isup)
C --> D(nMap)
C --> F(Sn1per)
B --> E(httpx)
E --> K(paramspider)
E --> G(Eyewitness)
B --> H(osmedeous)
B --> I(Heartbleed)
B --> J(takeover)
```

## Project setup
Create Folders (subdomains, urls, ips,patterns,params,javascript). here is a good oneliner:
```bash
export projectname=[name]
```
``` bash
mkdir ~/$projectname && cd ~/$projectname && mkdir subdomains urls ips patterns params javascripts downloads
```

## Recon
#### Recon-ng
install recon-ng. use it to search Passively for subdomains ips ports
here are some of the commands you may need
```bash
workspaces create

#insert domains
db insert domain

#insert company name
db insert company

# if you have a narrow scope you can enter the hosts using this command
db insert host

# now load some modules and then run
modules load recon/domai.....
run

# save the list 
modules load reporting/list
# save ip addresses
options set FILENAME ~/$projectname/ips/ips.txt
run
# save domains
options set COLUMN host
options set FILENAME ~/$projectname/subdomains/subdomains.txt
run
```

now make sure you are in your working directory:
```bash
cd ~/$projectname
```

#### Check if ips are alive:
check if ips are alive using isup.
```bash
cd ~/isup && rm tmp -R &&./isup.sh ~/$projectname/ips/ips.txt && cp ~/isup/tmp/valid-ips.txt ~/$projectname/ips/valid-ips.txt
```

#### Validate subdomains:
validate domains using httpx:
```bash
cat ~/$projectname/subdomains/subdomains.txt | httpx -verbose > ~/$projectname/urls/urls.txt
```

#### Use Nmap Aggressive Scan:
```bash
nmap -iL ~/$projectname/ips/valid-ips.txt -sSV -A -T4 -O -Pn -v -F -oX $projectname_nmap_result.xml
```
#### Sn1per - WebApp Mode: 
```bash
sniper -f ~/$projectname/ips/ips.txt -m massweb -w $projectname && mkdir ~/$projectname/sniper && cp /usr/share/sniper/loot/workspace/judgeme ~/$projectname/sniper/ -R
```
then save the result and copy them to our working folder!
```bash
# complete this
```

#### Eyewitness to take Screenshots of all URLS
```bash
cd ~/$projectname/ && eyewitness -f ~/$projectname/urls/urls.txt
```
```bash
zip -r $projectname.zip foldername
```

#### ParamSpider 
to Hunt for URLS with Parameters automatically from wayback machine
```bash 
cat ~/$projectname/urls/urls.txt | while IFS="" read -r p || [ -n "$p" ] 
do 
	python3 ~/ParamSpider/paramspider.py --domain "$p" --exclude woff,png,svg,php,jpg --output ~/$projectname/params/$(echo $p | sed 's/https:\/\///g ; s/http:\/\///g').txt 
done && cat ~/$projectname/params/* > ~/$projectname/params/all.txt
```
Technique to Clean Params from XSS:
```bash
cp ~/$projectname/params/all.txt | sed 's/FUZZ/[whatever-you-like]/g' > ~/$projectname/params/all-changed.txt 
```

#### Nuclei
for scanning everything:
```bash
nuclei -l ~/$projectname/urls/urls.txt -o ~/$projectname/nucleai_cve_result.txt
```
for cve only:
```bash
nuclei -l ~/$projectname/urls/urls.txt -t /root/nuclei-templates/cves/ -o ~/$projectname/nucleai-cve-result.txt
```

#### Jaeles:
```bash
cat ~/$projectname/urls/urls.txt | jaeles scan -s 'cves' -s 'sensitive' -s 'fuzz' -s ‘common' -s 'routines' report -o ~/$projectname/Jaeles-cve-result.txt --title "[$projectname] Jaeles Full Report"
```

#### chopchop 

```bash
~/ChopChop/gochopchop scan --url-file ~/$projectname/urls/urls.txt --threads 4 -e csv --export-filename ~/$projectname/chopchop-result.txt
```

#### inception
```bash
# write this
```

#### get all urls
Gau - for realtime URL extraction when performing manual search so you can have urls to attack. Hunt for Links that have Parameters by using gau (Get all URLS) and displaying all links that have params: 
```bash
# this might need some work
./gau --blacklist png,jpg,gif ~/$projectname/urls/urls.txt --o ~/$projectname/urls/urls-gau.txt --verbose
```

#### download all of the javascripts with JSScanner 
first download everything:
```bash
~/JSScanner/script.sh  ~/$projectname/params/all.txt && cp  Jsscanner_results/ ~/$projectname/javascripts -r
```

#### download pdf,xls,docs files
```bash 
cat ~/$projectname/urls/urls.txt | while IFS="" read -r p || [ -n "$p" ] 
do 
	proxychains python3 ~/metagoofil/metagoofil.py -d $p -t pdf,doc,xls,ppt,docs,xls,pptx -e 30 -l 12 -o ~/$projectname/downloads/
done
```

#### Osmedeous, search for vulenerablities: 
Select one of these actions
directly run on vuln scan and directory scan on list of domains :
```bash
osmedeus scan -f vuln-and-dirb -t ~/$projectname/subdomains/subdomains.txt
```
scan list of targets :
```bash
osmedeus scan -T ~/$projectname/subdomains/subdomains.txt
```
get target from a stdin and start the scan with 2 concurrency :
```bash
cat ~/$projectname/subdomains/subdomains.txt | osmedeus scan -c 2
```
start a simple scan with default 'general' flow :
```bash
osmedeus scan -t sample.com
```
then save the result and copy them to our working folder!
```bash
# complete this
```

#### Check for Heartbleed:

```bash
cat ~/$projectname/subdomains/subdomains.txt | while read line ; do echo "QUIT" | openssl s_client -connect $line:443 2>&1 | grep 'server extension "heartbeat" (id=15)' || echo $line: safe; done
```

#### Takeover Tool: 
```bash
takeover -l ~/$projectname/subdomains/subdomains.txt -v -t 10
```
 
#### Use Smuggler
on URLs list to test for http requests that could desync, and posting multiple chunked requests to smuggle external sources so the backend server will forward the request with cookies, data to the front end server

```bash
cat ~/$projectname/urls/urls.txt | python3 smuggler.py -l ~/$projectname/smuggler-result.txt
```
 
#### Find XSS Vulnerabilities from Paramspider & Dalfox New!
Since we have params urls from paramspider, dalfox needs to know where to inject, and you can define it with XSS instead of FUZZ, so here is a command to replace this from the result, and create a new list to be used on dalfox.
```bash
cat ~/$projectname/params/all.txt | sed 's/FUZZ/XSS/g' > ~/$projectname/all-dalfox-ready.txt
```
You are now ready for parsing the urls into dalfox in pipe mode:
```bash
cat ~/$projectname/all-dalfox-ready.txt | dalfox pipe | cut -d " " -f 2 > ~/$projectname/dalfox-all-result.txt
```
```bash
dalfox file ~/$projectname/all-dalfox-ready.txt | cut -d " " -f 2 > ~/$projectname/dalfox-all-result.txt
```
For Deeper Attacks add this: `--deep-domxss`

Silence --silence Prints only PoC When found and progress

## After recon
Scanning Javascript Files for Endpoints, Secrets, Hardcoded credentials, IDOR, Open redirect and more Paste URLS into `alive.txt`. Run script `alive.txt` - Examine the results using GF advanced patterns

Use tree command, cat into subdirectories:
```
cat * */*.txt
cat */*.js | gf api-keys	
cat /*/*.txt | gf ssrf > /root/Desktop/ssrf.txt
```

Or New Method with GitLeaks: New!

Scan a Directory with Javascripts, Files, Json Etc.. for Secrets!
```
gitleaks --path=/directory -v --no-git
```

Scan a File with Any Extension for Secrets!
```
gitleaks --path=/file.xxx -v --no-git
```

#### After Recon: New!

When you find Keys/Tokens - Check from here: https://github.com/streaak/keyhacks

Examine the Results Manually

B) Pattern Check Example for Results with gf & gf-patterns: 

After you have the Parameters Gathered, we want to check for specific patterns and possible vulnerable URLs that can be attacked using Meg or other Fuzzing Tools.
```
cat /root/Desktop/Bounty/params.txt | gf xss | sed 's/FUZZ/ /g' >> /root/Desktop/Bounty/xss_params_forMeg.txt
```

********************************************************************************************************************

OSINT & Passive Amplified Attacks: (Raspberry Pi)

OSINT:

Perform OSINT using spiderfoot

One off 1337 Powerful Command Attacks with amass:




#### use Gotty - https://github.com/yudai/gotty

```
gotty -p 1337 -w recon-ng 
```

# Tools

Here are some of the tools that we use when we perform Live Recon Passive ONLY on Twitch:

1) Recon-ng
https://github.com/lanmaster53/recon-ng
2) httpx
https://github.com/projectdiscovery/httpx
3) isup.sh
https://github.com/gitnepal/isup
4) Arjun
https://github.com/s0md3v/Arjun
5) jSQL
https://github.com/ron190/jsql-injection
6) Smuggler
https://github.com/defparam/smuggler
7) Sn1per
https://github.com/1N3/Sn1per
8) Spiderfoot 
https://github.com/smicallef/spiderfoot
9) Nuclei
https://github.com/projectdiscovery/nuclei
10) Jaeles
https://github.com/jaeles-project/jaeles
11) ChopChop
https://github.com/michelin/ChopChop
12) Inception
https://github.com/proabiral/inception
13) Eyewitness
https://github.com/FortyNorthSecurity/EyeWitness
14) Meg
https://github.com/tomnomnom/meg
15) Gau - Get All Urls
https://github.com/lc/gau
16) Snallygaster
https://github.com/hannob/snallygaster
17) NMAP
https://github.com/nmap/nmap
18) Waybackurls
https://github.com/tomnomnom/waybackurls
19) Gotty
https://github.com/yudai/gotty
20) GF
https://github.com/tomnomnom/gf
21) GF Patterns
https://github.com/1ndianl33t/Gf-Patterns
22) Paramspider
https://github.com/devanshbatham/ParamSpider
23) XSSER
https://github.com/epsylon/xsser
24) UPDOG
https://github.com/sc0tfree/updog
25) JSScanner
https://github.com/dark-warlord14/JSScanner
26) Takeover
https://github.com/m4ll0k/takeover
27) Keyhacks
https://github.com/streaak/keyhacks
28) S3 Bucket AIO Pwn
https://github.com/blackhatethicalhacking/s3-buckets-aio-pwn
29) BHEH Sub Pwner Recon
https://github.com/blackhatethicalhacking/bheh-sub-pwner
30) GitLeaks
https://github.com/zricethezav/gitleaks
31) Domain-2IP-Converter
https://github.com/blackhatethicalhacking/Domain2IP-Converter
32) Dalfox
https://github.com/hahwul/dalfox
33) Log4j Scanner
https://github.com/Black-Hat-Ethical-Hacking/log4j-scan
34) Osmedeus
https://github.com/j3ssie/osmedeus
35) getJS
https://github.com/003random/getJS
36) MetaGoofil
https://github.com/opsdisk/metagoofil
