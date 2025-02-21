;; Machine description for Xilinx MicroBlaze synchronization instructions.
;; Copyright (C) 2011-2018 Free Software Foundation, Inc.
;;
;; This file is part of GCC.
;;
;; GCC is free software; you can redistribute it and/or modify it
;; under the terms of the GNU General Public License as published
;; by the Free Software Foundation; either version 3, or (at your
;; option) any later version.
;;
;; GCC is distributed in the hope that it will be useful, but WITHOUT
;; ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
;; or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public
;; License for more details.
;;
;; You should have received a copy of the GNU General Public License
;; along with GCC; see the file COPYING3.  If not see
;; <http://www.gnu.org/licenses/>.

(define_insn "atomic_compare_and_swapsi"
  [(set (match_operand:SI 0 "register_operand" "=&d")		;; bool output
        (unspec_volatile:SI
          [(match_operand:SI 2 "nonimmediate_operand" "+Q")	;; memory
           (match_operand:SI 3 "register_operand" "d")		;; expected value
           (match_operand:SI 4 "register_operand" "d")]		;; desired value
          UNSPECV_CAS_BOOL))
   (set (match_operand:SI 1 "register_operand" "=&d")		;; val output
        (unspec_volatile:SI [(const_int 0)] UNSPECV_CAS_VAL))
   (set (match_dup 2)
        (unspec_volatile:SI [(const_int 0)] UNSPECV_CAS_MEM))
   (match_operand:SI 5 "const_int_operand" "")			;; is_weak
   (match_operand:SI 6 "const_int_operand" "")			;; mod_s
   (match_operand:SI 7 "const_int_operand" "")			;; mod_f
   (clobber (match_scratch:SI 8 "=&d"))]
  ""
  {
    output_asm_insn ("add  \t%0,r0,r0", operands);
    output_asm_insn ("lwx  \t%1,%y2,r0", operands);
    output_asm_insn ("addic\t%8,r0,0", operands);
    output_asm_insn ("bnei \t%8,.-8", operands);
    output_asm_insn ("cmp  \t%8,%1,%3", operands);
    output_asm_insn ("bnei \t%8,.+20", operands);
    output_asm_insn ("swx  \t%4,%y2,r0", operands);
    output_asm_insn ("addic\t%8,r0,0", operands);
    output_asm_insn ("bnei \t%8,.-28", operands);
    output_asm_insn ("addi \t%0,r0,1", operands);
    return "";
  }
)
