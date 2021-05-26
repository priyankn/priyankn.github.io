---
layout: post
title: Security Blog Post
subtitle: Authentication Bypass in Mobile APIs
cover-img: /assets/img/path1.jpg
tags: [blog]
comments: true
share-title: Technical Analysis of Greyhound mobile APIs Critical Vulnerabilities
share-description: Greyhound Critical Vulnerabilities, mobile APIs
readtime: true
---

## Product Description

Greyhound Lines Inc. (U.S.) provides intercity bus service across North America. It provides a personal ticketing kiosk services via its web and mobile applications for Greyhound customers. These applications help customers manage e-ticketing, mobile check-in, and the program.

## Vulnerability Details

Critical vulnerabilities were identified in the Greyhound APIs primarily due to insufficient authentication controls. Exploitation of these can result in the exposure of personally identifiable information (PII) for the customers who had joined the Road Rewards program. Additionally, an attacker can also remotely exploit an internet-exposed web service that hosts account information for Greyhound customers as well as other sensitive information. An attacker could use this vulnerability to gain access unrestricted access and completely take over user accounts belonging to affected members.

As the time of this disclosure, this vulnerability affects at least a million members.

### Impact

The impact of these discovered vulnerabilities is as follows. When exploited, attackers can perform the following:

* Hijack (take over the accounts) of more than a million people’s Greyhound Road Rewards accounts
* Use that information to access and update users’ personal information, such as personal addresses and contact information like email addresses and phone numbers
* View and update upcoming trips as well as users’ travel preferences (e.g., if a user frequently travels alone)


## Disclosure Timeline
* 12/08/2018: Initial discovery
* Vendor contacted multiple times, no response
* 01/23/2019: Issue reported to US-CERT 
* 02/19/2019: FirstGroup Plc made contact via Greyhound support
* 03/01/2019: Full vulnerability details disclosed to vendor
* No vendor follow-up and is no longer responsive (The vulnerabilities are still not fixed)
* 04/11/2019: Partial details for the vulnerabilities disclosed publicly

## Researcher

Priyank Nigam, Security Researcher (Bishop Fox)
