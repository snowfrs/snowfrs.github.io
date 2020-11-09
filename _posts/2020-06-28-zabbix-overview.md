---
title: Zabbix Overview
tags: zabbix
---
<!--more-->

![Zabbix-ports](/assets/img/blog/Zabbix-ports.png)

**Active agent**: The Zabbix agent periodically asks the Zabbix proxy (or Zabbix server, if configured to use the server directly) for the list of items that should be collected, and then the agent will keep collecting the item values using the configured intervals and sending the values to the proxy. The proxy does not need to poll all the clients for the values, so this is more efficient way of working.

**Passive agent**: The Zabbix agent passively waits for connections. When the Zabbix proxy requires the item values from a client, it connects to the client and gets the item values. This requires the proxy to schedule and initiate the connections to all agents, so it requires more work from the proxy.

Depending on the agent and item configurations, the same agent can be in active mode for some items and in passive mode for other items, if both ways are configured properly. Usually it is simpliest to just use one mode in items, preferably active (especially in large deployments).

Note that the active/passive mode used in **proxy** is not related to the active/passive mode of the **agent**. The proxy active/passive mode is only related to the communication between the Zabbix server and the Zabbix proxy: Active = proxy will connect to server, passive = server will connect to proxy. If a proxy is in passive mode, meaning that Zabbix server will poll Zabbix proxy for available data, an agent associated with that proxy can still be in active mode.

When a server or proxy is connecting to a passive agent, the default TCP destination port is **10050** (the IANA-assigned port for zabbix-agent).

For other Zabbix component connections (= connections to server or proxy) the default TCP destination port is **10051** (the IANA-assigned port for zabbix-trapper).