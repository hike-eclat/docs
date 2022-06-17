Quick Start
===========

Download the source code and build your docker image 

#. Follow :doc:`these instructions<installation>` to dowload, build and execute the container. 

.. Option 1. Working with offline and preinstalled doker image (only on LINUX)
  #. Make sure you have docker installed in your PC
  #. Download the docker image eclat.tar.gz
  #. Load the image: ``docker load < eclat.tar.gz``
  #. Create and execute container: ``docker run --rm -t -i --privileged --name eclat eclat:testbed  /sbin/my_init -- bash -l``

