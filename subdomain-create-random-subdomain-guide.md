# **How to Create a Randomized Subdomain for Security and Temporary Access**

### Introduction

Randomized subdomains are useful for security, temporary access, and preventing unauthorized tracking of predictable subdomains. This technique is commonly used by tunneling services like Localtunnel and Ngrok to create temporary public URLs for local applications.

In this article, we will explore how to generate randomized subdomains dynamically, register them using Cloudflare's API, and configure NGINX to handle requests securely.

### Why Use Randomized Subdomains?

1. **Security:** Prevents predictable subdomains, making it harder for attackers to guess.
2. **Temporary Access:** Ideal for testing, development, and ephemeral connections.
3. **Automation:** Allows dynamic subdomain assignment without manual intervention.

### Generating a Random Subdomain

To create a unique subdomain, we generate a random string and append it to our main domain.

#### **Bash Script**

```bash
RANDOM_SUBDOMAIN=$(openssl rand -hex 4)
echo "$RANDOM_SUBDOMAIN.example.com"
```

This generates a **random 8-character hexadecimal string** and appends it to `example.com`.

#### **Node.js Script**

```javascript
const crypto = require("crypto");
function getRandomSubdomain() {
  return crypto.randomBytes(4).toString("hex") + ".example.com";
}
console.log(getRandomSubdomain());
```

This creates a **random 8-character subdomain** using Node.js.

#### **Python Script**

```python
import secrets
random_subdomain = secrets.token_hex(4) + ".example.com"
print(random_subdomain)
```

Python’s `secrets` module ensures cryptographic randomness.

### Registering the Random Subdomain in Cloudflare

To make the subdomain accessible, we add a DNS A record using Cloudflare's API.

#### **Cloudflare API Script (Bash)**

```bash
ZONE_ID="YOUR_ZONE_ID"
API_TOKEN="YOUR_API_TOKEN"
RANDOM_SUBDOMAIN=$(openssl rand -hex 4)
DOMAIN="$RANDOM_SUBDOMAIN.example.com"
IP_ADDRESS="YOUR_SERVER_IP"

curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "Content-Type: application/json" \
     --data '{
        "type": "A",
        "name": "'$DOMAIN'",
        "content": "'$IP_ADDRESS'",
        "ttl": 1,
        "proxied": false
     }'
```

This script:

- Generates a random subdomain.
- Registers it in Cloudflare.
- Points it to your server.

### Configuring NGINX for the Random Subdomain

Once the subdomain is registered, we configure NGINX to serve requests.

#### **NGINX Configuration**

```nginx
server {
    listen 80;
    server_name *.example.com;

    location / {
        proxy_pass http://localhost:3000; # Replace with your app's port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

This allows NGINX to dynamically handle requests for any subdomain under `example.com`.

### Automating Subdomain Creation and Deletion

For temporary subdomains, we create a script to delete and recreate them periodically.

#### **Bash Script for Cycling Subdomains**

```bash
while true; do
    RANDOM_SUBDOMAIN=$(openssl rand -hex 4)
    DOMAIN="$RANDOM_SUBDOMAIN.example.com"

    # Register the new subdomain
    curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
         -H "Authorization: Bearer $API_TOKEN" \
         -H "Content-Type: application/json" \
         --data '{
            "type": "A",
            "name": "'$DOMAIN'",
            "content": "'$IP_ADDRESS'",
            "ttl": 1,
            "proxied": false
         }'

    echo "New subdomain registered: $DOMAIN"

    # Sleep for 10 minutes before replacing
    sleep 600

    # Delete the old subdomain (optional cleanup)
    curl -X DELETE "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/OLD_RECORD_ID" \
         -H "Authorization: Bearer $API_TOKEN"

done
```

### Conclusion

By implementing randomized subdomains, we improve security and flexibility in handling temporary access. This method is ideal for development, CI/CD pipelines, and tunneling services like Localtunnel.

Would you like to integrate this setup with GitHub Actions or Kubernetes for automation? Let’s discuss!
