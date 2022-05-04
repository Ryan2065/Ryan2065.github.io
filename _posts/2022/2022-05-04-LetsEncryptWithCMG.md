---
id: 726
title: 'CMG for the lab - Free Lets Encrypt Certs!'
date: 2021-02-06
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=726
permalink: /CMGLetsEncrypt/
categories:
    - ConfigMgr
    - CMG
    - Certificates
---

Let's say you're a IT Pro with your own ConfigMgr lab, and you want to hook up CMG, but you don't like messing with certificates. It seems like your only option is to buy a certificate. But... you're cheap!

Well, you can use Lets Encrypt to generate a trusted certificate for you if you already have a domain, with just a little bit of command line work! 

First up, start up WSL.  If you don't have WSL, Step 0 is to install it!

Lets Encrypt uses (maybe makes - not sure) a tool called CertBot to do your certificate work. So you first have to get that tool. Run these commands to get it:

```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot
sudo apt-get install openssl
```

The first command adds the certbot repository to your computer, so you can install directly from the command line. Then, you update any packages already existing so everything is up to date, then lastly you do the installs! One for certbot, and one for openssl so we can use their tools to export the cert to a pfx.

Now that you have certbot installed, generate the certificate!

```
sudo certbot -d mycoolanduniquecmgname.ephingadmin.com --manual --preferred-challenges dns certonly
```

So you probably don't want to generate a cert for my domain, so change that part. The string mycoolanduniquecmgname needs to be the name of your CMG server and needs to be unique across *all* CMG instances in your geographic instance. So be unique, something other than cmg1. 

The above command will start the process of generating a certificate and give you a manual step to do. The manual step comes after you say (y) to their questions, and give your email. It'll pop up with this prompt:

```
Please deploy a DNS TXT record under the name
_acme-challenge.mycoolanduniquecmgname.ephingadmin.com with the following value:

aa2DaK2Ckg-IaR17YDDEMWb2SJdSwaxRrx6S9T3y3BB

Before continuing, verify the record is deployed.
```

 What does that mean? Lets Encrypt doesn't just give out certs to anyone for your domain. So you have to prove you own this domain. How do you do that? Create a TXT record in your domain. Look up your hosts instructions. For Cloudflare, go to their [management dashboard](https://dash.cloudflare.com/login), click on your domain, click DNS, then click Add Record.

Once the record is added, it doesn't take effect immediately!!!!! You must verify it's implemented before hitting enter in the manual steps.

You can validate it with nslookup:

```
nslookup -q=TXT _acme-challenge.mycoolanduniquecmgname.ephingadmin.com
```

you should get this output if it's working:

```
Server:  UnKnown
Address:  192.168.85.154

Non-authoritative answer:
_acme-challenge.mycoolanduniquecmgname.ephingadmin.com text =

        "aa2DaK2Ckg-IaR17YDDEMWb2SJdSwaxRrx6S9T3y3BB"
```

Now, press enter and it will finish!

You'll get some good output that tells you where the cert is - that's super helpful!

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/mycoolanduniquecmgname.ephingadmin.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/mycoolanduniquecmgname.ephingadmin.com/privkey.pem
   Your cert will expire on 2022-08-02. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
```

Now, you have to go to that directory, but it's protected!  So, enter sudo super mode!

```
sudo su
cd /etc/letsencrypt/live/mycoolanduniquecmgname.ephingadmin.com
```

Lastly, create the cert. It will ask for an export password, this is a password you will put on the pfx - so you're creating one. You could leave it blank, but when I did that and tried to use it, blank was not a password I could give. So be sure you add a password!

```
openssl pkcs12 -export -out /tmp/cmgCert.pfx -inkey privkey.pem -in cert.pem -certfile chain.pem
```

This puts the cert in a new folder tmp and calls it cmgCert.pfx.

Lastly, let's bring it to civilization so we can access it in Windows.

Create a folder at the root of C - or do it somewhere else, I just hate typing paths with wrong slashes.

I created ```C:\CMGCert```

```
cp /tmp/certificate.pfx /mnt/c/cmgcert
```

Now you have a cert you can import to ConfigMgr for your cloud CMG!

Join me in 30 days when it expires and I have to figure out how to automatically update it!