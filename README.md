# VPS with storage: how to pick a big-disk server for backups, Plex, and file archives without overpaying for terabytes

Search for a "vps with storage" and you'll notice something quickly: most VPS plans are built for CPU and RAM, with disk space treated as an afterthought. A typical budget VPS gives you 20–80 GB of fast NVMe and charges painful rates for anything beyond that. If you're trying to host a Plex library, run Nextcloud for a family, or point nightly backups at a remote machine, that's exactly the wrong shape of server.

So this guide does three things: helps you figure out how much storage you actually need, shows you what separates a genuinely good storage VPS from a bad one (spoiler: it's usually the bandwidth fine print, not the disk), and then walks through one provider's lineup in detail — Sharktech, whose portfolio happens to cover all three storage flavors: NVMe VPS, HDD-backed cloud servers, and flat-rate S3 object storage. If you want to skip ahead, 👉 the full plan comparison table is further down.

## Two very different products hide behind this search

"VPS with storage" means one of two things, and mixing them up is the most expensive mistake in this category.

A **storage VPS** trades speed for capacity. It runs on spinning HDDs (usually in RAID), gives you anywhere from 1 TB to 16+ TB, costs roughly $10–15 per TB, and doesn't care much about IOPS. It's for data that mostly sits there: media libraries, backup archives, ISO collections, photos you rarely open.

A **regular VPS** is the opposite: small NVMe or SSD disk, high IOPS, built for databases, websites, and applications. Per terabyte, it's several times more expensive — but the disk is fast enough to run MySQL or serve video without stuttering.

If you buy a regular VPS and try to stuff 4 TB of movies onto it, you'll overpay badly. If you buy a storage VPS and try to run a database on it, you'll wonder why everything is slow. The self-hosting communities keep running into this — one popular Reddit thread in r/selfhosted is literally someone hunting for a cheap VPS with lots of HDD space for a manga collection, because every host they found was overpriced for bulk storage. That complaint is the whole niche in one sentence.

The honest first question, then, isn't "which provider" — it's "how much of my data needs to be fast, and how much just needs to exist?"

## Work out your actual number before you shop

Rough sizing, based on what these workloads really consume:

- **OS + Docker containers + a few apps:** 20–60 GB. Almost nothing, by modern standards.
- **A personal Nextcloud with photos and documents:** 200 GB–1 TB, depending on how many phone backups you keep.
- **A media library:** 1 TB per ~200 movies in decent quality, or per ~50 Blu-ray remuxes. TV shows scale faster. This is where multi-terabyte requirements come from.
- **Off-site backups:** the size of everything you back up, times your retention count. A 500 GB server backed up nightly for 14 days of retention needs ~7 TB if you keep full copies — though deduplicating tools like Borg or restic shrink that a lot.

A useful pattern once your needs exceed about 2 TB: split the workload. Keep a small, fast NVMe VPS for the applications, and put the bulk data on either a large HDD-backed server or S3 object storage. That split is cheaper and more resilient than one big server doing everything.

## The line item that quietly ruins cheap storage deals: egress

Disk price is the headline; **bandwidth is the trap**. A storage server is useless if moving data in and out costs more than the disk itself. Some hosts cap throughput on "storage" plans, meter egress aggressively, or bill overage at rates that turn a restore into a budget emergency.

What to check on any offer:

1. **Included outgoing transfer per month**, and the per-GB overage rate. Backups are write-heavy (incoming), but restores and media streaming are outgoing — that's the direction that costs money.
2. **Port speed.** A 1 Gbps port moves roughly 320 TB per month at full utilization. A 100 Mbps port manages ~32 TB. For a Plex box with multiple streams, 100 Mbps is a real bottleneck.
3. **Whether "unlimited" means unlimited.** It usually means unlimited *incoming*, with outgoing metered. That's fine — as long as you know the number.

Sharktech's approach here is unusually clear, because the terms are published rather than hidden: cloud services include **unlimited incoming bandwidth and 5,000 GB outgoing per month, with additional outgoing billed at $0.002 per GB**. That overage rate works out to $2 per TB — low enough that a monthly 10 TB restore won't require a meeting with your accountant. The Smart VPS line includes 4 TB of transfer in its entry configuration and scales up to around 300 TB on higher resource tiers. And every plan runs on a 1 Gbps port.

## NVMe vs SSD vs HDD: what the speeds actually are

Sharktech publishes per-volume performance estimates for its cloud storage tiers, which saves some guesswork:

| Storage type | Estimated throughput | Estimated IOPS |
| --- | --- | --- |
| NVMe | 1.2 GB/s | 18,000 |
| SSD | 350 MB/s | 6,000 |
| HDD | 120 MB/s | 3,000 |

Translate that into decisions: NVMe for databases and anything with random I/O; SSD for general-purpose VMs; HDD for data that's written once and read occasionally. An archive of family photos doesn't care about 18,000 IOPS. Your PostgreSQL primary very much does.

On the VPS side, Smart VPS runs on enterprise NVMe across the board. Independent testing by HostAdvice — which Sharktech cites on its own product page — measured over 6,000 random IOPS on 4K blocks, sub-millisecond network latency (0.547 ms to Google DNS), and roughly 19 GB/s memory throughput. Those aren't numbers you associate with a $4/month server, and they're the reason an NVMe VPS remains the right tool even when terabytes aren't the priority.

## Where Sharktech fits into this picture

Quick context, because it explains the pricing structure: Sharktech has operated since 2003, runs its own ISP (AS46844, peering at major internet exchange points), and has data centers in Los Angeles, Las Vegas, Denver, Chicago, and Amsterdam. The network was built around DDoS mitigation from the start — every Smart VPS includes 60 Gbps of DDoS protection, not as a paid add-on — and the platform carries a 99.999% uptime target. You can 👉 check the current Smart VPS lineup and configure a plan directly if you want to see the live numbers.

### Smart VPS: fast storage, up to 2 TB, sold as a resource pool

Smart VPS is less a "plan" and more a bucket of resources you carve up yourself. You buy a pool of vCPU, RAM, NVMe storage, and bandwidth, then deploy as many VMs as the pool supports — one big instance, or a dozen small ones spread across different cities, with no arbitrary VM count limit. Upgrades and downgrades happen without redeploying.

The current configurator offers resource tiers from XS up to 3XL:

- 2 to 128 vCPUs (Xeon Gold cores)
- 4 to 256 GiB DDR4 RAM
- 40 GiB to 2,000 GiB NVMe storage, plus a separate **backup storage** line, also up to 2,000 GiB — effectively 4 TB of total capacity on the top tier
- 4 TB of included transfer, configurable up to roughly 300 TB
- 1 IPv4 address included; Linux distributions include Ubuntu, Debian, AlmaLinux and others, and Windows Server can be installed via ISO with your own license

Pricing starts at **$7.95/month** on a monthly cycle for the entry XS pool (2 vCPU, 4 GiB RAM, 40 GiB NVMe, 4 TB transfer). The billing-cycle discounts are automatic and steep: 25% off quarterly, 35% off semi-annually, and 50% off annually — which puts the entry pool at **$3.98/month**. No coupon hunting required; you just pick the cycle at checkout. 👉 See live Smart VPS pricing and deployment options here.

That backup storage line is worth pausing on. Plenty of providers sell you disk and leave you to figure out offsite copies. Here it's a checkbox on the same order form — a sensible spot for restic/Borg repositories if the total data set is under a couple of terabytes.

### Public Cloud: the actual "lots of storage" option

When people say they want a VPS with storage, what they often need is Sharktech's Public Cloud tier — OpenStack-based VMs with **multi-tier storage where HDD is a first-class choice**. This is the product line where you can build a 12 TB media server or a backup target without NVMe pricing.

Four tiers are currently displayed, each a range you configure within:

- **Small** — 4–16 vCPU, 8–32 GB RAM, up to 2.4 TB SSD / 4.8 TB HDD / 1.2 TB NVMe, from **$39/mo**
- **Medium** — 8–32 vCPU, 16–64 GB RAM, up to 6.4 TB SSD / 12.8 TB HDD / 3.2 TB NVMe, from **$79/mo**
- **Large** — 32–128 vCPU, 64–256 GB RAM, up to 12 TB SSD / **24 TB HDD** / 6 TB NVMe, from **$249/mo**
- **Enterprise** — 64+ vCPU, 128 GB+ RAM, SSD starting at 5 TB and scaling from there, from **$499/mo**

Beyond the included commit, resources bill hourly: CPU at $0.0025 per core-hour, RAM at $0.0035 per GB-hour, and storage at $0.00002 per GB-hour for HDD, $0.00006 for SSD, and $0.00009 for NVMe. Run the math on the HDD rate — about 730 hours in a month — and on-demand HDD storage works out to roughly **$14.60 per TB per month**; NVMe lands near $65/TB. The gap is the whole storage-VPS story in three decimal places.

Each plan carries a maximum resource cap (outside Enterprise), so a runaway process can't generate a hyperscaler-style surprise bill. The first public IPv4 is free, additional ones cost $1.50/month. There's no vendor lock-in: you can download your disk images whenever you like, which matters more than people think until the day it suddenly matters a lot. 👉 Configure a Public Cloud VM with the storage mix you need.

### S3 Object Storage: flat $4.90/TB for pure archives

For data that doesn't need a server attached at all, Sharktech sells S3-compatible object storage at a flat **$4.90 per TB per month** — no minimum commitment, full S3 API compatibility, redundant clusters in their own data centers. The S3 order page illustrates the math plainly: 1 TB of storage plus 1 TB of bandwidth comes to $4.90/month, and storage plus bandwidth are the only two items on the invoice.

For comparison, Amazon S3's standard tier in US East runs about $0.023 per GB — roughly $23.56 per TB before you've paid a cent of egress or request fees. At archive scale, Sharktech's flat rate is less than a quarter of that. If your "VPS with storage" need is really "a big dumb bucket that never loses my backups," S3 is frequently the correct answer, and it's the one most storage-VPS shoppers overlook. 👏 Check current S3 object storage rates here. — that link also lets you 👉 compare the full Sharktech storage lineup in one place.

## Full plan comparison table

Every storage-relevant plan currently displayed across Sharktech's store, with the configuration ranges and prices as published:

| Plan | Storage (max config) | Compute | Bandwidth | Price (monthly billing) | Order |
| --- | --- | --- | --- | --- | --- |
| **Smart VPS** (XS–3XL resource pool) | 40 GB–2 TB NVMe + up to 2 TB backup storage | 2–128 vCPU, 4–256 GB DDR4 | 4–300 TB, 1 Gbps port | From **$7.95/mo** ($3.98/mo on annual, 50% off) | [Deploy Smart VPS](https://bit.ly/SharKTech) |
| **Public Cloud Small** | Up to 2.4 TB SSD / 4.8 TB HDD / 1.2 TB NVMe | 4–16 vCPU, 8–32 GB RAM | 20 TB+ (unlimited in, 5 TB out, $0.002/GB after) | From **$39/mo** | [Order Public Cloud Small](https://bit.ly/SharKTech) |
| **Public Cloud Medium** | Up to 6.4 TB SSD / 12.8 TB HDD / 3.2 TB NVMe | 8–32 vCPU, 16–64 GB RAM | 20 TB+ (same egress terms) | From **$79/mo** | [Order Public Cloud Medium](https://bit.ly/SharKTech) |
| **Public Cloud Large** | Up to 12 TB SSD / **24 TB HDD** / 6 TB NVMe | 32–128 vCPU, 64–256 GB RAM | 20 TB+ (same egress terms) | From **$249/mo** | [Order Public Cloud Large](https://bit.ly/SharKTech) |
| **Public Cloud Enterprise** | 5 TB+ SSD, unbounded above | 64+ vCPU, 128 GB+ RAM | 20 TB+ (same egress terms) | From **$499/mo** | [Order Public Cloud Enterprise](https://bit.ly/SharKTech) |
| **S3 Object Storage** | Any size, $4.90/TB flat | n/a (API/storage only) | Billed alongside storage | **$4.90/TB/mo** | [Order S3 Storage](https://bit.ly/SharKTech) |

For storage beyond even the Large tier's 24 TB, Sharktech also sells dedicated bare-metal servers in all five locations; those are quoted individually rather than listed at fixed prices, so if that's your situation, the sales team is the honest answer.

## What it costs in practice

Some concrete scenarios, using only the prices above:

**A 500 GB personal backup box.** Smart VPS XS on annual billing: $3.98/month, 40 GB NVMe is too small for 500 GB — so step up within the pool or, more sensibly, point restic at S3 instead: 0.5 TB ≈ $2.45/month. S3 wins for anything that's pure backup data with no compute attached.

**A 4 TB Plex server.** Public Cloud Small with the HDD option maxed (4.8 TB) starts around $39/month; egress for a household's streaming sits well inside the 5 TB outgoing allowance. Alternatively, a Smart VPS pool with 2 TB NVMe plus 2 TB backup storage handles a smaller library with faster scrubs, from $7.95/month before cycle discounts.

**A 16 TB archive.** S3 at $4.90/TB: about **$78/month**, with no server to patch. A Large-tier Public Cloud VM with 16 TB HDD gives you the same capacity attached to serious compute (32+ vCPU) from $249/month — you'd choose that only if the data needs to be mounted by applications, not just stored.

The pattern: pure capacity is cheapest in S3, capacity-with-compute lives in Public Cloud HDD tiers, and fast-attached storage is Smart VPS territory.

## Fine print worth knowing

A few things that don't make the marketing pages:

- **It's unmanaged by default.** Sharktech's own FAQ is blunt that basic command-line familiarity is expected. Support is 24/7/365 and reachable by phone — a rarity at this price level — but it's for infrastructure problems, not a Linux tutorial service. (They do offer a separately managed Cloud Applications Platform if you'd rather not touch a terminal.)
- **Payments are non-refundable**, per their terms as noted in independent reviews. Reasonable for this industry, but it argues for starting on a monthly cycle if you're unsure, then switching to annual to grab the 50% discount once you've verified the service fits.
- **cPanel costs extra** as an add-on if your workflow depends on it. Windows Server installs from ISO and requires your own license or a purchased one.
- **The plan names have shifted.** Older reviews reference fixed plans called Tiny, Small, Medium, Large, and Colossal; the portal has moved Smart VPS to a configurable XS–3XL resource-tier model, and the legacy product pages currently show out of stock. Don't let old review pricing confuse you — the configurator is the source of truth.
- **Reputation is modest in volume but consistent.** Trustpilot shows a 3.5/5 average across a small base of 13 reviews, and HostAdvice has recognized the service for uptime and support quality based on independent testing. Small sample, but nothing in it contradicts the benchmark story.

## Which one should you actually buy

Decide by what your data does, not by what sounds impressive:

1. **Apps, websites, databases, small fast storage** → Smart VPS, annual cycle. The entry pool at $3.98/month with 60 Gbps DDoS protection included is the standout value in the lineup.
2. **Multi-terabyte media or file storage that needs a real server** → Public Cloud, HDD tier, sized to the Small/Medium/Large that matches your TB count.
3. **Backups and archives with no compute attached** → S3 at $4.90/TB. This is the one most people overpay for by buying a server they don't need.
4. **Both** → the split setup: a small Smart VPS for the moving parts, S3 for the copies. For most homelab-scale setups this combination costs less than a single oversized storage VPS from the hosts that charge NVMe rates for cold data.

👉 Start with the Smart VPS configurator if you want to see real-time pricing on any of these paths.

The short version of the whole guide: know your terabytes, read the egress terms before the disk specs, and match the storage type to how often the data actually gets touched. Do those three things and the "vps with storage" search stops being a lottery.
