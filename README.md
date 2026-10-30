# DNS: A Records, Local DNS Cache & CNAME Records

**Hands-on Windows Server DNS troubleshooting | Azure-hosted Active Directory lab**

## Overview

Configured and tested DNS name resolution between an Azure-hosted Windows Server domain controller (**DC-1**) and a Windows client (**Client-1**) in the `mydomain.com` domain. Created and modified a DNS **A record**, examined Windows **Hosts-file** and **DNS-cache** behavior, cleared stale results, and configured a **CNAME** alias. Verified changes with PowerShell/Command Prompt and Windows DNS Manager.

## Lab environment

| Component | Configuration |
| --- | --- |
| Domain controller / DNS server | DC-1 — `10.0.0.4` |
| Domain-joined workstation | Client-1 — `10.0.0.5` |
| Active Directory DNS zone | `mydomain.com` |
| Tools | DNS Manager, `ping`, `nslookup`, `ipconfig`, Notepad, Windows Hosts file |
| A-record test | `mainframe.mydomain.com`: `10.0.0.4` → `8.8.8.8` |
| CNAME test | `search.mydomain.com` → `www.google.com` |

## What I demonstrated

- Diagnosed failed hostname resolution using `ping`, `nslookup`, and cached DNS records.
- Inspected and edited the Windows Hosts file, then tested local hostname mapping.
- Created an A record in the domain DNS zone and verified client resolution.
- Observed stale results after changing an A record; refreshed the DNS resolver cache and validated the new address.
- Created a CNAME alias and verified the target using `ping` and `nslookup`.

## Commands used

```powershell
ping mainframe
nslookup mainframe
ipconfig /displaydns
ipconfig /displaydns > dns.txt
ipconfig /flushdns
ping search
nslookup search
```

> **Evidence note:** The screenshots below document both the original failed lookups and the successful configuration and troubleshooting results. The Hosts-file exercise is included as supporting context for local name resolution. External/public target IPs are lab test values. Changing an A record does not immediately invalidate a client's cached answer; `ipconfig /flushdns` clears the client resolver cache, while later lookups repopulate it.

## Screenshot walkthrough

All **31 screenshots are displayed directly** below, with no expandable or collapsed sections. They follow the practical sequence of the lab.

### 1. Environment and baseline DNS troubleshooting

**01. DC-1 session opened** — Opened the domain controller session used to administer the DNS zone.

![01 — DC-1 session opened](screenshots/01-domain-controller-session.jpg)

**02. Client-1 session opened** — Opened the domain-joined Windows client for DNS troubleshooting.

![02 — Client-1 session opened](screenshots/02-client-session.jpg)

**03. Baseline ping fails** — `ping mainframe` could not resolve the hostname before a DNS record existed.

![03 — Baseline ping fails](screenshots/03-failed-ping-mainframe.jpg)

**04. Inspect existing DNS cache** — Displayed cached DNS entries while diagnosing the missing `mainframe` record.

![04 — Inspect existing DNS cache](screenshots/04-dns-cache-before-record.jpg)

**05. Baseline nslookup fails** — `nslookup mainframe` returned a non-existent-domain result.

![05 — Baseline nslookup fails](screenshots/05-failed-nslookup-mainframe.jpg)


### 2. DNS cache and local Hosts-file troubleshooting

**06. Review name-resolution sources** — Reviewed the relationship between cached DNS results, local Hosts entries, and DNS-server lookups.

![06 — Review name-resolution sources](screenshots/06-dns-resolution-notes.jpg)

**07. Export cached DNS records** — Saved `ipconfig /displaydns` output to a text file for review.

![07 — Export cached DNS records](screenshots/07-export-dns-cache.jpg)

**08. Document troubleshooting approach** — Compared cached DNS, Hosts-file entries, and DNS-server lookups during troubleshooting.

![08 — Document troubleshooting approach](screenshots/08-name-resolution-notes.jpg)

**09. Elevate Notepad for Hosts editing** — Opened Notepad with administrator permissions to allow editing of the protected Hosts file.

![09 — Elevate Notepad for Hosts editing](screenshots/09-elevate-notepad.jpg)

**10. Locate the Hosts file** — Navigated to `C:\Windows\System32\drivers\etc\hosts`.

![10 — Locate the Hosts file](screenshots/10-open-hosts-file.jpg)

**11. Test hostname before local override** — The `zebra` hostname did not resolve before the local mapping was added.

![11 — Test hostname before local override](screenshots/11-hosts-test-before.jpg)

**12. Validate Hosts-file mapping** — Mapped `zebra` to `127.0.0.1` in the Hosts file and confirmed successful ping replies.

![12 — Validate Hosts-file mapping](screenshots/12-hosts-test-after.jpg)

**13. Compare local and DNS lookup results** — Verified `zebra` resolution while `mainframe` remained unresolved by DNS.

![13 — Compare local and DNS lookup results](screenshots/13-hosts-and-dns-test.jpg)


### 3. Create and verify a DNS A record

**14. Launch DNS Manager** — Opened DNS Manager on DC-1 with administrator privileges.

![14 — Launch DNS Manager](screenshots/14-launch-dns-manager.jpg)

**15. Inspect the forward lookup zone** — Reviewed the `mydomain.com` zone and existing host records.

![15 — Inspect the forward lookup zone](screenshots/15-inspect-dns-zone.jpg)

**16. Verify the DNS target IP address** — Confirmed DC-1 used private IP `10.0.0.4` before creating the new A record.

![16 — Verify the DNS target IP address](screenshots/16-verify-dc-private-ip.jpg)

**17. Start A-record creation** — Selected **New Host (A or AAAA)** from the forward lookup zone.

![17 — Start A-record creation](screenshots/17-open-new-host-menu.jpg)

**18. Create the mainframe A record** — Entered `mainframe.mydomain.com` with address `10.0.0.4`.

![18 — Create the mainframe A record](screenshots/18-create-mainframe-record.jpg)

**19. Confirm the A record** — Verified `mainframe` appeared as a Host (A) record pointing to `10.0.0.4`.

![19 — Confirm the A record](screenshots/19-a-record-created.jpg)

**20. Verify A-record resolution** — `ping mainframe` resolved to `mainframe.mydomain.com` (`10.0.0.4`) and received four replies.

![20 — Verify A-record resolution](screenshots/20-ping-mainframe-success.jpg)


### 4. Modify an A record and observe stale cache

**21. Modify the A-record address** — Changed the A record target from `10.0.0.4` to `8.8.8.8` in DNS Manager.

![21 — Modify the A-record address](screenshots/21-modify-a-record.jpg)

**22. Confirm new A-record target** — Confirmed DNS Manager showed `mainframe` pointing to `8.8.8.8`.

![22 — Confirm new A-record target](screenshots/22-a-record-new-address.jpg)

**23. Observe stale cached address** — The client still resolved `mainframe` to the earlier `10.0.0.4` address.

![23 — Observe stale cached address](screenshots/23-cached-old-address.jpg)

**24. Inspect cached mainframe entry** — The `ipconfig /displaydns` export showed the old `mainframe` A record cached as `10.0.0.4`.

![24 — Inspect cached mainframe entry](screenshots/24-inspect-stale-cache.jpg)


### 5. Refresh the client DNS cache

**25. Clear the DNS resolver cache** — Used `ipconfig /flushdns` and re-examined the client’s DNS cache.

![25 — Clear the DNS resolver cache](screenshots/25-flush-client-dns-cache.jpg)

**26. Verify the refreshed DNS result** — After flushing the cache, `ping mainframe` resolved to the updated address `8.8.8.8`.

![26 — Verify the refreshed DNS result](screenshots/26-resolve-updated-mainframe.jpg)


### 6. Create and test a CNAME alias

**27. Start CNAME creation** — Selected **New Alias (CNAME)** in the DNS zone.

![27 — Start CNAME creation](screenshots/27-new-cname-menu.jpg)

**28. Configure the search alias** — Entered `search.mydomain.com` as an alias for `www.google.com`.

![28 — Configure the search alias](screenshots/28-create-search-alias.jpg)

**29. Confirm the CNAME record** — Verified the `search` CNAME record pointed to `www.google.com`.

![29 — Confirm the CNAME record](screenshots/29-confirm-cname.jpg)

**30. Test resolution through the alias** — `ping search` resolved via the CNAME to a Google address and received four replies.

![30 — Test resolution through the alias](screenshots/30-ping-cname.jpg)

**31. Verify the CNAME using nslookup** — `nslookup search` returned the Google target and identified `search.mydomain.com` as the alias.

![31 — Verify the CNAME using nslookup](screenshots/31-nslookup-cname.jpg)

## Results

- **A record:** `mainframe` resolved to DC-1 (`10.0.0.4`) after creation; after the DNS record was changed and the client DNS cache cleared, it resolved to `8.8.8.8`.
- **Hosts file:** A test hostname (`zebra`) resolved locally to `127.0.0.1` after the local Hosts-file entry was added.
- **CNAME:** `search.mydomain.com` resolved through `www.google.com`; both `ping` and `nslookup` provided verification.

**Skills:** Windows Server DNS administration · Active Directory infrastructure · DNS troubleshooting · Local DNS cache management · A and CNAME records · Windows Hosts file · PowerShell
