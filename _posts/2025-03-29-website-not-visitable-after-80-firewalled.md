---
layout: post
---

I disable all 80/443 traffic from specific region by *pf* firewall, it works well except one, the Let's Encrypt certificate renew needs port 80 open, otherwise it will stop any visitors, any solutions?

The simplest one is to stop *pf* to finish certificate renew then restart it.

Another solution is to use [the DNS-01 challenge](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge) instead, but it requires that your DNS provider has an API, this makes everything complex.

## Make Caddy use Let's Encrypt as default CA

As my Let's Encrypt CA expired some days ago, so it moves to ZeroSSL automaticly, this has a problem when open the website it will alert you and need you click a button to trust the CA, so it's better to use Let's Encrypt again.

To direct Caddy use Let's Encrypt only, [The Caddy Documents](https://caddyserver.com/docs/caddyfile/options#acme-ca) has explained everything, just put the following lines *on the top* of your Caddyfile:

```
{
	acme_ca https://acme-staging-v02.api.letsencrypt.org/directory
}
```

More details, read [How to Renew Let's Encrypt Certificates Behind a Firewall](https://dodov.dev/blog/how-to-renew-lets-encrypt-certificates-behind-a-firewall).
