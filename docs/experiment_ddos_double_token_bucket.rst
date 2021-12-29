Example experiment (DDoS mitigation)
=====================================

In this experiment, we combine 7 HIKe programs inside an eCLAT script to implement a DDOS mitigation scenario.

In particular, we first use a token bucket meter to measure the packet rate per each IPv6 destination and detect "out of profile" flows. If the aggregate rate for a given IPv6 destination is "out of profile", we activate (only for the "out of profile" packet) another token bucket meter operating per (source, destination) couple. If the packet rate for a (source, destination) couple is "out of profile", we "blacklist" all packet with the specific (source, destination) for a time interval T=10 s. During the interval in which a flow is blacklisted, we sample one packet every 500 and redirect it over a layer 2 interface, on which we can capture and store or analyze the packets.

The source code of the eCLATscripts is reported `below <eCLAT script>`_.

.. Inside the container run: ``cd /opt/eclat-daemon && testbed/ddos_double_token_bucket_with_sampler.sh``

To execture the experiment, run the following command in the HIKe / eCLAT container:

.. code-block:: shell

   cd /opt/eclat-daemon && testbed/ddos_double_token_bucket_with_sampler.sh

A tmux will start, implementing the topology depicted below.
There are three namespaces:

* SUT (System Under Test)
* TG (Traffic Generator)
* COLLECTOR

The SUT is the namespace in which we run the eCLAT daemon and the HIKe VM is attached to the XDP hook on the incoming interface enp6s0f0. The TG is the namespace in which we generate traffic to be processed by our HIKe / eCLAT framework. The COLLECTOR namespace is used in some examples in which we redirect some incoming packets to the outgoing interface cl0 of the SUT.

.. code-block:: none

          +------------------+      +------------------+
          |        TG        |      |       SUT        |
          |                  |      |                  |
          |         enp6s0f0 +------+ enp6s0f0 <--- HIKe VM XDP loader
          |                  |      |                  |
          |                  |      |                  |
          |         enp6s0f1 +------+ enp6s0f1         |
          |                  |      |         + cl0  <-|- towards the collector
          +------------------+      +---------|--------+
                                              |
                                    +---------|------+
                                    |         + veth0|
                                    |                |
                                    |    COLLECTOR   |
                                    +----------------+

There are 7 windows in the TMUX, click with the mouse on the window name in the status bar to activate them.

In window TG1 and TG2 you can run the ping commands to generate traffic.
On TG1 the command ``ping -i 0.01 fc01::2`` which sends 100 p/s is displayed and ready to be executed.
On TG2 the command ``ping -i 0.5 fc01::3`` which sends 2 p/s is displayed and ready to be executed.

In window MAPS you can see the content of the eBPF maps.

In window CLT we have run the ``tcpdump -i veth0`` command to display the packets that are redirected to the collector.

To perform an experirent first run the 2 p/s ping ``ping -i 0.5 fc01::3`` on TG2. Go in MAPS window and can see that the flow with destination fc01::3 is monitored in the per destination token bucket and it remains IN_PROFILE. The packet monitor (map ``map_pcpu_mon``) counts the transmitted packets (code 0). 

Then add the 100 p/s ping ``ping -i 0.01 fc01::2`` on TG1. You will notice that after few seconds the ping are not replied, because the (src, dst) flow has been blacklisted (for 10 seconds). After 10 seconds the flow is removed from the blacklist and some ping replies are again received. Looking in the MAPS windows, you can see that token bucket per (src, dst) has been activated and that the packet monitor (map ``map_pcpu_mon``) shows many dropped packets (code 1). One packet every 500 packets is shown as redirected (code 2). You can check on the CLT window that the redirected packet has been captured by tcpdump.

eCLAT script
^^^^^^^^^^^^^^^^^^^^
.. code-block:: python

   # ddos_tb_2_levels
   # 
   # (ddos_tb_2_levels_sample_cst.eclat)
   #
   # first token bucket monitor per ip6 dst 
   # the out-profile packets are processed by a second token bucket per src,dst
   # the out-profile (src,dst) are blacklisted
   # for a time interval (e.g. 10 s) which is defined in ip6_hset.h: HIKE_IPV6_HSET_EXP_TIMEOUT_NS
   # token bucket parameters (rate, bucket) are defined in tb_defs.h
   # a packet every 500 blacklisted packets is redirected to an interface
   # the script is also counting the accepted, dropped and redirected packets

   from programs.mynet import hike_drop, hike_pass,  ip6_hset_srcdst, ip6_sd_tbmon, monitor, ip6_dst_tbmon, ip6_sd_dec2zero, l2_redirect
   from loaders.basic import ip6_sc

   # send all IPv6 packets to our chain
   ip6_sc[ipv6_sc_map] = { (0): (ddos_tb_2_lev) }
   ip6_sc.attach('DEVNAME', 'xdp')

   def ddos_tb_2_lev():
       PASS=0; DROP=1; REDIRECT=2
       ADD=1; LOOKUP=2
       BLACKLISTED = 0
       REDIRECT_IF_INDEX = 6
       IN_PROFILE = 0

       # (src,dest) in blacklist ?
       u64 : res = ip6_hset_srcdst(LOOKUP)
       if res == BLACKLISTED:
           # redirect one packet out of 500
           res = ip6_sd_dec2zero(500)
           if res == 0:
               monitor(REDIRECT)
               l2_redirect(REDIRECT_IF_INDEX) 
               return 0

           monitor(DROP)
           hike_drop()
           return 0

       # check the rate per (dst)
       res = ip6_dst_tbmon()
       if res != IN_PROFILE:
           # check the rate per (src,dst)
           res = ip6_sd_tbmon()
           if res != IN_PROFILE:
               # add (src,dest) to blacklist
               ip6_hset_srcdst(ADD)
               monitor(DROP)
               hike_drop()
               return 0

       monitor(PASS)
       hike_pass()
       return 0

