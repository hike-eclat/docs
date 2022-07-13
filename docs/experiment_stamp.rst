Experiment on STAMP (Simple Two-way Active Measurement Protocol)
----------------------------------------------------

In this experiment we use the `basic testbed <https://hike-eclat.readthedocs.io/en/latest/experiments.html#basic-testbed>`_.
An HIKe chain will be deployed on the SUT node. It implements a `STAMP <https://datatracker.ietf.org/doc/rfc8972/>`_ Reflector.

Once the testbed is launched, from the tmux window of the TG, a command is ready to send a STAMP packet to the SUT. The packet will be processed and returned to the TG.

It is possible to check a few logs in the DEBUG tmux window. In another TG window, one can also capture the returned packet using tcpdump to inspect it more deeply.

eCLAT script for STAMP stamp.eclat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: python

  # time
  # 
  # (time.eclat)
  #

  from programs.stamp import stamp_mono
  from loaders.hike_default import ip6_simple_classifier

  # send all IPv6 packets to our chain
  ip6_simple_classifier[ipv6_simple_classifier_map] = { (0): (stamp) }
  ip6_simple_classifier.attach('DEVNAME', 'xdp')

  def stamp():
      stamp_mono()
      return 0


The eBPF code can be found at https://github.com/netgroup/hikepkg-stamp.
