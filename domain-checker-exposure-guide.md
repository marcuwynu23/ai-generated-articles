# 🔍 How to Check If Subdomains Are Exposed Using crt.sh and Subfinder  

When managing a domain, one of the most important security checks is to see whether any of its **subdomains are exposed** to the internet. Exposed subdomains can reveal forgotten services, staging environments, or sensitive applications that attackers may exploit.  

In this article, we’ll walk through two powerful ways to discover and check subdomains: **crt.sh** (certificate transparency logs) and **subfinder** (an automated subdomain discovery tool). We’ll also cover how to verify which of those subdomains are actually live.  

---

## 1. Why Subdomain Enumeration Matters  
Subdomains often host applications, APIs, or services that aren’t meant to be public. Examples include:  
- `dev.example.com` (development environment)  
- `crm.example.com` (customer management system)  
- `secrets.example.com` (sensitive dashboards)  

If these are exposed, they can become easy targets for attackers. By enumerating and checking subdomains, you stay aware of what’s visible and reduce your attack surface.  

---

## 2. Using **crt.sh**  

[crt.sh](https://crt.sh) is a free service that indexes **certificate transparency logs**. Whenever a TLS/SSL certificate is issued, it gets logged publicly. If a subdomain has ever had a certificate, you can often find it here.  

### Manual Check  
1. Visit [crt.sh](https://crt.sh).  
2. In the search box, type:  
   ```
   %.yourdomain.com
   ```
   The `%` acts as a wildcard to return all subdomains.  

You’ll get a list of certificates containing subdomains.  

### Command Line (with `curl` + `jq`)  
On Linux or macOS:  
```bash
sudo apt install jq -y   # install jq if missing

curl -s "https://crt.sh/?q=%25.yourdomain.com&output=json"   | jq -r '.[].name_value'   | sed 's/\*\.//g'   | sort -u > crtsh-subs.txt
```

This command saves all unique subdomains found in SSL history into `crtsh-subs.txt`.  

---

## 3. Using **subfinder**  

[Subfinder](https://github.com/projectdiscovery/subfinder) is a fast, community-driven tool that queries multiple data sources to find subdomains.  

### Installation (Ubuntu):  
```bash
sudo apt update
sudo apt install golang-go -y
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# add Go binaries to PATH
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

### Usage:  
```bash
# basic scan
subfinder -d yourdomain.com

# only show subdomains (no logs)
subfinder -d yourdomain.com -silent

# save results to a file
subfinder -d yourdomain.com -silent -o subfinder-subs.txt
```

---

## 4. Checking Which Subdomains Are Exposed  

Finding subdomains is only the first step. Next, you need to confirm which ones are actually **alive** (responding to HTTP/HTTPS requests).  

For this, we use **httpx**, another tool from ProjectDiscovery.  

### Install httpx:  
```bash
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

### Run Exposure Check:  
```bash
cat subfinder-subs.txt crtsh-subs.txt | sort -u   | httpx -silent -status-code -o live-subs.txt
```

Example output:
```
https://www.example.com [200]
https://crm.example.com [302]
http://dev.example.com [403]
```

- **200, 301, 302** → Live and accessible.  
- **403** → Exists but restricted.  
- **Timeout** → Likely inactive.  

---

## 5. One-Shot Command (crt.sh + subfinder + httpx)  

For convenience, you can combine everything into one pipeline:  

```bash
(
  subfinder -d yourdomain.com -silent
  curl -s "https://crt.sh/?q=%25.yourdomain.com&output=json"     | jq -r '.[].name_value'
) | sed 's/\*\.//g' | sort -u | httpx -silent -status-code -o exposed.txt

cat exposed.txt
```

This will:  
1. Collect subdomains from both **subfinder** and **crt.sh**.  
2. Clean up duplicates and wildcards.  
3. Test which are live with **httpx**.  
4. Save the exposed subdomains into `exposed.txt`.  

---

## ✅ Conclusion  

- **crt.sh** is great for historical data on SSL certificates.  
- **subfinder** is fast and aggregates multiple sources.  
- **httpx** lets you confirm which subdomains are actually exposed.  

By combining these tools, you gain a complete picture of your domain’s subdomain footprint and can spot potential exposures before attackers do.  
