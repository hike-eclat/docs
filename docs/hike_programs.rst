HIKe programs documentation
===========================

hike_pass()
---------
Take the decision to pass the packet to the Linux kernel for further processing. The packet is not processed by the other programs in the eCLAT chain.

hike_drop()
---------
Take the decision to drop the packet. The packet is not processed by the other programs in the eCLAT chain.

monitor(u64 event)
------------------
Increment the counter associated with the event number received in input. The map with all the event counts can be accessed by the user space.

ip6_dst_tbmon()
------------------
Perform tocken bucket monitoring per IPv6 destination. Update the tocken bucket state and return 0 if the packet is "in profile". The parameters of the token bucket are configured when the HIKe eBPF program is compiled.

ip6_sd_tbmon()
------------------
Perform tocken bucket monitoring per IPv6 (source, destination) couple. Update the tocken bucket state and return 0 if the packet is "in profile". The parameters of the token bucket are configured when the HIKe eBPF program is compiled.

ip6_hset_srcdst(u64 action)
------------------
Hashset on ipv6 (src,dst)

Take the action argument (ARG2) in input.
ARG2 == LOOKUP: check whether the packet is in
 the hashset or not;
ARG2 == ADD: add the packet to the hashset if
 it is not already present;
ARG2 == LOOKUP_AND_CLEAN: add the packet to the
 hashset if it is not already present and clean up an expired entry.

ip6_sd_dec2zero(u64 count)
------------------

Implement a counter-to-zero per IPv6 (source, destination) couple. Initialize the counter-to-zero with the input value count. When the counter reaches zero, return zero and reset the counter to the input value count.
