# JWCUP
## G1GC

減少 pause time

* 較低的 throughput

比起 CMS ，較不需要調整參數

* CMS 也被認為較不適合非常大 heap 的情況

Region 根據 heap size 區分

* region 有分 young 和 old 

Young GC 發生時

* 依照狀況變成 survivor 或 old