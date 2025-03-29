---
layout: post
---

I disable all 80/443 traffic from specific region by *pf* firewall, it works well except one, the Let's Encrypt certificate renew needs port 80 open, otherwise it will stop any visitors, any solutions?

The simplest one is to stop *pf* to finish certificate renew then restart it.

Another solution is to use [the DNS-01 challenge](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge) instead, but it requires that your DNS provider has an API, this makes everything complex.

More details, read [How to Renew Let's Encrypt Certificates Behind a Firewall](https://dodov.dev/blog/how-to-renew-lets-encrypt-certificates-behind-a-firewall).
