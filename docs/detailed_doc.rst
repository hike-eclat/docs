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



Supported HIKe VM instructions
--------------------------------------------------------------

.. code-block:: text

  #### ALU instructions:
  64-bit:
  | Mnemonic     | Pseudocode
  |--------------|-------------------------
  | add  dst imm | dst += imm
  | add  dst src | dst += src
  | sub  dst imm | dst -= imm
  | sub  dst src | dst -= src
  | mul  dst imm | dst *= imm
  | mul  dst src | dst *= src
  | div  dst imm | dst /= imm
  | div  dst src | dst /= src
  | or   dst imm | dst \|= imm
  | or   dst src | dst \|= src
  | and  dst imm | dst &= imm
  | and  dst src | dst &= src
  | lsh  dst imm | dst <<= imm
  | lsh  dst src | dst <<= src
  | rsh  dst imm | dst >>= imm (logical)
  | rsh  dst src | dst >>= src (logical)
  | neg  dst     | dst = ~dst
  | mod  dst imm | dst %= imm
  | mod  dst src | dst %= src
  | xor  dst imm | dst ^= imm
  | xor  dst src | dst ^= src
  | mov  dst imm | dst = imm
  | mov  dst src | dst = src
  | arsh dst imm | dst >>= imm (arithmetic)
  | arsh dst src | dst >>= src (arithmetic)
  -----------------------------------------

  #### Endianess conversion (Byteswap) instructions:
  | Mnemonic | Pseudocode
  |----------|-------------------
  | le16 dst | dst = htole16(dst)
  | le32 dst | dst = htole32(dst)
  | le64 dst | dst = htole64(dst)
  | be16 dst | dst = htobe16(dst)
  | be32 dst | dst = htobe32(dst)
  | be64 dst | dst = htobe64(dst)
  -------------------------------

  #### Memory instructions:
  | Mnemonic            | Pseudocode
  |---------------------|-------------------------------------------
  | ld64    dst imm     | dst = imm
  | ldx8    dst src off | dst = *(uint8_t  *) (src + off)
  | ldx16   dst src off | dst = *(uint16_t *) (src + off)
  | ldx32   dst src off | dst = *(uint32_t *) (src + off)
  | ldx64   dst src off | dst = *(uint64_t *) (src + off)
  | st8     dst off imm | *(uint8_t  *) (dst + off) = imm
  | st16    dst off imm | *(uint16_t *) (dst + off) = imm
  | st32    dst off imm | *(uint32_t *) (dst + off) = imm
  | st64    dst off imm | *(uint64_t *) (dst + off) = imm
  | stx8    dst src off | *(uint8_t  *) (dst + off) = src
  | stx16   dst src off | *(uint16_t *) (dst + off) = src
  | stx32   dst src off | *(uint32_t *) (dst + off) = src
  | stx64   dst src off | *(uint64_t *) (dst + off) = src
  --------------------------------------------------------------------

  #### Branch instructions:
  64-bit:
  | Mnemonic         | Pseudocode
  |------------------|-------------------------------------------
  | ja   off         | PC += off
  | jeq  dst imm off | PC += off if dst == imm
  | jeq  dst src off | PC += off if dst == src
  | jgt  dst imm off | PC += off if dst > imm
  | jgt  dst src off | PC += off if dst > src
  | jge  dst imm off | PC += off if dst >= imm
  | jge  dst src off | PC += off if dst >= src
  | jlt  dst imm off | PC += off if dst < imm
  | jlt  dst src off | PC += off if dst < src
  | jle  dst imm off | PC += off if dst <= imm
  | jle  dst src off | PC += off if dst <= src
  | jset dst imm off | PC += off if dst & imm
  | jset dst src off | PC += off if dst & src
  | jne  dst imm off | PC += off if dst != imm
  | jne  dst src off | PC += off if dst != src
  | jsgt dst imm off | PC += off if dst > imm (signed)
  | jsgt dst src off | PC += off if dst > src (signed)
  | jsge dst imm off | PC += off if dst >= imm (signed)
  | jsge dst src off | PC += off if dst >= src (signed)
  | jslt dst imm off | PC += off if dst < imm (signed)
  | jslt dst src off | PC += off if dst < src (signed)
  | jsle dst imm off | PC += off if dst <= imm (signed)
  | jsle dst src off | PC += off if dst <= src (signed)
  | call imm         | f(r1, r2, ..., r5); Function call
  | exit             | return r0
  ---------------------------------------------------------------
  
