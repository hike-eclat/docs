Experiment on EIP (Extensible In-band Processing)
----------------------------------------------------

.. code-block:: none

  +------------------+      +------------------+      +------------------+      +------------------+
  |        r1        |      |        r2        |      |        r3        |      |        r4        |
  |                  |      |                  |      |                  |      |                  |
  |              i12 +------+ i21          i23 +------+ i32          i34 +------+ i43              |
  |                  |      |                  |      |                  |      |                  |
  |                  |      |                  |      |                  |      |                  |
  +------------------+      +------------------+      +------------------+      +------------------+

In this experiment, we set up a testbed with four routers (running in separated network namespaces) connected as shown in the figure above.
The testbed will create a tmux session with multiple windows. In particular, we have one window for each router, plus 1 more window for routers r2 and r3 where the eCLAT daemon will be running. Lastly, a MAIN window, a MAPS window and a DEBUG window.

You can kill the entire session by running: ``cd /opt/eclat-daemon && scripts/kill-tmux.sh``. But you don't need to manually kill the session before starting a new one, as this operation will be performed automatically.

On this topology we can run experiments to test a few EIP Information Elements (IEs).
A packet containing EIP data is sent from r1 to r4. The data is processed by nodes r2 and r3 before reaching the destination.

The eBPF code can be found at https://github.com/netgroup/hikepkg-eip

Path Tracing example
^^^^^^^^^^^^^^^^^^^^^^^^
You can run the testbed with the command:

.. code-block:: shell

  cd /opt/eclat-daemon && components/eip/testbed/pt_eip_testbed.sh

This will deploy the :ref:`eCLAT script for Path Tracing<eclat-script-eip>` on routers r2 and r3.
You can re-attach to the tmux by running the script: ``cd /opt/eclat-daemon && scripts/resume-tmux.sh``.

You can now send a packet from r1 using tcpreplay, the command will be ready at the R1 window.
On the R4 window you can start tcpdump to analyze the received packet. Also in this case the command will be ready to be executed on the R4 window.
Use the DEBUG window to see the logs printed by both r2 and r3. It's likely that you'll see twice the same logs if the packet is forwarded correctly, because logs from both routers will be shown.

.. _eclat-script-eip:

eCLAT script for Path Tracing eip_pt.eclat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: python

    # eip_pt
    # 
    # (eip_pt.eclat)
    #

    from programs.hike_default import hike_pass
    from programs.eip import mcd, hello
    from loaders.hike_default import ip6_simple_classifier

    # send all IPv6 packets to our chain
    ip6_simple_classifier[ipv6_simple_classifier_map] = { (0): (eip_pt) }
    ip6_simple_classifier.attach('DEVNAME', 'xdp')

    def eip_pt():
        u64 : res = mcd()
        if res == 0:
            hello()
        hike_pass()
        return 0
