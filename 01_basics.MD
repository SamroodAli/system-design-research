# System Design
What is system design ?
- System design is typically designing large scale systems, basically how we scale up

# Example of delicious
1) First we got to buy a domain from a vendor like godaddy,namecheap etc
- These vendors in turn buy our domain from ICANN. 
ICANN is a Non profit entity that maintains these records. These vendors pay ICANN for maintenance and upkeep
- Now that we got a domain, we need to start pointing the domain to our computer/server, one machine in the case of delicious.
We have to tell the entire world that when they type `del.icio.us`, they should connect to our computer.
Why ? because people can easily remember domains, and not IP addresses, just like how we save 
phone numbers with the name of the contact so that it's easy to remember
- How do we do this domain <> Ip address mapping ? Our vendors like namecheap, godaddy provides interfaces/dashboards
for us to add these mappings. ( Look up `A records` and `C records`);
- Now if someones looking up `del.icio.us`, they can know which computer/ip address to connect. Obviously it would be
taxing on ICANN if every internet user requests ICANN for the IP addresses associated with domains, and ICANN is a Non-profit
organization, so we have other parties helping with this lookup. Parties who have vested interest in us accessing the internet
and these are our ISPs (internet service providers) as well as big tech companies like Google who maintain DNS servers. 
If a different ISP provides faster internet, we would obviously switch over, and Ip address resolution is the firt step
in the request-response cycle, so if the first step itself is slow, the entire internet would feel slow, so ISPs have a huge
interest in maintaining their own DNS servers. DNS servers are servers created for the sole purpose of answering the IP addresses (or other domains) associated with the domains
we search for. But ofcourse, ICANN is the source of truth.
- Now that we have every user accessing these various DNS servers maintained by different parties apart from ICANN,
we essentially have solved the problem of ICANN being a single point of failure
- But now we have a different problem. All these DNS servers must maintain the correct domain IP address mapping, and for this
need to request ICANN for updates to the mapping. Now, if all these DNS servers requested a copy of the mapping from
ICANN, it would overload ICANN and would bring ICANN down. These copies are also huge amount of information and would introduce
further bottlenecks and latencies. In order to solve, DNS servers only fetch updates. DNS servers keep a timestamp of the last update
and passes this to ICANN and ICANN responds with the updates since this timestamp.
DNS servers can now update their entries as well as the timestamp.
- When users search for a domain, some DNS servers look up ICANN if an IP address for a domain is not found, 
but this depends on the implementation of the DNS server and many DNS servers don't do this. This is why there is a certain expectation
around how fast changes to this mapping are reflected worldwide. We can expect anywhere from 6 hours to 24 hours on average for every
DNS to be updated with the changes we introduce to the domain-ip address mapping. This is why most domain registrars show that it would
take around 24 hours for the change to domains to be introduced.
- Ofcourse browsers also maintain copy of the IP addresses of the domains already resolved. This often has a Time to live (ttl) after which it is invalidated and is often
also invalidated when the IP address doesn't return a proper response or is missing.
- Now that we have an IP address, how do we locate the machine associated with the IP address ?
- This is solved as IP addresses are routable as compared to mac addresses. Internet nodes like routers talk to each other till users
find the IP address. Any device that connects to the internet is allocated an IP address by ISPs. This can be different everytime we
connect to the internet and is why it's called a dynamic IP address.

2) Leased lines and static IP addresses
- Each time we connect to the internet, we can possibly get allocated different IP addresses. This means, we cannot store
our dynamic IP address in ICANN as it can change easily. We cannot keep changing the IP address associated with
our domain every time our IP address changes as ICANN changes take time.
- This is where we can get a static IP address, and possibly a leased line dedicated to our machine as well. A leased line
is a dedicated physical connection provided by our ISP to our machine/network. We can contact our ISPs to get one,
and is typically very expensive. A static IP address is a fixed IP address dedicated to our network.

3) Full stack
Once users can find our machine with the static IP address, we can write code full stack code to route users 
and show appropriate pages and content. But now this is where we start facing issues
- When we update our code, we have to copy updated code, stop existing proces and start running the new code.
This will lead to downtime
- Our machines/laptops can restart/reboot/crash for various reasons. All these can lead to downtime
- There can be other hardware issues or infrastructure issues like losing electricitiy. These also would lead to downtime
- There is also limits to resources. One machine like a laptop has only limited resources like RAM, storage etc. We would
soon reach bottlenecks here as well

##  How will we solve these problems delicious faced ?
- We can introduce multiple machines. This is a good idea but now have introduced multiple new problems like
a) Which computer's IP address should we give in ICANN ?
b) Should each of these new machines have their own database ?

### Load balancer
- We can setup one machine that routes incoming web traffic to other machines. This machine is called a load balancer and solves the
problem of needing to setup each IP address of each machine in ICANN
- Ofcourse now load balancer would be a single point of failure. To solve this, we can introduce `redundancy` 
by setting up multiple load balancers and buy multiple static IP addresses for each load balancer. 
ICANN does let us map multiple IP addresses to one domain.

```bash
dig devstoic.com
```
Using `dig`, we can find out all the static IP addresses associated with a domain

- Now have solved the problem of having limited resources with multiple machines and solved
the problem of ICANN domain-ip mapping by setting up load balancers

### Would load balancers go down
- A single load balancer can become a single point of failure which is why we can have multiple
load balancers each with it's own static IP address
- Load balancers are very thin clients in that all they do is route incoming traffic to other machines
- We would have changes in our codebase but changes to load balancer routing logic is very very rare if it even happens,
so there are no code deployments or downtime with load balancers that we need to worry about

Because of all these reasons, the chance of a load balance going down is very very low.
