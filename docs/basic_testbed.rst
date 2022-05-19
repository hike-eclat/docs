Basic testbed 
-------------------------

In this simple experiment, we run the eCLAT script called basic_example.eclat. This script executes a HIKe program that can extract some information from the packet and display it in a log. For example, it can display transport ports for UDP or TCP packets.

The source code of the eCLAT script is reported :ref:`below<eclat-script-basic>`.

.. Inside the container run: ``cd /opt/eclat-daemon && testbed/basic_testbed.sh``

To execute the experiment, run the following command in the HIKe / eCLAT container:

.. code-block:: shell

   cd /opt/eclat-daemon && testbed/basic_testbed.sh

A tmux session will start. It deploys the topology depicted below, with the following three namespaces:

* SUT (System Under Test)
* TG (Traffic Generator)
* COLLECTOR

The SUT is the namespace in which we run the eCLAT daemon and the HIKe VM is attached to the XDP hook on the incoming interface enp6s0f0. The TG is the namespace in which we generate traffic to be processed by our HIKe / eCLAT framework. The COLLECTOR namespace can be used in some examples in which we redirect some incoming packets from the SUT to the COLLECTOR, but it is not used in this basic example. 

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

Commands to control the tmux session
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* To exit from the tmux session type ``Ctrl-b`` ``d``.

* To resume the session ``cd /opt/eclat-daemon && scripts/resume-tmux.sh``.

* To kill the tmux session ``cd /opt/eclat-daemon && scripts/kill-tmux.sh`` (note that the previous session is automatically killed if the experiment is executed again).

Windows in the tmux session
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are 9 windows in the TMUX, click with the mouse on the window name in the status bar to activate them.

In the TG1 and TG2 windows you can type and run the commands to generate traffic from the TG namespace:
* Command prepared in TG1: ``tcpreplay -i enp6s0f0 hike_v3/testbed/pkts/pkt_ipv6_udp.pcap``
* Command prepared in TG2: ``ping -i 5 fc01::3``

The MAIN, MAPS and DEBUG windows are in the default namespace of the container.
In the MAPS window you can see the content of the eBPF maps.
In the DEBUG window you can see the low-level debug printout of the HIKe programs.

The SUT, SUT2 and SUTDA windows are in the SUT namespace. The eCLAT daemon is executed in the SUTDA windows.

You can execute the tcpreply command from TG1 to send an udp packet and read the UDP source and destination ports written by the show_pkt_info HIKe/eBPF program in the log (shown in the DEBUG window).

You can execute the ping command from TG2 and it will send an ICMP packet every 5 seconds. In this case, the transport protocol 58 will be displayed in the log.

Now you can try to modify the eCLAT script basic_example.eclat: ``cd /opt/eclat-daemon && nano test/eclat_scripts/basic_example.eclat``.

For example, change line 20 from ``show_pkt_info(TRANSP_LAYER, myvar)`` to ``show_pkt_info(NET_LAYER | TRANSP_LAYER, 2000)``.

Now both Network Layer and Transport Layer information are displayed in the log, and the displayed "User info" passed as parameter to the show_pkt_info HIKe/eBPF program is changed from 1000 to 2000.



.. _eclat-script-basic:

eCLAT script basic_example.eclat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: python

   # basic example
   # 
   # (basic_example.eclat)
   #

   #from programs.mynet import hike_drop, hike_pass, monitor, show_pkt_info
   from programs.hike_default import hike_drop, hike_pass, monitor
   from programs.info import show_pkt_info
   from loaders.hike_default import ip6_simple_classifier

   # send all IPv6 packets to our chain
   ip6_simple_classifier[ipv6_simple_classifier_map] = { (0): (basic_example) }
   ip6_simple_classifier.attach('DEVNAME', 'xdp')

   def basic_example():

       LAYER_2=1; NET_LAYER=2; TRANSP_LAYER=4

       u64 : myvar = 1000
       show_pkt_info(TRANSP_LAYER, myvar)

       hike_pass()
       return 0
