+++
title = "Https for everything"
slug = ""
categories = [
  "tech",
]
tags = [
  "https",
]
date = "2016-11-18T00:59:57+08:00"
draft = false

+++


HTTPS is highly recommended for every web site, as a web developer I am building both my personal
and company app under https. Applying for the certificates could be a block for you to migrate/build
your site to https as you need to pay for it and it could take quite a while. Thanks to [letsencrypt](https://letsencrypt.org/) 
now we can have free open certificate authorify for our sites.

I will list the least steps to build a site by using [certbot](https://certbot.eff.org/) 

#### Install certbot

```sh
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
```

After that, you can run `./certbot-auto --help` to check if it's installed successfully.

#### Use certbot to generate certificates

```sh
./certbot-auto certonly --standalone -d www.yourdomain.com -d sub.yourdomain.com
```

Make sure `www.yourdomain.com` and `sub.yourdomain.com` is online at `80` port as this command will
check the validation. You can ignore `--standalone` at the moment, it's just a plugin
for certbot to generated software independent certificates. After running the command, there will be 4 files generated
at `/etc/letsencrypt`. They are `privKey.pem`, `fullchain.pem`, `cert.pem` and `chain.pem`.
Usually the web server only needs to point to the previous two for enableing https.
I will go through you how to point to certificates ad different server (nginx, apache) has different way.
You can google for it by yourself

#### Renew the certificates

The certificates are only valid for 90 days, luckily the renew is easy.

```sh
./certbot-auto renew
```

That's it. If certbot check the certificates are due for renewal, it will renew them.
I also create a daily cron task to renew it automatically.
`0 0 * * * ~/certbot-auto renew`

---

[Reference](https://certbot.eff.org/docs/using.html#getting-certificates-and-choosing-plugins)

