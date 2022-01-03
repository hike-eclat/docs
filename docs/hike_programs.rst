HIKe programs documentation
===========================

Here you can find the documentation of HIKe programs from the point of view of the eCLAT developer.

hike_pass()
---------
Take the decision to pass the packet to the Linux kernel for further processing. The packet is not processed by the other programs in the eCLAT chain.

hike_drop()
---------
Take the decision to drop the packet. The packet is not processed by the other programs in the eCLAT chain.

show_pkt_info(u64 ``select_layers``, u64 ``user_info``)
---------------------------------------

Print debug information about a packet.
``select_layers`` is defined as a bitmap to select the layers that will be printed, with the following option bits:

LAYER_2=1; NET_LAYER=2; TRANSP_LAYER=4

``user_info`` is a u64 that is provided by the calling chain and printed by ``show_pkt_info``


monitor(u64 ``event``)
------------------
Increment the counter associated with the ``event`` number received in input. The map with all the event counts can be accessed by the user space.

ip6_dst_tbmon()
------------------
Perform token bucket monitoring per IPv6 destination. Update the tocken bucket state and return 0 if the packet is "in profile". The parameters of the token bucket are configured when the HIKe eBPF program is compiled.

ip6_sd_tbmon()
------------------
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

ip6_alt_mark()
------------------
Decode the Alternate Mark TLV in the Hop-by-hop Options Extension Header (done) and in the Destination Options Extension Header (work in progress).


ip6_hset_srcdst(u64 ``action``)
------------------
Hashset on ipv6 (src,dst)

Take the action argument in input.

``action`` == LOOKUP: check whether the packet is in
the hashset or not;

``action`` == ADD: add the packet to the hashset if
it is not already present;

``action`` == LOOKUP_AND_CLEAN: add the packet to the
hashset if it is not already present and clean up an expired entry.

ip6_sd_dec2zero(u64 ``count``)
------------------

Implement a counter-to-zero per IPv6 (source, destination) couple. Initialize the counter-to-zero with the input value ``count``. When the counter reaches zero, return zero and reset the counter to the input value ``count``.
