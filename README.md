# traefik + letsencrypt

Inspired by https://github.com/htpcBeginner/docker-traefik I am trying to set up traefik to serve some dummy site with SSL from letsencrypt

## site overview

- the DNS entry dataweeder.cloud resolves to 127.0.0.1 for local development
- cloudflare is used for DNS-based ACME validation

## observed errors:

- letsencrypt is generating a suitable staging certificate
- traefik is NOT serving that
- I am unable to alter the default certificates

```
traefik  | time="2023-03-05T16:40:15Z" level=debug msg="No default certificate, fallback to the internal generated certificate" tlsStoreName=default
traefik  | time="2023-03-05T16:40:15Z" level=debug msg="Adding certificate for domain(s) *.dataweeder.cloud,dataweeder.cloud"
traefik  | time="2023-03-05T16:40:15Z" level=debug msg="No default certificate, fallback to the internal generated certificate" tlsStoreName=default
```

## reproducing the error

```
# set up a suitable domain with 127.0.0.1 forward in DNS
# in the .env file set the variables (replace dataweeder.cloud with your own DNS entry):

DOMAINNAME_CLOUD_SERVER=dataweeder.cloud
CLOUDFLARE_EMAIL=contact@dataweeder.cloud
CLOUDFLARE_API_KEY=<<key>>
	
LOCAL_IPS=127.0.0.1/32,10.0.0.0/8,192.168.0.0/16,172.16.0.0/12
CLOUDFLARE_IPS=173.245.48.0/20,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,141.101.64.0/18,108.162.192.0/18,190.93.240.0/20,188.114.96.0/20,197.234.240.0/22,198.41.128.0/17,162.158.0.0/15,104.16.0.0/13,104.24.0.0/14,172.64.0.0/13,131.0.72.0/22

docker compose up
```

Then the following log entry will show up:

```
traefik  | time="2023-03-05T16:17:53Z" level=debug msg="No default certificate, fallback to the internal generated certificate" tlsStoreName=default
traefik  | time="2023-03-05T16:19:32Z" level=debug msg="legolog: [INFO] [*.dataweeder.cloud] The server validated our request"
traefik  | time="2023-03-05T16:21:07Z" level=debug msg="No ACME certificate generation required for domains [\"whoami.dataweeder.cloud\"]." ACME CA="https://acme-staging-v02.api.letsencrypt.org/directory" routerName=whoami-rtr@docker rule="Host(`whoami.dataweeder.cloud`)" providerName=dns-cloudflare.acme
```

The obtained ACME cert file looks like this:


```
{
  "dns-cloudflare": {
    "Account": {
      "Email": "contact@dataweeder.cloud",
      "Registration": {
        "body": {
          "status": "valid",
          "contact": [
            "mailto:contact@dataweeder.cloud"
          ]
        },
        "uri": "https://acme-staging-v02.api.letsencrypt.org/acme/acct/91519634"
      },
      "PrivateKey": "key",
      "KeyType": "4096"
    },
    "Certificates": null
  }
}
```

However, https://dataweeder.cloud:
- shows the default traefik certificate and not the one from letsencrypt
- traefik is not serving the whoami route only a 404

What is going wrong here? How can I fix the SSL settings?
