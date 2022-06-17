============
Installation
============

The instructions on how to build a docker container with the full test and development environment for HIKe / eCLAT are available here: `eclat-docker  <https://github.com/netgroup/eclat-docker>`_. 

The main source code repositories are:

- `eclat-daemon  <https://github.com/netgroup/eclat-daemon>`_ (eCLAT daemon, CLI, eCLAT language parser)
- `hike  <https://github.com/netgroup/hike-public>`_ (HIKe core functionality)

The HIKe programs are organized in packages. The needed packages are automatically downloaded by eCLAT
in the folders inside ``eclat-daemon/components/programs/`` (one folder per package). The source code 
repositories of the packages that are provided with the HIKe framework are documented  
`here <https://hike-eclat.readthedocs.io/en/latest/hike_programs.html>`_.

