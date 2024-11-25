# 數位IC前段考題
這是集結各前輩分享後整理得面試考題
reference
https://www.dcard.tw/f/tech_job/p/238023076
https://www.dcard.tw/f/tech_job/p/240953224
## <font color=blue> 觀念題</font>
### 1. glitch
```
輸入的訊號改變或是經過一些邏輯運算，輸出訊號無法完全同步，造成短時間的錯誤波型輸出(這種時脈非常短)的錯誤訊號被稱為glitch

可以用gray code 減少glitch 的產生

如何解glitch以static power reduction 中的clock gated來說
加入一個"latch"電路
```
參考 https://hackmd.io/@derek8955/SkeOMZKzo#Clock-Gating
### 2. 除頻電路
```
1. 使用counter
2. 使用 !作為判斷
3. 比較特別的是 /3 => 2.5CYCLE
解法:
https://hackmd.io/@q7QVb5coQ1GGIzNCQcFyJw/verilog_div3code
```
### 3. clock gating作法
參考 https://hackmd.io/@derek8955/SkeOMZKzo#Clock-Gating
### 4. DMA (Direct Memory Access) 的運作原理 
```
用來提供外部裝置或是記憶體之間的直接高速資料傳輸，不需要經過CPU，節省CPU的資源
```
YOUTUBE (講解什麼是DMA)
https://www.youtube.com/watch?v=2_X9qD12_pw
CSDN
https://blog.csdn.net/weixin_51244852/article/details/134357186

### 5.合成時下constraint multi-cycle & false path 的目的, 並相對應的電路上要怎麼處理
```
multi- cycle path:
部分特定電路數據需要多個cycle才可以完成傳輸，若要延長cycle就需要另外設定multi-cycle放寬時序要求

false-path:
特定部分電路不影響電路的正確功能，所以可以忽略不檢查，用來減少不必要的違反time violation report
```
參考 
https://hackmd.io/KYHxs7UERNCLVKZZ4WPvXg


### 6. 
## <font color=blue> CDC</font>
可參考
1. https://hackmd.io/QyJXUtEERzCUbuocM0GkFg
2. https://hackmd.io/@TRChen/S1DOZOOS6
3. 老李系列
### 1. 低頻率的clock傳輸數據到高頻率的clock, 造成高頻率的clock多次重覆取到data, 如何解決, 並畫出電路
```
單bit
1. 2 dff synchronizer
2. pulse synchronizer
多bit
1. NDFF_BUS with Gray Code(多dff)
2. MUX_NDFF/MUX_PULSE for Data Path(快到慢會miss)
3. Handshake(latency 大, 面積小)
4. Asynchronous FIFO 
```

### 2. CDC中 slow clk 到 fast clk 時，在slow clk 的一個週期中，經歷了多次fast clk high edge，那麼對應的Gray Code 就會有多個 bits 發生變化，這個不會產生多個bits同步的問題嗎？
```
雖然slow clk 同步過來的值可能和之前的值相比有多個bit發生變化，但是這些bit的翻轉不是同時發生的
```

### 3. 為甚麼多bit數據傳輸, 不能使用1-bit synchronizer分別同步?
```
不同位元因為硬體電路中的寄生效應等等造成latency可能會不同，造成位元之間不同步(Data Convergence Issue)，有些位元提前到達或是延遲造成數據錯誤
```
### 4. Pulse synchronizer的原理以及應用場景
```
1. 將aclk的pulse轉換為一個level
2. 將2-flop syn 同步信號
3. 在bclk將同步過來的訊號轉為pulse - 用xor
注意:
將pulse syn 變成level，若這個level 無法滿足bclk的3 edge 要求，就無法同步
```
### 5. Mux synchronizer 的原理以及應用場景
```
用MUX作為control signal 用來控制
1. control path :　單bit 用來控制2 synchronizer傳遞enable給mux
2. data path    :  傳輸資料
```
![image](https://hackmd.io/_uploads/ry8rxInzJe.png)
reference :[ https://medium.com/@victor785413/%E9%80%B2%E9%9A%8E%E6%8A%80%E5%B7%A7-clock-domain-crossing-cdc-fpga-asic-design-1a67f6cbeadc]

### 6. MTBF(mean time between failures)是什麼
```
平均故障時間間隔，數值越高表示越穩定
總運作時間 = 21 小時
事件總數 = 3
MTBF = 總運作時間/事件數
MTBF = 21/3 = 7 小時
```

### 7. 畫出 Async FIFO & gray code 並解釋
reference : https://hackmd.io/QyJXUtEERzCUbuocM0GkFg?both

### 8. 什麼是metastability?
```
訊號在不同時序間傳遞，違反set time and hold time constrain，造成一些不穩定狀態，無法輸出正確的邏輯電位，可能造成電路錯誤

需要注意的是  不可能完全避免metastability
1. 這是一個物理現象，訊號輸出不會立刻到高邏輯或是低邏輯
2. ACLK 可能會剛好在BCLK變化時進行取樣，即使經過嚴謹的設計仍無法完全避免
```
### 9. Two flip-flop synchronizer
```
2個DFF組成的synchronizer
第一個syn是用來將input 第一次同步，訊號可能處於metastability
第二個syn是用來二次同步，經過第一次同步後的訊號，已經穩定
ps 若是還擔心可以用3dff syn但大部分的形況不需要
```

### 10. 為什麼要使用gray code
```
1. 減少cross clk glitch 產生
2. 提升同步穩定性
3. check FIFO empty or full easier
4. avoid data covergence
```


## <font color=blue> EDA & IC Design Flow </font>
### 1. 完整的Design flow + EDA Tool + 使用的檔案
<font color=red>**務必一定要背熟(EDA Tool 可以選擇自己常用的講就好)**</font>


|Stage                          |  EDA Tools                                   |  Input Files |Output Files  |
|-----------------|-------------|-------------|-----------|
|1. RTL Design                  |  Verilog/VHDL Editors                        |  Functional specifications, RTL files  |Completed RTL files|                             
|2. Functional Simulation       |  Synopsys VCS, Cadence Xcelium, QuestaSim    |  RTL files, Testbench files|Waveform files (VCD/FSDB), Simulation logs|
|3. Logic Synthesis             |  Synopsys Design Compiler, Cadence Genus     |  RTL files, SDC files, .lib files, Process technology files|Gate-Level Netlist, Timing reports, Area reports |
|4. Static Timing Analysis      |  Synopsys PrimeTime, Cadence Tempus          |  Gate-Level Netlist, SDC files, .lib files, SPEF files (opt.)  |Timing reports, SDF files     |       
|5. Gate-Level Simulation       |  Synopsys VCS, Cadence Xcelium, QuestaSim    |  Gate-Level Netlist, SDF files, Testbench files |Waveform files (VCD/FSDB), Simulation logs |                        
|6. Place & Route (PnR)         |  Cadence Innovus, Synopsys IC Compiler II    |  Gate-Level Netlist, LEF files, SDC files, Process technology files|DEF files, GDSII files, Layout reports      | 
|7. Parasitic Extraction        |  Synopsys StarRC, Cadence Quantus            |  DEF/GDSII files, LEF files|SPEF files                                   |                                           
|8. Post-Layout STA             |  Synopsys PrimeTime, Cadence Tempus          |  Gate-Level Netlist, SPEF files, SDC files   |Post-Layout Timing Reports, Updated SDF files |                            
|9. Post-Simulation             |  Synopsys VCS, Cadence Xcelium, QuestaSim    |  Post-Layout Netlist, SPEF files, Updated SDF files, Testbench files|Waveform files (VCD/FSDB), Functional and timing validation reports| 
|10. DRC & LVS Verification     |  Mentor Calibre, Synopsys IC Validator       |  GDSII files, LEF files, Process technology files  |DRC reports, LVS reports                              |                 
|11. Power Analysis             |  Synopsys PrimePower, Cadence Voltus         |  Gate-Level Netlist, SPEF files, VCD/SAIF files  |Power reports    



### 2. RTL simulation 和 gate level netlist simulation 和 Post simulation 差異
```
RTL simulation: check function & behavior work
Gate level netlist simulation: 加入sdf 模擬時序延遲，驗證netlist 和 RTL 功能一樣
Post simulation: 精確模擬無力布局layout後，是否滿足time constrain(加入RC值 and 布局latency ...)
```


| RTL Simulation | Gate-Level Simulation | 	Post-Layout Simulation |
| -------- | -------- | -------- |
| RTL 文件 + Testbench    | 門級網表 + SDF + Testbench    | 門級網表 + SDF + RC 數據 + Testbench     |
| 	驗證功能邏輯  | 驗證功能 + 時序   | 驗證功能 + 時序 + 物理效應     |

### 3. SDC 什麼是 false path、multi cycle、input/output delay、max/min delay

reference https://hackmd.io/KYHxs7UERNCLVKZZ4WPvXg


## <font color=blue> APR</font>
### 1. APR flow
```
floorplan -> powerplan -> placement -> cts & opt -> routing & opt
```
### 2. Power ring:
```
1. 穩定VDD & GND可以均勻的分布到所有區域
2. 減少IR-DROP
3. SUPPORT HIGH POWER
```
#### 常見問題
IR DRIP 過大　
原因 : power ring 寬度不足
解決 : 增加power ring 寬度、改善power mesh的密度

![image](https://hackmd.io/_uploads/SJE9RBoGJx.png)
### <font color=blue>3. EM/IR Drop:</font>
```
 EM:
是指金屬導線中的電子在高電流密度下流動時，因為電子對金屬離子的動量傳遞，導致金屬原子從一個地方移動到另一個地方，造成開路或是短路
IR DROP:
電源網路中因電流引起的電壓損失，造成性能下降，timing violation
```

### 有寫錯的地方請聯絡 : jiaho181@gmail.com
