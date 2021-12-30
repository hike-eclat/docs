Detailed documentation
===========================

Compile/load eCLAT chains and HIKe programs 
--------------------------------------------

[to be completed]

The source code files (with extension ``.bpf.c``) of each package ``package_name`` are copied
in the folder ``eclat_daemon/components/programs/package_name``

The include files with extension ``.h`` are read from the same folder if present, if they are 
not available in the same folder, they are read from ``eclat_daemon/hike_v3/src``

Read and write from/to BPF maps
-------------------------------

Read from map

.. code-block:: python

  include os
  include json
  
  # Command Abstraction Layer
  include cal 
  
  BASE_PATH =  '/sys/fs/bpf/maps'
  PACKAGE = 'package_name'
  PROGRAM = 'program_name'
  MAP = 'map_name'
  map_path = f"{BASE_PATH}/{PACKAGE}/{PROGRAM}/{MAP}"
  map_as_array = []
        
  if os.path.exists(map_path):
    try :
        map_as_array = json.loads(cal.bpftool_map_dump(map_path))
    except Exception as e:
        print (e)
        print (map_path)
        return -1
  else:
    return -1
  return 0

