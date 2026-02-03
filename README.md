# Custom-Certbot

Often, Let's Encrypt certbot refuses to update certs if you have manual added domain. Often required for `mail.example.com` domains with DNS challenge, which cannot host a `ACME challenge`. So renewal fails.

Thus, we need a custom certbot.

Disable certbot:

```
sudo systemctl disable certbot.timer
sudo systemctl stop certbot.timer
sudo mv /etc/cron.d/certbot /etc/cron.d/certbot.disabled
```

# Custom bot

Create:

```
nano /usr/local/bin/certs.sh
```

Paste:

```
#!/bin/bash
for cert in $(certbot certificates | grep "Certificate Name:" | awk '{print $3}'); do
    if [ "$cert" != "mail.example.com" ]; then
        certbot renew --cert-name "$cert"
    fi
done
```

Modify:

```
chmod +x /usr/local/bin/certs.sh
```

Cron:

```
crontab -e
```

Add:

```
30 2 * * * /usr/local/bin/certs.sh
```
