.. _bgp:

BGP
---

:abbr:`BGP (Border Gateway Protocol) is one of the Exterior Gateway Protocols
and the de facto standard interdomain routing protocol. The latest BGP version
is 4. BGP-4 is described in :rfc:`1771` and updated by :rfc:`4271`. :rfc:`2858`
adds multiprotocol support to BGP.

IPv4
^^^^

A simple eBGP configuration:

**Node 1:**

.. code-block:: none

  set protocols bgp 65534 neighbor 192.168.0.2 ebgp-multihop '2'
  set protocols bgp 65534 neighbor 192.168.0.2 remote-as '65535'
  set protocols bgp 65534 neighbor 192.168.0.2 update-source '192.168.0.1'
  set protocols bgp 65534 address-family ipv4-unicast network '172.16.0.0/16'
  set protocols bgp 65534 parameters router-id '192.168.0.1'

**Node 2:**

.. code-block:: none

  set protocols bgp 65535 neighbor 192.168.0.1 ebgp-multihop '2'
  set protocols bgp 65535 neighbor 192.168.0.1 remote-as '65534'
  set protocols bgp 65535 neighbor 192.168.0.1 update-source '192.168.0.2'
  set protocols bgp 65535 address-family ipv4-unicast network '172.17.0.0/16'
  set protocols bgp 65535 parameters router-id '192.168.0.2'


Don't forget, the CIDR declared in the network statement MUST **exist in your
routing table (dynamic or static), the best way to make sure that is true is
creating a static route:**

**Node 1:**

.. code-block:: none

  set protocols static route 172.16.0.0/16 blackhole distance '254'

**Node 2:**

.. code-block:: none

  set protocols static route 172.17.0.0/16 blackhole distance '254'


IPv6
^^^^

A simple BGP configuration via IPv6.

**Node 1:**

.. code-block:: none

  set protocols bgp 65534 neighbor 2001:db8::2 ebgp-multihop '2'
  set protocols bgp 65534 neighbor 2001:db8::2 remote-as '65535'
  set protocols bgp 65534 neighbor 2001:db8::2 update-source '2001:db8::1'
  set protocols bgp 65534 neighbor 2001:db8::2 address-family ipv6-unicast
  set protocols bgp 65534 address-family ipv6-unicast network '2001:db8:1::/48'
  set protocols bgp 65534 parameters router-id '10.1.1.1'

**Node 2:**

.. code-block:: none

  set protocols bgp 65535 neighbor 2001:db8::1 ebgp-multihop '2'
  set protocols bgp 65535 neighbor 2001:db8::1 remote-as '65534'
  set protocols bgp 65535 neighbor 2001:db8::1 update-source '2001:db8::2'
  set protocols bgp 65535 neighbor 2001:db8::1 address-family ipv6-unicast
  set protocols bgp 65535 address-family ipv6-unicast network '2001:db8:2::/48'
  set protocols bgp 65535 parameters router-id '10.1.1.2'

Don't forget, the CIDR declared in the network statement **MUST exist in your
routing table (dynamic or static), the best way to make sure that is true is
creating a static route:**

**Node 1:**

.. code-block:: none

  set protocols static route6 2001:db8:1::/48 blackhole distance '254'

**Node 2:**

.. code-block:: none

  set protocols static route6 2001:db8:2::/48 blackhole distance '254'

Route Filter
^^^^^^^^^^^^

Route filter can be applied using a route-map:

**Node1:**

.. code-block:: none

  set policy prefix-list AS65535-IN rule 10 action 'permit'
  set policy prefix-list AS65535-IN rule 10 prefix '172.16.0.0/16'
  set policy prefix-list AS65535-OUT rule 10 action 'deny'
  set policy prefix-list AS65535-OUT rule 10 prefix '172.16.0.0/16'
  set policy prefix-list6 AS65535-IN rule 10 action 'permit'
  set policy prefix-list6 AS65535-IN rule 10 prefix '2001:db8:2::/48'
  set policy prefix-list6 AS65535-OUT rule 10 action 'deny'
  set policy prefix-list6 AS65535-OUT rule 10 prefix '2001:db8:2::/48'
  set policy route-map AS65535-IN rule 10 action 'permit'
  set policy route-map AS65535-IN rule 10 match ip address prefix-list 'AS65535-IN'
  set policy route-map AS65535-IN rule 10 match ipv6 address prefix-list 'AS65535-IN'
  set policy route-map AS65535-IN rule 20 action 'deny'
  set policy route-map AS65535-OUT rule 10 action 'deny'
  set policy route-map AS65535-OUT rule 10 match ip address prefix-list 'AS65535-OUT'
  set policy route-map AS65535-OUT rule 10 match ipv6 address prefix-list 'AS65535-OUT'
  set policy route-map AS65535-OUT rule 20 action 'permit'
  set protocols bgp 65534 neighbor 2001:db8::2 route-map export 'AS65535-OUT'
  set protocols bgp 65534 neighbor 2001:db8::2 route-map import 'AS65535-IN'

**Node2:**

.. code-block:: none

  set policy prefix-list AS65534-IN rule 10 action 'permit'
  set policy prefix-list AS65534-IN rule 10 prefix '172.17.0.0/16'
  set policy prefix-list AS65534-OUT rule 10 action 'deny'
  set policy prefix-list AS65534-OUT rule 10 prefix '172.17.0.0/16'
  set policy prefix-list6 AS65534-IN rule 10 action 'permit'
  set policy prefix-list6 AS65534-IN rule 10 prefix '2001:db8:1::/48'
  set policy prefix-list6 AS65534-OUT rule 10 action 'deny'
  set policy prefix-list6 AS65534-OUT rule 10 prefix '2001:db8:1::/48'
  set policy route-map AS65534-IN rule 10 action 'permit'
  set policy route-map AS65534-IN rule 10 match ip address prefix-list 'AS65534-IN'
  set policy route-map AS65534-IN rule 10 match ipv6 address prefix-list 'AS65534-IN'
  set policy route-map AS65534-IN rule 20 action 'deny'
  set policy route-map AS65534-OUT rule 10 action 'deny'
  set policy route-map AS65534-OUT rule 10 match ip address prefix-list 'AS65534-OUT'
  set policy route-map AS65534-OUT rule 10 match ipv6 address prefix-list 'AS65534-OUT'
  set policy route-map AS65534-OUT rule 20 action 'permit'
  set protocols bgp 65535 neighbor 2001:db8::1 route-map export 'AS65534-OUT'
  set protocols bgp 65535 neighbor 2001:db8::1 route-map import 'AS65534-IN'

We could expand on this and also deny link local and multicast in the rule 20
action deny.

RPKI
^^^^

:abbr:`RPKI (Resource Public Key Infrastructure)` is a framework :abbr:`PKI (Public Key Infastucrure)`
designed to secure the Internet routing insfratructure.
It associate a BGP route announcement with the correct originating :abbr:`ASN (Autonomus System Number)` and check it validation.
RPKI described in :rfc:`6480`. This is a separate server. You can find more details at RIPE-NNC_.

Imported prefixes during the validation may have values: valid, invalid and notfound.

* The valid state means that prefix and ASN that originated it match the :abbr:`ROA (Route Origination Authorizations)` base.
* Invalid means that prefix/prefix length and ASN that originated it doesn't match with ROA.
* Notfound means that prefix not found in ROA.

We can build route-maps for import, based on these states.
Simple RPKI configuration, where 'routinator' - RPKI cache server with ip '10.11.11.1'.

.. code-block:: none

  set protocols rpki cache routinator address '10.11.11.1'
  set protocols rpki cache routinator port '3323'

Example route-map for import. We can set local-preference logic based on states.
Also we may not import prefixes with the state 'invalid'.

.. code-block:: none

  set policy route-map ROUTES-IN rule 10 action 'permit'
  set policy route-map ROUTES-IN rule 10 match rpki 'valid'
  set policy route-map ROUTES-IN rule 10 set local-preference '300'
  set policy route-map ROUTES-IN rule 20 action 'permit'
  set policy route-map ROUTES-IN rule 20 match rpki 'notfound'
  set policy route-map ROUTES-IN rule 20 set local-preference '125'
  set policy route-map ROUTES-IN rule 30 action 'deny'
  set policy route-map ROUTES-IN rule 30 match rpki 'invalid'

.. _RIPE-NNC: https://github.com/RIPE-NCC/rpki-validator-3/wiki
