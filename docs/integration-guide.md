---
layout: page
title: Integration Guide
permalink: /docs/integration-guide/
top_graphic: 1
---

[<- Back to Documentation List](/docs/)

This document contains helpful advice if you are a hosting provider or large website integrating Let's Encrypt, or you are writing client software for Let's Encrypt.

# Plan for Change

Both Let's Encrypt and the Web PKI will continue to evolve over time.  You should make sure you have the ability to easily update all services that use Let's Encrypt. If you're also deploying clients that rely on Let's Encrypt certificates, make especially sure that those clients receive regular updates.

In the future, we're likely to change: the root and intermediate certificates from which we issue; the hash algorithms we use when signing certificates; the types of keys and key strength checks for which we are willing to sign end-entity certificates; and the ACME protocol. We will always aim to give as much advance notice as possible for such changes, though if a serious security flaw is found in some component we may need to make changes on a very short term or immediately. For intermediate changes in particular, you should not hardcode the intermediate to use, but should use the [`Link: rel="up"`](https://github.com/ietf-wg-acme/acme/blob/master/draft-ietf-acme-acme.md#certificate-issuance) header from the ACME protocol, since intermediates are likely to change.

You will also want a way to keep your TLS configuration up-to-date as new attacks are found on cipher suites or protocol versions.

We will be providing a low-volume developers' mailing list to receive news of important changes like the above (watch this space).

# Who is the Subscriber

Our [CPS and Subscriber Agreement](/repository/) indicate that the Subscriber is whoever holds the private key for a certificate. For hosting providers, that's the provider, not the provider's customer. If you're writing software that people deploy themselves, that's whoever is deploying the software.

The contact email provided when creating accounts (aka registrations) should go to the Subscriber. We'll send email to that address to warn of expiring certs, and notify about changes to our [privacy policy](/privacy).  If you're a hosting provider, those notifications should go to you rather than a customer. Ideally, set up a mailing list or alias so that multiple people can respond to notifications, in case you are on vacation.

The upshot of this is that, if you are a hosting provider, you do not need to send us your customers' email addresses or get them to agree to our Subscriber Agreement. You can simply issue certificates for the domains you control and start using them.

# One Account or Many?

In ACME, it's possible to create one account and use it for all authorizations and issuances, or create one account per customer. This flexibility may be valuable. For instance, some hosting providers may want to use one account per customer, and store the account keys in different contexts, so that an account key compromise doesn't allow issuance for all of their customers.

However, for most larger hosting providers we recommend using a single account and guarding the corresponding account key well. This makes it easier to identify certificates belonging to the same entity, easier to keep contact information up-to-date, and easier to provide rate limits adjustments if needed. We will be unable to effectively adjust rate limits if many different accounts are used.

# Storing and Reusing Certificates and Keys

A big part of Let's Encrypt's value is that it enables automatic issuance as part of provisioning a new website.  However, if you have infrastructure that may repeatedly create new frontends for the same website, those frontends should first try to use a certificate and private key from durable storage, and only issue a new one if no certificate is available, or all existing certificates are expired.

For Let's Encrypt, this helps us provide services efficiently to as many people as possible. For you, this ensures that you are able to deploy your website whenever you need to, regardless of the state of Let's Encrypt.

As an example, many sites are starting to use Docker to provision new frontend instances as needed. If you set up your Docker containers to issue when they start up, and you don't store your certificates and keys durably, you are likely to hit rate limits if you bring up too many instances at once. In the worst case, if you have to destroy and re-create all of your instances at once, you may wind up in a situation where none of your instances is able to get a certificate, and your site is broken for several days until the rate limit expires. This type of problem isn't unique to rate limits, though. If Let's Encrypt is unavailable for any reason when you need to bring up your frontends, you would have the same problem.

Note that some deployment philosophies state that crypto keys should never leave the physical machine on which they were generated. This model can work fine with Let's Encrypt, so long as you make sure that the machines and their data are long-lived, and you manage rate limits carefully.

# Picking a Challenge Type

If you're using the http-01 ACME challenge, you will need to provision the challenge response to each of your frontends before notifying Let's Encrypt that you're ready to fulfill the challenge. If you have a large number of frontends, this may be challenging. In that case, using the dns-01 challenge is likely to be easier. Of course, if you have many geographically distributed DNS responders, you have to make sure the TXT record is available on each responder.

Additionally, when using the dns-01 challenge, make sure to clean up old TXT records so the response to Let's Encrypt's query doesn't get too big.

If you want to use the http-01 challenge anyhow, you may want to take advantage of HTTP redirects. You can set up each of your frontends to redirect /.well-known/acme-validation/XYZ to validation-server.example.com/XYZ for all XYZ. This delegates responsibility for issuance to validation-server, so you should protect that server well.

# Central Validation Servers

Related to the above two points, it may make sense, if you have a lot of frontends, to use a smaller subset of servers to manage issuance. This makes it easier to use redirects for http-01 validation, and provides a place to store certificates and keys durably.

# Implement OCSP Stapling

Many browsers will fetch OCSP from Let's Encrypt when they load your site. This is a [performance and privacy problem](https://blog.cloudflare.com/ocsp-stapling-how-cloudflare-just-made-ssl-30/).  Ideally, connections to your site should not wait for a secondary connection to Let's Encrypt. Also, OCSP requests tell Let's Encrypt which sites people are visiting. We have a good privacy policy and do not record individually identifying details from OCSP requests, we'd rather not even receive the data in the first place. Additionally, we anticipate our bandwidth costs for serving OCSP every time a browser visits a Let's Encrypt site for the first time will be a big part of our infrastructure expense.

By turning on OCSP Stapling, you can improve the performance of your web site, provide better privacy protections for your users, and help Let's Encrypt efficiently serve as many people as possible.

Additionally, OCSP Stapling forms the basis for two other technologies you may want to use in the future: Must Staple, which provides for revocation that actually works, and Certificate Transparency proofs (which we will be delivering by OCSP, and which must be stapled for Chrome to recognize them).

# Let's Encrypt IPs

Let's Encrypt will validate from a number of different IP addresses in the future, and will not announce which ones in advance. You should make sure your validation server is available to all IPs.

Some people who are issuing for non-HTTP services want to avoid exposing port 80 to anyone except Let's Encrypt's validation server. If you're in that category you may want to use the DNS challenge instead.
