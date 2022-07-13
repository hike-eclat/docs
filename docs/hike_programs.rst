HIKe packages documentation
===========================

Here you can find the documentation of HIKe packages and programs from the point of view of the eCLAT developer.

The HIKe programs are organized in packages. Already available packages are listed hereafter, developers can
add more packages.

- `hike_default <#hike-default-package>`_ : default package with basic programs
- `meter <#meter-package>`_ : counters and token buckets for packet flows
- `sampler <#sampler-package>`_ : select a packet every N packet for a flow
- `info <#info-package>`_ : retrieve information from packet
- `alt_mark <#alt-mark-package>`_ : alternate marking in HIKe
- `tutorial <#tutorial-package>`_ : Deep Packet Inspection with a tutorial purpose
- `stamp <#stamp-package>`_ : Simple Two-Way Active Measurement Protocol with HIKe
- `eip <#eip-package>`_ : prototype implementation of Extensible In-band Processing (EIP)


hike_default package
---------

Default package with basic programs

hike_pass()
^^^^^^^^^^^
Take the decision to pass the packet to the Linux kernel for further processing. The packet is not processed by the next programs in the eCLAT chain.

hike_drop()
^^^^^^^^^^^
Take the decision to drop the packet. The packet is not processed by the next programs in the eCLAT chain.

monitor(u64 ``event``)
^^^^^^^^^^^
Increment the counter associated with the ``event`` number received in input. The map with all the event counts can be accessed by the user space.

ip6_hset_srcdst(u64 ``action``)
^^^^^^^^^^^
Hashset on ipv6 (src,dst)

Take the action argument in input.

``action`` == LOOKUP: check whether the packet is in
the hashset or not;

``action`` == ADD: add the packet to the hashset if
it is not already present;

``action`` == LOOKUP_AND_CLEAN: add the packet to the
hashset if it is not already present and clean up an expired entry.

l2xcon()
^^^^^^^^^^^

This program is used to redirect a packet at layer 2 (i.e. from one layer 2 interface to another layer 2 interface).

It looks up the incoming layer 2 interface id in a map that returns the outgoing layer 2 interface id.
The map is called ``l2xcon_map`` and it needs to be configured beforehand.

`stamp_xcon_map.py <https://github.com/netgroup/hikepkg-stamp/blob/main/python/stamp_xcon_map.py>` provides an example on how it is possible to modify the ``l2xcon_map`` map programmatically using python

using the command line the map can be updated as follows:

.. code-block:: shell

   bpftool map update pinned /sys/fs/bpf/maps/hike_default/l2xcon/l2xcon_map key hex {key_string} value hex {value_string}
   bpftool map update pinned /sys/fs/bpf/maps/hike_default/l2xcon/l2xcon_map key hex 01 00 00 00 value hex 02 00 00 00

The l2xcon() program does not return the flow of execution to the eCLAT chain, i.e. the packet is not processed by the next programs in the eCLAT chain.

ip6_kroute()
^^^^^^^^^^^

This program routes an IPv6 packet with a lookup in the kernel routing table.

The ip6_kroute() program does not return the flow of execution to the eCLAT chain, i.e. the packet is not processed by the next programs in the eCLAT chain.

meter package
---------

Counters and token buckets for packet flows (see `Github repo <https://github.com/netgroup/hikepkg-meter>`_)

ip6_dst_tbmon()
^^^^^^^^^^^
Perform token bucket monitoring per IPv6 destination. Update the tocken bucket state and return 0 if the packet is "in profile". The parameters of the token bucket are configured when the HIKe eBPF program is compiled.

ip6_sd_tbmon()
^^^^^^^^^^^
Perform token bucket monitoring per IPv6 (source, destination) couple. Update the tocken bucket state and return 0 if the packet is "in profile". The parameters of the token bucket are configured when the HIKe eBPF program is compiled. 

Key and value of the map ``pcpu_sd_tbmon`` are:

.. code-block:: c

  //see ip6_hset.h
  struct key {
    struct in6_addr saddr; // 16 bytes in network-order (big-endian)
    struct in6_addr daddr; // 16 bytes in network-order (big-endian)
  };
  /*
    see tb_defs.h
    rate is expressed in (tokens/(2^shift_tokens)) / (2^base_time_bits ns)
    bucket_size is expressed in tokens/(2^shift_tokens) 
    last_tokens is expressed in tokens/(2^shift_tokens)
    last_time is expressed in ns
  */
  struct value {
    U64 rate; U64 bucket_size;
    U64 last_tokens; U64 last_time;
    U64 base_time_bits; U64 shift_tokens;     
  } ;

ip6_dst_meter()
^^^^^^^^^^^
Counts the packets per IPv6 destination.

ip6_sd_meter()
^^^^^^^^^^^
Counts the packets per IPv6(source, destination) couple.

sampler package
---------

Select a packet every N packet for a flow (see `Github repo <https://github.com/netgroup/hikepkg-sampler>`_)

ip6_sd_dec2zero(u64 ``count``)
^^^^^^^^^^^

Implement a counter-to-zero per IPv6 (source, destination) couple. Initialize the counter-to-zero with the input value ``count``. When the counter reaches zero, return zero and reset the counter to the input value ``count``.

info package
--------------

Retrieve information from packet (see `Github repo <https://github.com/netgroup/hikepkg-info>`_)

show_pkt_info(u64 ``select_layers``, u64 ``user_info``)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Print debug information about a packet.
``select_layers`` is defined as a bitmap to select the layers that will be printed, with the following option bits:

LAYER_2=1; NET_LAYER=2; TRANSP_LAYER=4

``user_info`` is a u64 that is provided by the calling chain and printed by ``show_pkt_info``


alt_mark package
-------------------

Alternate marking in HIKe (see `Github repo <https://github.com/netgroup/hikepkg-alt_mark>`_)


ip6_alt_mark()
^^^^^^^^^^^^^^^^^
Decode the Alternate Mark TLV in the Hop-by-hop Options Extension Header (done) and in the Destination Options Extension Header (work in progress).

tutorial package
-----------------

Deep Packet Inspection with a tutorial purpose (see `Github repo <https://github.com/netgroup/hikepkg-tutorial>`_)

(work in progress)

stamp package
-----------------

Simple Two-Way Active Measurement Protocol with HIKe

Links:

- `Github repo <https://github.com/netgroup/hikepkg-stamp>`_
- `Experiments doc <https://hike-eclat.readthedocs.io/en/latest/experiments.html#>`_
- `STAMP RFC 8972 <https://datatracker.ietf.org/doc/rfc8972/>`_

stamp_mono()
^^^^^^^^^^^^^^^^^
Monolithic implementation of STAMP reflector in eBPF.
STAMP packets are processed using data written in maps.
Maps should be configured by a controller with data relative to:

- time (delta between boot time and real time is stored because the eBPF program can only access boot time);
- layer 2 MAC addresses;
- layer 3 IP addresses;
- layer 4 UDP ports;
- layer 2 interfaces to perform cross connect.

stamp()
^^^^^^^^^^^^^^^^^
Implementation of STAMP reflector in eBPF. Only the part relative to the actual STAMP packet contents plus the layer 2/3 addresses and the layer 4 ports.
Filtering, UDP checksum calculation, layer 2 cross connect and XDP pass are missing.

filter()
^^^^^^^^^^^^^^^^^
Checks if there is UDP and if destination port is STAMP, then returns 0. Otherwise returns 1.

udp_checksum()
^^^^^^^^^^^^^^^^^
Calculates UDP checksum of the packet.

eip package
-----------------
Programs for the processing of several EIP Information Elements.

Links:

- `Github repo <https://github.com/netgroup/hikepkg-eip>`_
- `Experiments doc <https://hike-eclat.readthedocs.io/en/latest/experiments.html#experiment-on-eip-extensible-in-band-processing>`_
- `Headers draft <https://eip-home.github.io/eip-headers/draft-eip-headers-definitions.html>`_

mcd()
^^^^^^^^^
Process the Compressed Path Tracing (CPT) Information Element (IE) (see the headers draft).

The eBPF program is executed in the intermediate nodes. When a probe packet is received, the Midpoint Compressed Data (MCD) is computed. It needs to read data from maps to work. The data comprises a timestamp, an interface load and an interface ID.

