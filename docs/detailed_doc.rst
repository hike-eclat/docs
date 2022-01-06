Detailed documentation
===========================

Compile/load eCLAT chains and HIKe programs 
--------------------------------------------

[to be completed]

The source code files (with extension ``.bpf.c``) of each package ``package_name`` are copied
in the folder ``eclat_daemon/components/programs/package_name``

The include files with extension ``.h`` are read from the same folder if present, if they are 
not available in the same folder, they are read from ``eclat_daemon/hike_v3/src``


eCLAT transpiler labels (resolved at load time) 
--------------------------------------------

It is possible to define labels in eCLAT scripts that are replaced at load time when the eCLAT script
is transpiled. The value of a label is set with the ``--define`` option of the ``eclat.py``.
For example if the label ``DEVNAME`` is present in the eCLAT script ``script.eclat``, 
the value ``eth0`` can assigne to this label as follows:

``python eclat.py --load script.eclat --define DEVNAME eth0``

Read and write from/to BPF maps
-------------------------------

Read the full map into a python list

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
  
  # map_as_array contains the map as a list of (key, value) pairs
  print (map_as_array)

Update a map with a (key, value) pair using ``cal_map_update`` and in raw hex mode

.. code-block:: python

  include os
  from hex_types import u32 
  
  # Command Abstraction Layer
  include cal 
  
  BASE_PATH =  '/sys/fs/bpf/maps'
  PACKAGE = 'package_name'
  PROGRAM = 'program_name'
  MAP = 'map_name'
  map_path = f"{BASE_PATH}/{PACKAGE}/{PROGRAM}/{MAP}"

  if os.path.exists(map_path):
    try :
        # update using cal_map_update
        key = u32(256) # objects are converted using their to_hex() method
        value = 512 # int are converted with 8 bytes
        cal.cal_map_update(map_path, key, value)  
        
        # raw hex mode update
        hex_key = ["00","01","00", "00"]
        hex_value = ["00", "02", "00", "00","00", "00","00", "00"]
        cal.bpftool_map_update(map_path, hex_key, hex_value, map_reference_type="pinned", value_type="hex")
    except Exception as e:
        print (e, map_path)
        return -1
  else:
    return -1

Parsing packets in HIKe eBPF programs
--------------------------------------------

.. code-block:: none

  cur->mhoff   : mac header offset
  cur->nhoff   : nework header offset
  cur->thoff   : transport header offset
  cur->dataoff : the offset to the position that you still have to parse
                 (usually the packet up to cur->dataoff has already been parsed)

note that cur->thoff is not really the transport layer, but it can changed when parsing the packet

it usually starts as the first header after the basic network header,
a program that parses the headers after the basic header may decide to advance cur->thoff


eCLAT data types and operators 
------------------------------

=============== ==================
eCLAT data type eBPF (C) data type
=============== ==================
   u64            __u64 
   u32            __u32 
   u16            __u16
   u8             __u8
   s64            __s64 
   s32            __s32 
   s16            __s16
   s8             __s8
=============== ==================



