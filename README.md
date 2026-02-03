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

sudo systemctl stop apache2
sudo certbot certonly --standalone -d mail.example1.com -d mail.example2.com -d mail.example3.com
sudo systemctl start apache2

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

# Custom mail cron

Create:

```
nano /usr/local/bin/mail-certs.sh
```

Paste:

> NOTE: Don't forget to add your domains here! example1, example2 etc.

```
#!/bin/bash
sudo systemctl stop apache2
sudo certbot certonly --standalone -d mail.example1.com -d mail.example2.com -d mail.example3.com
sudo systemctl start apache2
```

Modify:

```
chmod +x /usr/local/bin/mail-certs.sh
```

Cron:

```
crontab -e
```

Add:

```
0 3 1,7,13,19,25 * *  /usr/local/bin/mail-certs.sh
```

This runs at 03:00 on the 1st, 7th, 13th, 19th, and 25th.
