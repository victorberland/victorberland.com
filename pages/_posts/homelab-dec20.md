---
title: My all-in-one homelab
date: 2020-12-22
layout: post
---
I always liked when people would show off their homelabs on Reddit for me to gather inspiration and learn from how they've set up their systems and infrastructure.

So I thought I'd follow the flow and write a bit about my own setup and the decisions I've made around it.

![](https://s.v4.is/victorberland.com/homelab-1220/DSC01432.JPG)


## The server

The main centre of attention here is my Dell Poweredge R240 which basically is the heart of my system. I got it in the beginning of 2019 when I was 16 with a Xeon 2136, 16GB RAM, a 256GB SSD and three 1TB spinny drives (one WD red, the two others blue), and it's been serving all my needs smoothly. A madlad you may call me for buying a server brand new, but I honestly couldn't be arsed to bother with older hardware that needs Java JRE for the Idrac interface, is harder to find parts for, make more noise and consumes more electricity. And the second-hand market for servers here mostly consisted of the dreadful 2950s or the big R710s, both of which are becoming quite old now. This is sitting on my "rack" which actually is just the BROR metal shelving unit from Ikea. It's well built and the width perfectly fits a rack server.

I was initially running Proxmox probably the entire first year of owning the machine, but then I started to notice that more and more of my services were being migrated to much more efficitent Docker containers instead of full-fledged VMs. I was also starting to run short on RAM and the high prices on unbuffered DDR4 ECC sticks came haunting for me, so I had to find another solution which didn't involve upgrading from the 16GB I already had in it. The choice landed on Rancher OS which appealed to me being an OS designed to run Docker containers and just that. This meant as little overhead for the main OS as possible and most importantly as little maintenance and breakable parts as possible.

Because I already had most of my services already running in the Docker VM, migration was a simple process. Probably the most time-taking task was restoring the ZFS datapool which had all my archives on it. In retrospect I probably wouldn't have had to completely wipe that one too, but I just wanted a fresh slate to build my new system off of.

Running just Docker containers has made setting up new services and maintaining them so much easier. My reverse proxy automatically detects new containers and routes to them, i can easily backup a container and its volumes, and most importantly I have lots of resources to spare.

So this is what I'm running as of writing this:
- Traefik reverse proxy
- Minio object storage
- Nextcloud
- Gitea Git server
- Fathom analytics
- Thumbor image compression proxy
- Bandwidth Hero (I'll probably make a post about this in the future)
- Miniflux RSS
- Funkwhale music


So it's not that much to be honest. I have quite a few additional containers running too, but these are clients' websites that are hosted on my server. Every client deployment has a Docker Compose file which spins up all of the necessary containers for that site. Usually it's Cockpit CMS and a small container for Restic backups (more on that later).

There are a few things I would like to change upon in the future. Possibly move the file archives to an external encloure and have the three spinny drives in the server just for Minio object storage. When directly given the disks, Minio can run its erasure protection code. It's also possible to add drives to the array compared to RAID/ZFS where such wouldn't be possible. Monitoring is also something that needs to be improved. Currently I just have a free account at Uptime Robot that emails me if the system no longer can be reached, but I'd also like more advanced metrics such as system status through the IDRAC and UPS status. I've looked at LibreNMS for this and seems to suit my needs, but I just haven't gotten around to setting it up yet.


## Network

For my routing needs I just have a Ubiquiti EdgeRouter X handling my firewall and connectivity. There isn't really much exciting going on here, i haven't bothered or had a need for VLANS and the router just works all the time. This is my second unit as the first one I bought stopped working after a power outage (probably the sudden loss of power that killed it), so I invested in an Eaton 3S UPS which serves my needs and protects my lab from going down immediately if the power drops out. Brownouts and surges is something I used to be worried about, which now the UPS deals with and handles just fine.

I used to have a dumb 8-port Netgear switch between the EdgeRouter and server gear, but I really didn't need the extra ports, so I'm currently directly plugged into the ports of the router. I really only need two ports for uplink and IDRAC and having a switch for that was just a waste of power. If i start using my Raspberry Pi more extensively I may reintroduce it to the rack though.

Those who know me may know I love independence and self reliance, but I had to bite the bullet and give up some of that when it came to DNS. The convenience and relief when you don't have to host multiple nameservers yourself is worth it, and so I've gone for NS1's service for this. Their free plan lasted me quite a while, but I outgrew that one a month ago and had to upgrade to their paid plan at 300 USD per year. It's a steep price for just DNS, but at least I don't have to pay extra for cloud servers to handle that. They also have monitoring which lets me set up failover to a cloud server in case the one at home goes down. Could make for a fun project to do in the future.


## Backups
As mentioned previously, I use Restic to back up my volumes, and each deployment has its own Restic container which automatically backs everything up. I'm currently sending all this to Scaleway's object storage which is both fast and one of the cheapest offerings i've found that's also based in Europe. I only have around 300GB backed up to Scaleway, so the price is far from killing me, but I may look into using a Raspberry Pi as my local backup which then can dump everything to an external drive I can bring to our cabin for cold storage.


I hope this post was of use and inspiration for others out there, and if you have any questions or feedback I'd love to hear them. Just drop me an e-mail at vb@victorberland.com .
