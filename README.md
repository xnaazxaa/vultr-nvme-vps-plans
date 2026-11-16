# NVMe VPS Hosting on Vultr: Which Plan Fits, What the $300 Credit Actually Covers, and Where NVMe Stops Helping

Vultr's cheapest NVMe plan is $6.00/month: 1 vCPU, 1 GB RAM, 25 GB of NVMe storage, billed monthly with an hourly meter running underneath. The $300 credit offered at signup belongs to new accounts, needs a card or PayPal attached, and expires 30 days after it's issued. It is a month of free testing, not a free year of hosting.

What follows is the price structure you can verify today, the difference between the $5 plan and the $6 plan (one of them isn't NVMe), how the credit's expiry date changes what you should deploy, and how much RAM each workload actually needs before you pick a tier.

> **Start at the $6.00 High Performance AMD plan** (1 vCPU, 1 GB RAM, 25 GB NVMe) for a static site, a staging box, a VPN endpoint, or a single small container. Move up as soon as a production database is involved. 👉 [See Vultr's current NVMe plans and pricing](https://bit.ly/VuLtr)

## Vultr's NVMe price points, and the $5 plan that isn't NVMe

The $5.00 Cloud Compute configuration is listed with 25 GB of SSD, not NVMe. The $6.00 High Performance AMD entry tier is the first one that puts NVMe under the operating system. A dollar a month separates the two, which makes the cheaper plan a false economy if disk latency is the reason you searched for NVMe hosting in the first place.

| Plan | vCPU / RAM | Storage | Price and billing | Order |
| --- | --- | --- | --- | --- |
| Cloud Compute, vc2-1c-1gb | 1 vCPU / 1 GB | 25 GB SSD | $5.00/mo, $0.007/hr | [Deploy this size](https://bit.ly/VuLtr) |
| High Performance AMD, entry | 1 vCPU / 1 GB | 25 GB NVMe | $6.00/mo | [Deploy this size](https://bit.ly/VuLtr) |
| High Performance AMD, top of the published range | 12 vCPU / 24 GB | NVMe | $216.00/mo | [Deploy this size](https://bit.ly/VuLtr) |

Three things about that table matter more than the rows.

**Billing is hourly with a monthly ceiling.** Vultr caps the meter at 672 hours, which is 28 days. Run a server for a full calendar month and you pay the monthly rate, not 744 hours of hourly billing. Spin one up for a weekend and you pay for the weekend.

**The catalog is much larger than the table.** Vultr lists more than 80 configurations across shared, high performance and dedicated CPU lines, with a floor around $2.50/month for the smallest regional and IPv6-oriented sizes. Prices differ by region: the same plan in São Paulo runs roughly half again as much as it does in Frankfurt or New Jersey. Use the table above as checkpoints, then confirm the exact figure and storage type on the plan card before you order.

**Bandwidth is metered.** Every tier carries a transfer allowance rather than an unmetered port, and the allowance changes by plan line. If your workload is bandwidth-heavy rather than disk-heavy, read that number before the price.

## NVMe vs SATA on a VPS: the honest version

NVMe connects storage over PCIe instead of the SATA III interface, whose practical ceiling is around 550 MB/s. A Gen4 drive can move multiple GB/s. On a virtual server, the improvement you will actually feel shows up in random I/O and latency, not in the sequential number printed on the product page.

That distinction gets buried in this category's marketing. Hosts routinely run a 7,000 MB/s headline next to an FAQ that admits there is no single speed multiplier, because VPS storage performance depends on the host's hardware, on how much I/O your allocation gets, and on what your workload asks of it. A 25 GB virtual disk is a slice of a shared pool. It will not deliver a benchmark drive's throughput, and any article quoting you 1M+ IOPS as a VPS expectation is describing the hardware, not your server.

What NVMe genuinely helps:

- **Databases whose working set spills past RAM.** MySQL, PostgreSQL, MongoDB and Redis all stall on index and join reads that reach disk. Lower latency per read is where the gain comes from.
- **Storefronts under traffic spikes.** WooCommerce and Magento issue a lot of small concurrent reads, which is the pattern SATA handles worst.
- **Search engines.** Elasticsearch, Solr and Meilisearch write indexes and read them back constantly.
- **CI runners.** Dependency installs, compiles and artifact writes spend their time waiting on I/O.
- **Container hosts.** Pulling and unpacking image layers is a burst of small writes.
- **Log-heavy services, large imports and exports, local backups.**

What it does not help:

- **CPU-bound processes.** If your app pegs the CPU with idle disk, a faster disk buys nothing. A higher CPU tier does.
- **Workloads that fit in page cache.** The second read comes from RAM on any storage type.
- **Static file serving limited by the transfer cap.** The network is the bottleneck there, not the disk.

A practical test before you upgrade: if your monitoring shows sustained I/O wait above single digits, NVMe is the right lever. If the CPU graph is flat at 100% while the disk sits quiet, spend the money on cores.

## The $300 credit: 30 days, not a free year

The terms that decide whether the credit is worth anything:

- **New accounts only**, one promotional code per user, duplicate accounts excluded.
- **A card or PayPal must be linked.** Vultr does offer a $0.00 card-link path, so you can attach a payment method without a deposit.
- **Unused credit expires after 30 days.** Vultr's own account documentation describes free credits as lasting "up to 30 days." Users who have watched the billing page report the same: credit lands, then the unused balance vanishes roughly a month later.
- **Credits are non-refundable and payments are final.** There is no cash-out on a leftover balance.
- **Billing starts at deploy and continues while the server is stopped.** A powered-off instance still costs money. Only destroying it stops the meter.

Run the arithmetic before you plan around it. The $6.00 plan costs about $6 for a full month, so a single small instance can't consume $300 before the credit expires. To actually use it you'd need to burn around $10 of compute a day: one 12 vCPU / 24 GB instance at $216/month plus a couple of mid-size boxes, or a short-lived cluster for a load test you'd otherwise run badly.

Which means the credit suits a specific kind of buyer: someone who wants to benchmark a real deployment, test a migration, or run a short experiment at scale, and who will delete everything before day 30. If you want a cheap long-term home for a blog, the credit does nothing for you past the first month, and the standard rate applies from day 31.

Two habits make the credit safe. Open the billing page on day one and write down the expiry date. Then destroy, not stop, every server you deployed before that date. 👉 [Open a Vultr account and check the current credit terms](https://bit.ly/VuLtr)

## How much RAM each workload needs

Sizing is where most overpayment happens. The rules below assume Linux and a normal web stack.

**Static site, reverse proxy, WireGuard, small bot: 512 MB to 1 GB.** The $6.00 tier covers this with room to spare.

**One application plus a small production database: 2 GB minimum, 4 GB comfortable.** A LEMP stack with MySQL on 1 GB works until the database grows, then the OOM killer starts choosing processes for you. This is the single most common upgrade path.

**Several sites or a handful of Docker containers: 4 GB to 8 GB.** Container overhead is small; the databases and language runtimes inside them are not.

**Redis or an in-memory store: size to your dataset plus headroom**, and look at the memory-optimized line rather than stretching a general-purpose plan. RAM is the constraint, not IOPS.

**CI runner: 4 GB and up.** This is one of the few workloads where NVMe earns its price, because builds are dominated by small reads and writes.

On the "is 1 GB enough for a database" question: SQLite, a Redis cache, or a Postgres instance you're only testing against will run fine. MySQL with real customer data will not stay comfortable for long. Vultr's published range tops out at 12 vCPU / 24 GB on the tier tracked above, so a workload needing more memory than that pushes you to a different plan family entirely.

## Where Vultr sits next to other NVMe hosts

Comparison shopping on NVMe VPS hosting comes down to three variables: the entry price, whether the advertised price is a promotion, and where the servers sit. A few verified data points for context, all of them the host's own published figures.

| Host | Entry NVMe configuration | Price | Term and caveats |
| --- | --- | --- | --- |
| IONOS VPS+ | 1 vCore / 2 GB / 60 GB NVMe (VPS S+) | $2/mo for 3 months, $6/mo list | 1-year contract; promotion covers the first 3 months only |
| Arct Cloud | 1 vCPU / 1 GB / 15 GB NVMe | $3.89/mo | DDoS protection and 24/7 support included |
| AsiaWebHost | 2 vCPU / 4 GB / 60 GB NVMe | S$55.00/mo | Singapore region, monthly billing, no contract |
| DartNode | 1 vCPU / 1 GB / 30 GB NVMe | $8.95/mo | US South, 1 Gbps unmetered within fair use |

IONOS is the clearest illustration of why first-month pricing deserves suspicion. Its page shows a "$4/month" headline, a $6 list price on the smallest plan, and a $2 promotional rate that only applies for three months against a one-year commitment. The savings figures are real, but comparing that $2 to Vultr's $6 compares a three-month teaser to a standard rate.

Arct Cloud's $3.89 entry tier is a 15 GB disk and 1 GB RAM, which is thinner than Vultr's 25 GB at $6, so the two aren't the same purchase despite the price gap. AsiaWebHost prices in SGD and is worth a look if your users are in Southeast Asia and you'd rather not pay for cross-Pacific latency. DartNode sits above Vultr on entry price with more storage per dollar.

Vultr's own case rests on breadth: 32 locations listed, including Singapore, Tokyo, Osaka, Seoul, Sydney, Melbourne, Mumbai, Bangalore and Delhi NCR for Asia-Pacific readers, with the same plan priced consistently across most of them. None of the hosts above publishes a renewal price separate from its list price in the pages reviewed, so ask before you commit to a term.

## Questions people ask before ordering

**Is an NVMe VPS a fixed amount faster than SATA?** No fixed multiplier exists. SATA III caps around 550 MB/s and NVMe over PCIe runs into multiple GB/s, but those are interface ceilings. What you get on a virtual machine depends on the host, your allocation and your access pattern. Expect the biggest gains in random reads and latency.

**Will 1 GB of RAM run my database?** A test instance, yes. A production MySQL or Postgres database with customer data, no. Start at 2 GB and watch memory usage for the first month.

**What happens when the $300 credit runs out?** Billing continues at the standard rate, hourly with the 672-hour monthly cap. There's no automatic shutdown and no refund on unused credit, so destroy anything you don't intend to keep paying for.

**Can I save money by stopping a server?** No. Charges continue while an instance is stopped. Destroying it is what stops billing.

**Do I need the NVMe tier if my site is small?** If your pages are cached and your database is tiny, the $5 SSD plan performs the same in practice. The $1 difference makes the decision easy, but don't pay for a larger NVMe plan to fix a problem that's actually CPU or bandwidth.

## How to decide

Match the plan to the workload, not to the storage label:

- **Static site, staging environment, VPN, single container** → the 1 GB NVMe plan at $6.00/month.
- **One app with a growing database** → 2 GB of RAM minimum, and accept that you'll upgrade within a year.
- **Multiple sites, Docker, or a CI runner** → 4 GB and up; this is where NVMe pays for itself.
- **Redis or an in-memory database** → memory-optimized line, sized to the dataset.
- **A short experiment that needs real scale** → use the credit, deploy large, destroy by day 30.

Before you order, confirm four things: that the tier you selected actually lists NVMe rather than SSD, the transfer allowance on that plan, the price in the region you're deploying to, and the expiry date on any credit in your account. Those four numbers explain most of the bill surprises people post about.

👉 [Compare Vultr's NVMe plans and start with the size that matches your workload](https://bit.ly/VuLtr)
