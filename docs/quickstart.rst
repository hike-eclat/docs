Quick Start
===========

Option 1. Working with offline and preinstalled doker image, on LINUX

#. Make sure you have docker installed in your PC
#. Download the docker image eclat.tar.gz
#. Load the image: ``docker load < eclat.tar.gz``
#. Create and execute container: ``docker run --rm -t -i --privileged --name eclat eclat:testbed  /sbin/my_init -- bash -l``

Option 2. Downloading the source code and building your docker image 

#. Follow the instructions to dowload, build and execute the container available :doc:`here<installation>`
