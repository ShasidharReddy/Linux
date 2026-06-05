# Mail Servers

## 8.1 Overview

Mail infrastructure often includes:

- SMTP server for sending/receiving mail
- IMAP/POP3 server for mailbox access
- DNS records for identity and routing
- Anti-spam and anti-malware controls

Common Linux components:

- Postfix
- Dovecot
- OpenDKIM
- SpamAssassin
- Rspamd

## 8.2 Protocol Summary

| Protocol | Purpose | Default Ports |
|---|---|---|
| SMTP | Mail transfer/submission | 25, 587, 465 |
| IMAP | Mailbox access | 143, 993 |
| POP3 | Mailbox access | 110, 995 |

## 8.3 Postfix Overview

Postfix is a popular Mail Transfer Agent (MTA).

Roles:

- Receive inbound mail
- Relay outbound mail
- Accept authenticated submission

## 8.4 Dovecot Overview

Dovecot commonly provides:

- IMAP server
- POP3 server
- Local mail delivery in some designs
- Authentication services

## 8.5 Basic Installation Example

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y postfix dovecot-imapd dovecot-pop3d
sudo systemctl enable --now postfix dovecot
```

## 8.6 Postfix Main Files

| Purpose | Path |
|---|---|
| Main config | `/etc/postfix/main.cf` |
| Master services | `/etc/postfix/master.cf` |
| Mail queue | `/var/spool/postfix/` |
| Logs | `/var/log/mail.log` or journal |

## 8.7 Dovecot Main Files

| Purpose | Path |
|---|---|
| Main config dir | `/etc/dovecot/` |
| Protocol config | `/etc/dovecot/conf.d/` |
| Logs | journal or mail log |

## 8.8 Minimal Postfix Concepts

Important settings:

- `myhostname`
- `mydomain`
- `myorigin`
- `inet_interfaces`
- `mydestination`
- `relayhost`
- `smtpd_tls_cert_file`
- `smtpd_tls_key_file`

Example:

```conf
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example.com/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem
smtpd_use_tls = yes
smtpd_tls_security_level = may
smtp_tls_security_level = may
```

## 8.9 Submission Service on Port 587

Enable authenticated submission in `master.cf`.

Conceptual example:

```conf
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
```

## 8.10 Dovecot Maildir Example

```conf
mail_location = maildir:~/Maildir
protocols = imap pop3
```

## 8.11 TLS for Mail

Use TLS for:

- SMTP submission
- IMAP/POP3 secure ports
- Server-to-server mail when supported

## 8.12 SPF

SPF helps receiving servers verify which hosts may send mail for a domain.

Example TXT record:

```dns
example.com. IN TXT "v=spf1 mx ip4:203.0.113.10 include:_spf.example.net -all"
```

## 8.13 DKIM

DKIM signs outgoing mail with a private key.

Receiving servers validate with a public DNS TXT record.

Example selector record concept:

```dns
mail2025._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkq..."
```

## 8.14 DMARC

DMARC builds on SPF and DKIM alignment and provides reporting.

Example:

```dns
_dmarc.example.com. IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; adkim=s; aspf=s"
```

Policy options:

- `none`
- `quarantine`
- `reject`

## 8.15 Reverse DNS for Mail

Public mail servers should have proper PTR records.

Common expectation:

- IP resolves to mail hostname
- Hostname resolves back to same IP

## 8.16 Common Mail Security Practices

- Use authenticated submission on 587
- Avoid open relay configurations
- Publish SPF, DKIM, DMARC
- Use TLS certificates for mail hostnames
- Monitor blacklists and reputation
- Protect webmail and admin interfaces

## 8.17 Anti-Spam and Anti-Abuse Notes

Common controls:

- RBL checks
- Rate limiting
- Greylisting
- Spam scoring
- Malware scanning
- DMARC policy enforcement

## 8.18 Mail Troubleshooting Commands

```bash
sudo postfix check
sudo postqueue -p
sudo systemctl status postfix dovecot
sudo tail -f /var/log/mail.log
openssl s_client -connect mail.example.com:587 -starttls smtp
openssl s_client -connect mail.example.com:993
```

## 8.19 Basic Mail Server Setup Summary

1. Set hostname and DNS.
2. Install Postfix and Dovecot.
3. Configure local delivery and authentication.
4. Enable TLS.
5. Publish MX, SPF, DKIM, DMARC, PTR.
6. Test sending and receiving.
7. Monitor logs and reputation.

---

### 8.7 Mail Checklist
- SMTP, IMAP submission ports planned
- TLS cert for mail hostname installed
- SPF published
- DKIM configured
- DMARC policy published
- PTR configured
- Open relay test performed

### 8.7 Mail Troubleshooting
```bash
postqueue -p
postfix check
openssl s_client -connect mail.example.com:587 -starttls smtp
```

### 8.5 Scenario: Basic Mail Host DNS Records
```dns
example.com.            IN MX 10 mail.example.com.
mail.example.com.       IN A 203.0.113.30
example.com.            IN TXT "v=spf1 mx -all"
_dmarc.example.com.     IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
mail2025._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIB..."
```

### 8.8 Mail Reinforcement
- Mail is more than just SMTP service.
- DNS identity records matter heavily for deliverability.
- Open relays are disastrous.
- Reverse DNS matters.
- DKIM and DMARC help domain reputation.
- TLS for submission is expected.

### 8.7 Mail Commands
```bash
postfix status
postqueue -p
doveadm who
```

### 8.5 Validate Mail
```bash
openssl s_client -connect mail.example.com:587 -starttls smtp
```
