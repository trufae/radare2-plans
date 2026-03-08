
• I removed the trivial core_print_string wrapper from cmd_print.inc.c:6739. data is now treated as non-null as you requested,
  and the only local helper left is core_print_string_or_json:6739, because it still removes a real duplicated branch. The plain
  ps* and pu paths in the main switch now call r_print_string directly in cmd_print.inc.c:8367 and cmd_print.inc.c:8533, which
  saves lines and removes the pointless null checks.

  I verified it with make -j > /dev/null and smoke checks for psb, psw, ps, psx, and pu.

  Leftover APIs that still look removable or reducible:

  - r_print_offset_sg in r_print.h:249: dead declaration, no implementation.
  - r_print_addr_tostring and r_print_byte_tostring in print.c:395 and print.c:711: no in-tree users outside print.c, so they can
    become private or disappear after the wrappers go away.
  - r_print_addr, r_print_byte, r_print_section, r_print_offset, and r_print_string in print.c:463, print.c:747, print.c:934,
    print.c:580, and print.c:821: these are still print-through wrappers keeping RPrint coupled to RCons.
  - r_print_string_strbuf in print.c:753: no in-tree caller outside print.c:821 yet. Either start using it from cmd_print, or
    make it private until it is fully cleaned from RCons touches.

  The next concrete call sites to collapse are in print.c:1263, print.c:1391, print.c:1483, print.c:1563, plus the remaining
  r_print_section uses in cmd_print.inc.c:9034 and cmd_print.inc.c:9166, and the r_print_offset wrapper still used by disasm in
  disasm.c:3526 and disasm.c:8130.
