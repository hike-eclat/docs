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

In the TG1 and TG2 windows you can run the ping commands to generate traffic from the TG namespace.
On TG1 the command ``ping -i 0.01 fc01::2`` which sends 100 p/s is displayed and ready to be executed.
On TG2 the command ``ping -i 0.5 fc01::3`` which sends 2 p/s is displayed and ready to be executed.

The MAIN, MAPS and DEBUG windows are in the default namespace of the container.
In the MAPS window you can see the content of the eBPF maps.
In the DEBUG window you can the low-level debug printout of the HIKe programs.

The SUT, SUT2 and SUTDA windows are in the SUT namespace. The eCLAT daemon is executed in the SUTDA windows.

The CLT windows runs in the COLLECTOR namespace. In the CLT window we could run the ``tcpdump -i veth0`` command to display the packets that are redirected to the collector.

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
