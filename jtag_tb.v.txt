//**************************************************************************************
//
// Module:  jtag_tb
//
// Owner:           Zhenyang Wu
// Creation Date:   2022.08.08 (yyyy-mm-dd)
//
// Description: This module implements the testbench function which controls and
//      monitors the jtag interface. 
//
//**************************************************************************************
`timescale 1ns/1ps

`define JPWRCTL 5'b01000
`define JPWRSTAT 5'b01001
`define JNARSEL 5'b11100
`define JIDCODE 5'b11110
`define JBYPASS 5'b11111

`define IDCODE_XTENSA 32'h120034e5

module jtag_tb(
    input wire dsp_core_clk,

    output reg TCK,
    output reg TMS,
    output reg TRSTN,
    output reg TDI,
    input  wire TDO
);

reg [1:0] r_count;
reg [31:0] temp_32;
reg [7:0] temp_8;
reg [4:0] IR_reg;
integer i;

initial begin
    TCK = 1'b0;
    TMS = 1'b1;
    TRSTN =1'b0;
    TDI = 1'b1;
    r_count = 2'b0;
    temp_32 = 32'b0;
    temp_8 = 8'b0;
end

//JTCK should be dsp core clk div by 4 (spec recommended)
always@(posedge dsp_core_clk) begin
    r_count = r_count+1'b1;
    if(r_count == 2'b11)
        TCK = ~TCK;
end

task jtag_TMS_1;
    @(negedge TCK);
    TMS = 1'b1;
endtask

task jtag_TMS_0;
    @(negedge TCK);
    TMS = 1'b0;
endtask

task jtag_reset;
    jtag_TMS_1;
    @(negedge TCK);
    TRSTN = 1'b0;
    repeat(20)@(posedge TCK);
    @(negedge TCK);
    TRSTN = 1'b1;
    jtag_TMS_0;
endtask //jtag_reset

task jtag_idcode_read;
    output [31:0] rdata;

    IR_reg = `JIDCODE; 
    $display("JTAG select IR IDCODE.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end
    //repeat(2) jtag_TMS_1;
    repeat(1) jtag_TMS_1;
    $display("JTAG update IR.");
    
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR.");
    @(posedge TCK);
    for(i=0;i<32;i=i+1) begin
        @(posedge TCK);
        temp_32[i] = TDO;
    end

    $display("JTAG IDCODE is %h", temp_32);
    repeat(2) jtag_TMS_1;
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
    //if(temp_32 !== `IDCODE_XTENSA)
    //    $error("The IDCODE is mismatched, expected 0x%h, received 0x%h. Please check if jtag tb is connected correctly.", `IDCODE_XTENSA, temp_32);

    rdata = temp_32;
endtask

task jtag_pwrctl_write;
    input [7:0] wdata;
    
    IR_reg = `JPWRCTL; 
    $display("JTAG select IR JPWRCTL.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end

    repeat(1) jtag_TMS_1;
    $display("JTAG update IR.");
    
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR.");
    //@(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(negedge TCK);
        TDI = wdata[i];
        if(i == 7) begin
            TMS = 1'b1;
        end
    end
    $display("JTAG PWRCTL write value is 0x%h", wdata);
    repeat(1) jtag_TMS_1;
    $display("JTAG update DR.");
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
endtask

task jtag_pwrctl_read;
    output [7:0] rdata;
    
    IR_reg = `JPWRCTL; 
    $display("JTAG select IR JPWRCTL.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end

    repeat(1) jtag_TMS_1;
    $display("JTAG update IR.");
    
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR.");
    @(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(posedge TCK);
        temp_8[i] = TDO;
    end
    $display("JTAG PWRCTL read value is 0x%h", temp_8);
    repeat(2) jtag_TMS_1;
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
    rdata = temp_8;
endtask

task jtag_pwrstat_read;
    output [7:0] rdata;
   
    IR_reg = `JPWRSTAT; 
    $display("JTAG select IR JPWRSTAT.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end

    repeat(1) jtag_TMS_1;
    $display("JTAG update IR.");
    
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR.");
    @(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(posedge TCK);
        temp_8[i] = TDO;
    end
    $display("JTAG PWRSTAT read value is 0x%h", temp_8);
    repeat(2) jtag_TMS_1;
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
    rdata = temp_8;
endtask

task jtag_bypass;
    IR_reg = `JBYPASS; 
    $display("JTAG select IR BYPASS.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end
    repeat(1) jtag_TMS_1;
    $display("JTAG update IR.");
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
endtask

task jtag_narsel_read;
    input [6:0] regno;
    output [31:0] rdata;
    output busy;
    output error;
    reg rbw;
    reg [7:0] nar;
    reg [31:0] ndr;
    reg [39:0] DR_reg;

    rbw = 1'b0;
    nar = {regno,rbw};
    ndr = 32'h0;
    DR_reg = {ndr,nar};

    IR_reg = `JNARSEL; 
    $display("JTAG select IR NARSEL.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end

    repeat(1) jtag_TMS_1;
    $display("JTAG update IR");
    
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR NAR");
    //@(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(negedge TCK);
        TDI = nar[i];
        if(i == 7) begin
            TMS = 1'b1;
        end
    end
    repeat(1) jtag_TMS_1;
    $display("JTAG update DR");
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR NDR");
    @(posedge TCK);
    for(i=0;i<32;i=i+1) begin
        @(posedge TCK);
        temp_32[i] = TDO;
    end

    repeat(2) jtag_TMS_1;
    $display("JTAG update DR");
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR NAR");
    @(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(posedge TCK);
        temp_8[i] = TDO;
    end
    
    repeat(2) jtag_TMS_1;
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
    jtag_reset;

    rdata = temp_32;
    busy = temp_8[1];
    error = temp_8[0];
endtask

task jtag_narsel_write;
    input [6:0] regno;
    input [31:0] wdata;
    output busy;
    output error;
    reg rbw;
    reg [7:0] nar;
    reg [31:0] ndr;
    reg [39:0] DR_reg;

    rbw = 1'b1;
    nar = {regno,rbw};
    ndr = wdata;
    DR_reg = {ndr,nar};

    IR_reg = `JNARSEL; 
    $display("JTAG select IR NARSEL.");
    repeat(2) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift IR.");
    for(i=0;i<5;i=i+1) begin
        @(negedge TCK);
        TDI = IR_reg[i];
        if(i == 4) begin
            TMS = 1'b1;
        end
    end

    repeat(1) jtag_TMS_1;
    $display("JTAG update IR");
    
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR NAR");
    //@(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(negedge TCK);
        TDI = nar[i];
        if(i == 7) begin
            TMS = 1'b1;
        end
    end
    repeat(1) jtag_TMS_1;
    $display("JTAG update DR");
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR NDR");
    //@(posedge TCK);
    for(i=0;i<32;i=i+1) begin
        @(negedge TCK);
        TDI = ndr[i];
        if(i == 31) begin
            TMS = 1'b1;
        end
    end

    repeat(1) jtag_TMS_1;
    $display("JTAG update DR");
    repeat(1) jtag_TMS_1;
    repeat(2) jtag_TMS_0;
    $display("JTAG shift DR NAR");
    @(posedge TCK);
    for(i=0;i<8;i=i+1) begin
        @(posedge TCK);
        temp_8[i] = TDO;
    end
    
    repeat(2) jtag_TMS_1;
    repeat(1) jtag_TMS_0;
    $display("JTAG back to idle");
    jtag_reset;

    busy = temp_8[1];
    error = temp_8[0];
endtask

initial begin
    reg [7:0] pwrctl_rdata;
    reg [7:0] pwrstat_rdata;
    reg [31:0] narsel_rdata;
    reg [31:0] idcode_rdata;
    reg narsel_busy;
    reg narsel_error;

    #9700000;
    jtag_reset;
    #1000;
    //read DEVID in JTAG accessport
    jtag_idcode_read(idcode_rdata);
    if(idcode_rdata !== `IDCODE_XTENSA)
        $error("JTAG IDCODE mismatch, expected 0x%h, received 0x%h", `IDCODE_XTENSA, idcode_rdata);

    jtag_pwrctl_write(8'h07); //wakeup debug module
    #1000;
    jtag_reset;
    jtag_pwrctl_write(8'h87); //enable jtagdebuguse
    #1000;

    //read device id through NAR in dsp debug module
    jtag_narsel_read(7'h72, narsel_rdata, narsel_busy, narsel_error);
    $display("narsel_rdata=0x%h, busy = 0x%h, error = 0x%h", narsel_rdata, narsel_busy, narsel_error);
    if(narsel_rdata !== `IDCODE_XTENSA)
        $error("NAR DEVID mismatch, expected 0x%h, received 0x%h", `IDCODE_XTENSA, narsel_rdata);

    //write and read back data to DDR reg through NAR in dsp debug module
    jtag_narsel_write(7'h45, 32'hdeadbeef, narsel_busy, narsel_error);
     $display("narsel write 0xdeadbeef, busy = 0x%h, error = 0x%h", narsel_busy, narsel_error);
    jtag_narsel_read(7'h45, narsel_rdata, narsel_busy, narsel_error);
     $display("narsel_rdata=0x%h, busy = 0x%h, error = 0x%h", narsel_rdata, narsel_busy, narsel_error);
     if(narsel_rdata !== 32'hdeadbeef)
        $error("NAR DDR write and read back data mismatch, expected 0x%h, received 0x%h", 32'hdeadbeef, narsel_rdata);
end

endmodule
