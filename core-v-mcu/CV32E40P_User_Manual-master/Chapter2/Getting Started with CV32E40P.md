# 上手CV32E40P

这一章讨论在你的设计中使用CV32E40P所要进行的初始步骤以及需求。



## 寄存器组

CV32E40P有两种不同的寄存器组实现。这两种不同的实现使用不同的工艺，分为触发器（`cv32e40p_register_file_ff.sv`）或者锁存器（`cv32e40p_register_file_latch.sv`）两种不同的实现。使用时必须要选择其中一种实现。对于这两种实现更多的信息可以在[*寄存器组*]()这一节查看。



## 时钟门控

CV32E40P需要使用者实现时钟门控模块。对于不同的工艺来说，该模块通常都是特定的，因此我们只提供一个仅用于仿真的时钟门控模块于`cv32e40p_sim_clock_gate.sv`。这个文件还包含了一个`cv32e40p_clock_gate`的模块，它包含以下的IO端口：

- `clk_i`：时钟输入
- `en_i`：时钟使能输入
- `scan_cg_en_i`：扫描时钟门控使能输入（即使`en_i`没有使能时，若该端口使能则也会激活时钟）
- `clk_o`：时钟门控输出

在CV32E40P中，时钟门控在`cv32e40p_sleep_unit.sv`以及`cv32e40p_register_file_latch.sv`两个文件中有使用。对于使用锁存器的寄存器组中时钟门控的行为请参考[*寄存器组*]()一节。

`cv32e40p_sim_clock_gate.sv`不能用作综合。对于ASIC或者FPGA综合的用途来说，需要使用者自行实现适合特定工艺下的时钟门控`cv32e40p_clock_gate`模块。

> 在官方给出的FPGA综合指南当中，直接使用FPGA厂商提供的`tc_clk_gating`模块来替换源码中的时钟门控模块：
>
> ```verilog
> tc_clk_gating core_clock_gate_i
> //this module is found in $COREVMCU/ips/tech_cells_generic/src/fpga/tc_clk_xilinx.sv
> (
>    .clk_i     ( clk_i       ),
>    .en_i      ( clock_en    ),
>    .test_en_i ( scan_cg_en  ),
>    .clk_o     ( clk_o       )
> 
> );
> ```
>
> 由于FPGA一般使用触发器工艺的寄存器组，因此寄存器组不需要考虑时钟门控模块的实现问题。