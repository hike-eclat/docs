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

  include cal
  include json
  include ok
  self.map_as_array = []
        
  if os.path.exists(self.map_path):
    try :
        self.map_as_array = json.loads(cal.bpftool_map_dump(self.map_path))
    except Exception as e:
        print (e)
        print (self.map_path)
        return -1
  else:
    return -1
  return 0

