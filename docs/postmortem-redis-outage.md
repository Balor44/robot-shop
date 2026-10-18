# Postmortem: Cart failures after Redis outage


Status: Resolved
Severity: Medium (cart broken, site still up)
Author: Nwafor Obinna
Date: 2026-10-18


## Summary


The Redis StatefulSet was scaled to zero. The cart service depends on Redis, so adding items to the cart failed while the rest of the shop kept working. Service was restored by scaling Redis back to one replica.


## Impact


* Duration: Sun Oct 18 14:58:19 PDT 2026 to Date: Sun, 18 Oct 2026 16:55:50 PDT 
* Users could browse the catalogue but could not add items to the cart
* The availability SLO alert did not fire


## Timeline (WSL clock, PDT)


1. <start time>: Redis scaled to zero replicas
2. <time>: Cart requests started failing
3. <time>: Error lines found in Loki for the cart service
4. <time>: Root cause identified as Redis having no running pods
5. <recovery time>: Redis scaled back to one replica, cart requests succeeded


## Detection


Detected through logs in Loki, using a query for errors from the cart service. The Grafana availability panel and the burn rate alert stayed green.


## Root cause


The Redis StatefulSet had zero running pods, so the cart service could not read or write cart data.


## What went well


* Centralised logs made the failing service easy to find
* Recovery took one command


## What went poorly


* No alert fired, so a person had to notice the problem
* The availability probe only checks the home page, which does not depend on Redis


## Action items


1. Add a probe that checks a cart API endpoint, so cart failures count against the SLO
2. Add an alert on cart error rate or on Redis pod availability
3. Run Redis with persistence and a readiness check, since a single replica is the real weakness


## Lessons learned


An SLI built only on the home page can say everything is fine while a key feature is down. SLIs should follow what users actually do.


