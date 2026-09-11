# ORION Internal Security Assessment - Field Notes

## Assessment Information

**Date:** 2026-09-11
**Assessment Type:** Internal Black-Box Penetration Test
**Source System:** KALI01
**Source IP:** 192.168.244.40
**Scope:** 192.168.244.0/24

## Provided Information

The following network information was provided for the assessment:

- Network: 192.168.244.0/24
- Default Gateway: 192.168.244.2
- DNS Server: 192.168.244.10
- DNS Search Domain: corp.orion.test

## Rules / Assumptions

- Testing is restricted to the isolated ORION lab enviroment.
- No credentials are provided initially.
- Testing starts from an internal network connection.
- Denial-of-Service testing is out of scope.

---

## 2026-09-11

### Assessment Start

The assessment system was connected to the internal network using provided network configuration.
Basic connectivity and DNS functionality were verified.

### Initial Knowledge

At the beginning of the reconnaissance:

- The assessment network is 192.168.244.0/24.
- The configured DNS server is 192.168.244.10.
- The DNS search domain is corp.orion.test.
- The roles and purposes of individual hosts are unknown.
- No user credentials are available.

### Host Discovery

**Objective:** Identify active hosts within the assinged network.

**Command:**

```bash
nmap -sn 192.168.244.0/24
```

**Result:**

- 192.168.244.1
- 192.168.244.2
- 192.168.244.10
- 192.168.244.30
- 192.168.244.40
- 192.168.244.254

**Observation:**

- 192.168.244.40 is the assessment system itself.
- 192.168.244.10 is the DNS server.
- 192.168.244.2 identified as default gateway through local routing table
- 192.168.244.1 identified as Windows system connected via VMware through nmap
- 192.168.244.254 most likely VMware infrastructure through nmap -> no tcp service found

### Service Enumeration

**Objective:** Identify ports for relevant hosts.

**Command:**

```bash
nmap -sV 192.168.244.10 192.168.244.30
```

**Result:**

- 192.168.244.10
  - 88/tcp Kerberos - Authetication protocol of AD
  - 389/tcp Microsoft AD LDAP
  - 3268/tcp Microsoft AD LDAP Global Catalog
  - Hostname: DC01
  - Windows OS
- 192.168.244.30
  - 22/tcp OpenSSH
  - 3000/tcp returned HTTP responses
  - Linux OS

**Observation:**

- 192.168.244.10 is most likely a AD DC -> AD Serverfunctions (Kerberos, AS LDAP and Global Catlog) called DC01.
- 192.168.244.30 has a Linus Host exposing SSH service and an HTTP server.

### Web Enumeration - 192.168.244.30

**Initial observations:**

- HTTP service responds successfully.
- Application returns multiple browser security headers.
- CORS header `Access-Control-Allow-Origin: *` is present and requires further review.
- Custom header `X-Recruiting` discloses the path `/#/jobs`.

#### Manual Application Review

**Objective:** Identify visible application functionality and potential input points.

**Observed functionality:**

- Webshop for Juice and various miscellaneous items
- web01.corp.orion.test:3000/#/
  - Search bar -> input
  - Search in laguage selection -> input
  - Add to basket button, detailed view of products, links in topbar and sidemenu.
- web01.corp.orion.test:3000/#/login
  - Login page with password recovery and register links
  - Email and Password input.
- web01.corp.orion.test:3000/#/forgot-password
  - Password recovery page only allows email input -> valid mail doesn't unlock other inputs.
- web01.corp.orion.test:3000/#/register
  - Registration page allows email, password, repeat and security question input.
- web01.corp.orion.test:3000/#/basket
  - Basket overview, checkout link
- web01.corp.orion.test:3000/#/contact
  - Feedback page with comment text area, rating and CAPTCHA input.
- web01.corp.orion.test:3000/#/about
  - lorem ipsum about us text with link to terms and conditions.
- web01.corp.orion.test:3000/ftp/legal.md
  - lorem ipsum text
