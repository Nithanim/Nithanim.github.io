---
layout: posts
title:  "The Quirks of CNAME and SRV"
date:   1970-01-01 02:00:00 +0100
author: Nithanim
categories: misc 
---

Did you ever try to setup a mail server or srv records for some application and wondered why it misbehaves or does not work at all? Well, me too so I share my experiences and fixes with you!

## The lost mail
At first we take a look at CNAME records.
They more or less forward a subdomain to another domain by simply specifying both the names of the source and the target. For example we can forward `src.example.com.` to `dest.example.com.`.
As an advanced and more real world example we now try to setup a common domain that exposes a web- and mailserver.

The webserver will be available under `www.example.com` and the mailserver gets `mail.example.com`.
We quickly setup the the www to an A record and let all other subdomains simply point to it by CNAME because we figure that it will save us some work when we change our hoster (or the IP-address in general).
The mailserver gets its MX record for `example.com.` pointing to `mail.example.com.`.

Everything should now work flawlessly but we figure that it would be a good idea to be able to type `example.com` in the browser and be automatically forwarded to `www` instead of getting the boring "unreachable" message and being forced to type the "www." in front of it.
We quickly setup a CNAME record for `@` (the shorthand for the domain itself, `example.com`) pointing to `www.example.com.`.

After some time we realize that our mailserver no longer receives our emails.
The problem is that a CNAME record also somehow counts as a MX record and is prioritized over the MX record.
The mailserver that is listening on `mail.example.com` will therefore not get any mails.
The only fix is to specify the `@` as A record and let it point to the same IP-address as `www`.

## The service that sometimes works and sometimes doesn't

SRV records are not only really long and weird to specify but they sometimes behave completely random.
SRV are not automatically used.
They are an additional record a application has actively query for to work. The examples I know of are Minecraft and TeamSpeak3.

As an example we want to setup a Minecraft server at `example.com`.
We are renting a server at some random provider and we got the IP-address `257.34.25.9` and some port `25583` where our server is running on.

We quickly setup `minecraft.example.com` to point to `257.34.25.9` which immediately works connecting to `minecraft.example.com:25583` in the game client.
We would like to ditch that unpleasant port number at the end so we setup an SRV record `_minecraft._tcp.minecraft.example.com.` pointing to `minecraft.example.com`.

We immediately fire up the game client to discover that the server displayed on the server list is not our server but someone else's.
After one (or some more) presses on refresh suddenly our server appears.

As I said in the beginning, SRV records are not processed the same as other records like A or CNAME.
When we enter `minecraft.example.com` the game tries to resolve `minecraft.example.com` which is successful since we pointed it to the hoster.
It immediately connects to it and shows some other server because our server runs at an arbitrary port but that one is running on the default port.

In the meantime while resolving the normal record the game client also sends the request for a SRV record which gets answered but after the normal request.
So when we press refresh we suddenly see our server because the game now knows the right port number we got from the SRV request while we were puzzled about seeing the wrong server.

To circumvent this problem we need to give the A (or CNAME) record some arbitrary name that we will not use.
Lets name it mc.example.com. We also modify the SRV record to `_minecraft._tcp.minecraft.example.com.` pointing to `mc.example.com.`.
This way the request of the game client for `minecraft.example.com` will not return any server so it will not try to connect and display someone else's server.
When the SRV answer arrives a split second later, Minecraft now knows the port number and resolves the name that is returned.
With both the IP-address and the port number it can now directly connect to the right server without any trouble.

To sum up, let's make an overview over the records we used to make it work:
```
_minecraft._tcp.minecraft.example.com. SRV 25583 mc.example.com.
mc.example.com. A 257.34.25.9
```
or with placeholder:
```
_minecraft._tcp.<subdomain>.<domain> SRV <port> <something>.<domain>
<something>.<domain> A <ip>
```
