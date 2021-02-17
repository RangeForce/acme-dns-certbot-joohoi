# acme-dns-certbot-joohoi

An example [Certbot](https://certbot.eff.org) client hook for [acme-dns](https://github.com/joohoi/acme-dns).

This authentication hook automatically registers acme-dns accounts and prompts the user to manually add the CNAME records to their main DNS zone on initial run. Subsequent automatic renewals by Certbot cron job / systemd timer run in the background non-interactively.

Requires Certbot >= 0.10, Python requests library.

## Installation

1) Install Certbot using instructions at [https://certbot.eff.org](https://certbot.eff.org)

2) Make sure you have the [python-requests](http://docs.python-requests.org/en/master/) library installed.

3) Download the authentication hook script: `curl -s -o /etc/letsencrypt/acme-dns-auth.py https://raw.githubusercontent.com/RangeForce/acme-dns-certbot-joohoi/master/acme-dns-auth.py`

4) Make it executable: `chmod 0700 /etc/letsencrypt/acme-dns-auth.py`

5) Set necessary environment variables to configure the script's execution:

- The only one you **have to** set is `ACMEDNS_URL`

```text
export ACMEDNS_URL="https://auth.some-other-domain.com"
export ACMEDNS_STORAGE_PATH="/path/to/some/other/location"
export ACMEDNS_ALLOW_FROM='["192.168.10.0/24", "::1/128"]'
export ACMEDNS_FORCE_REGISTER=True
```

## Usage

On initial run:

```text
certbot certonly \
  --manual \
  --manual-auth-hook /etc/letsencrypt/acme-dns-auth.py \
  --preferred-challenges dns \
  --debug-challenges \
  --domain example.org \
  --domain \*.example.org
```

Note that the `--debug-challenges` is mandatory here to pause the Certbot execution before asking Let's Encrypt to validate the records and let you to manually add the CNAME records to your main DNS zone.

After adding the prompted CNAME records to your zone(s), wait for a bit for the changes to propagate over the main DNS zone name servers. This takes anywhere from few seconds up to a few minutes, depending on the DNS service provider software and configuration. Hit enter to continue as prompted to ask Let's Encrypt to validate the records.

After the initial run, Certbot is able to automatically renew your certificates using the stored per-domain acme-dns credentials. 
