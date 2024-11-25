## SDC 
<font color="#1936C9">**電路中定義constrain的重要文件提供時序約束**</font>
通常使用在以下三種場合
```
1. Synthesis (Design Complier)
2. STA(Primetime)
3. Place and Route (Innovus)
```

### <font color="#1936C9">Creat clock</font>
> create_clock -name CLK1 -period 10 [get_ports clk]

### <font color="#1936C9">Max/Min delay</font>
#### Max delay(setup time)
```
信號在傳遞到接收時脈域的建立時間窗口內需要保持穩定，否則接收器可能無法正確鎖存數據，確保延遲不會超過允許的範圍確保信號能夠在設定的時序約束內傳遞到接收端，以避免違反建立時間（setup time）要求。
```
> set_max_delay [-datapath_only] [-from <node_list>][-to <node_list>][-through <node_list>]
#### Min delay(hold time)
```
如果信號到達接收端的速度過快（即延遲過短），可能會破壞接收時脈域的保持時間要求，確保信號在傳遞過程中不會因延遲過短而觸發保持時間（hold time）違規
```
>set_min_delay [-from <node_list>] [-to <node_list>][-through <node_list>]

                        
reference https://blog.csdn.net/weixin_41230430/article/details/141886987

### <font color="#1936C9">Clock Uncertainty</font>
Clock Uncertainty 是時鐘信號的不確定性，來自 Clock Jitter 和 Clock Skew，在STA，它直接影響 Setup 和 Hold 的檢查，對設計性能有重要影響。
> set_clock_uncertainty 0.2 [get_clocks CLK1]

#### Clock Skew
```
Clock Skew 是指同一時鐘信號在不同元件（例如 Flip-Flops 或 Memory Cells）之間的到達時間差異。
```
##### How to fix clock skew
```
解決 Clock Skew 的核心方法是通過 時鐘樹綜合（CTS） 和物理設計優化來平衡時鐘信號的延遲。同時，使用 STA 工具和物理設計工具進行反覆分析和修正，確保設計在高性能和高速條件下仍能穩定運行
```

### <font color="#1936C9">Input/Output Delay</font>
IO邊界上的約束
#### **Input delay:** 
設計接收到輸入信號的時間與時鐘邊緣之間的延遲
外部設備 ---> (2.0ns 延遲) ---> 設計內部
> set_input_delay -max 2.0 -clock [get_clocks CLK1] [get_ports data_in]
![image](https://hackmd.io/_uploads/S1oP1_nGyx.png)

#### Output delay:
輸出信號到達外部設備的時間點相對於時鐘邊緣的延遲。
設計內部 ---> (3.0ns 延遲) ---> 外部設備

>set_output_delay -max 3.0 -clock [get_clocks CLK1] [get_ports data_out]
![image](https://hackmd.io/_uploads/BkJcJd3zke.png)

### <font color="#1936C9">False path</font>
**<font color=red>False Path 僅用於跨時鐘域或無效路徑，不要將真正需要滿足時序的路徑誤設為 False Path，否則會導致設計失效</font>**
False Path 是指在設計中存在的數據路徑，雖然工具會進行時序分析，但實際上這些路徑不會影響電路的正確功能。因此，這些路徑可以忽略時序檢查，從而減少不必要的時序違規報告和設計約束的壓力。
例如afifo 中的第一個dff一定會time violation 這時候就可以設定false path  不去檢查

> set_false_path -from [get_ports clk1] -to [get_ports clk2]

### <font color="#1936C9">Multicycle path</font>
對於某些特定設計（pipeline、CDC），數據可能需要多個時鐘週期才能完成傳輸
STA 工具通常默認一個時鐘週期內完成所有邏輯操作。若計算的延遲大於時鐘週期，則需要設置 Multicycle Path 來放寬時序要求。

假設一個跨時鐘域的同步器需要 3 個時鐘週期完成穩定化
> set_multicycle_path 3 -setup -from [get_pins async_data_reg/Q] -to [get_pins sync_reg/D]
set_multicycle_path 2 -hold -from [get_pins async_data_reg/Q] -to [get_pins sync_reg/D]


### <font color="#1936C9">False Path vs Multicycle Path 比較</font>
|          | false path | Multicycle Path|
| -------- | -------- | -------- |
| 比較      | 完全忽略該路徑的時序檢查     | 告訴工具延長該路徑的時序要求     |



### <font color="#1936C9">Clock Transition Time</font>
Clock Transition Time 是指時鐘信號從低電平到高電平（或反之）轉換所花費的時間





## SDF
<font color="#1936C9">**作為 STA 和模擬的輸入文件，用於分析延遲和驗證時序**</font>

```
* 邏輯合成後生成初步 SDF
dc_shell> read_verilog design_rtl.v
dc_shell> read_lib timing.lib
dc_shell> write_sdf design_initial.sdf

* 佈局佈線後生成 RC 提取的 SDF
icc_shell> read_netlist design_netlist.v
icc_shell> write_parasitics design.spef
icc_shell> write_sdf design_layout.sdf

* PrimeTime 生成最終 SDF
pt_shell> read_verilog design_netlist.v
pt_shell> read_lib timing.lib
pt_shell> read_spef design.spef
pt_shell> write_sdf design_final.sdf
```
### 總結: 

.SDF 文件生成的方式取決於設計流程中的階段：
```
邏輯綜合工具生成粗略的 SDF。
佈局佈線工具生成基於 RC 提取的 SDF。
靜態時序分析工具生成最終精確的 SDF。
```
生成的工具與指令：
```
dc_shell：write_sdf
IC Compiler：write_sdf
PrimeTime：write_sdf
```


## EDA Tool
### 1. Design Complier: 用來Synthesis
輸入:
```
RTL
.SDC
.LIB ..
```
輸出
```
Netlist.v
Design Constraint(DC後的.sdc)
```

### 2. Primetime: 檢查時序違反，還提供詳細的延遲路徑、slack 報告，以及對設計的優化建議
輸入:
```
Netlist.v
.sdc(DC後的.sdc)
.sdf(STA的primetime產生)
```
輸出
```
時序分析報告（如 setup/hold violations）。
優化建議。
```

## 有寫錯的地方請聯絡
jiaho181@gmail.com

