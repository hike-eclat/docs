Basic testbed 
-------------------------

In this simple experiment, we run the eCLAT script called basic_example.eclat.

The source code of the eCLAT script is reported `below <eCLAT script>`_.

The eCLAT script only displays some information in the log.

.. Inside the container run: ``cd /opt/eclat-daemon && testbed/basic_testbed.sh``

To execute the experiment, run the following command in the HIKe / eCLAT container:

.. code-block:: shell

   cd /opt/eclat-daemon && testbed/basic_testbed.sh

A tmux will start, implementing the topology depicted in the DDoS mitigation example. Refer to that example
for the instructions on how to manage the tmux session.

Also the topology is the same, with the three namespaces SUT, TG, COLLECTOR.
The SUT is the namespace in which we run the eCLAT daemon and the HIKe VM is attached to the XDP hook
on the incoming interface enp6s0f0. 
The TG is the namespace in which we generate traffic to be processed by our HIKe / eCLAT framework.
The COLLECTOR namespace is used in some examples in which we redirect some incoming packets to the outgoing interface cl0 of the SUT.

In windows TG1 and TG2 you can run the ping commands to generate traffic.

eCLAT script basic_example.eclat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: python

    # basic example
    # 
    # (basic_example.eclat)
    #

    #from programs.mynet import hike_drop, hike_pass, monitor, show_pkt_info
    from programs.hike_default import hike_drop, hike_pass, monitor
    from programs.info import show_pkt_info
    from loaders.basic import ip6_sc

    # send all IPv6 packets to our chain
    ip6_sc[ipv6_sc_map] = { (0): (basic_example) }
    ip6_sc.attach('DEVNAME', 'xdp')

    def basic_example():

        LAYER_2=1; NET_LAYER=2; TRANSP_LAYER=4

        u64 : myvar = 1000
        show_pkt_info(TRANSP_LAYER, myvar)

        hike_pass()
        return 0
