# JTAG_driver
A verilog driver module that meet the JTAG standard

	• Signal description
		○ dsp_core_clk: using dsp core clk source.
		○ TCK: the freqency of TCK is dsp_core_clk divided by 4(spec recommend).
		○ TMS, TRSTN, TDI, TDO: no special comments.
	• Set up process
		○ Instance jtag_tb in tb.
		○ Find dsp core clk in DUT and connect it to jtag_tb dsp_core_clk port.
		○ Connect DUT corresponding top level JTAG IO PAD to jtag_tb.
			
	• Task description
		○ jtag_reset;
			§  de-assert and assert the JTRSTN port to complete full reset operation.
		○ jtag_bypass;
			§ Apply a bypass operation.
		○ Jtag_idcode_read( output [31:0] rdata);
			§ Read the DEVID value in JTAG access port (idcode detail on xtensa debug guide page 13)
		○ Jtag_pwrctl_read( output [7:0] rdata);
			§ Read pwrctl reg value (pwrctl detail on xtensa debug guide page 30)
		○ Jtag_pwrctl_write( input [7:0] wdata);
			§ Write pwrctl reg value (pwrctl detail on xtensa debug guide page 30)
		○ Jtag_pwrstat_read( output [7:0] rdata);
			§ Read pwrstat reg value (pwrctl detail on xtensa debug guide page 34)
		○ Jtag_narsel_read( input [6:0] regno, output [31:0] rdata, output busy, output error);
			§ Read reg in dsp debug module selected by Nexus reg address through JTAG interface
			§ In addition, this task will also tell if read operation is successful by error bit (page 18)
			§ Dsp debug module Nexus reg address map is on xtensa debug guide page 7
		○ Jtag_narsel_write( input [6:0] regno, input [31:0] wdata, output busy, output error);
			§ Write reg in dsp debug module selected by Nexus reg address through JTAG interface
			§ In addition, this task will also tell if write operation is successful by error bit (page 18)
			§ Dsp debug module Nexus reg address map is on xtensa debug guide page 7
	• Jtag tb sanity test check
		○ The sanity test check start from line 387 in initial block in file jtag_tb.v
