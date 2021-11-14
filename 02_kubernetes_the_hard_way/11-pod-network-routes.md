# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

Les routes nécessaires au réseau de pods ont été créé pour le TP.

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
