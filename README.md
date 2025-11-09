# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
#Aim
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment.
Apparatus Required
Vivado 2023.1
Procedure
1. Launch Vivado 2023.1
Open Vivado and create a new project.
2. Design the Verilog Code
Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench
Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files
Create both the design module and the testbench in the Vivado project.
5. Run Simulation
Run the behavioral simulation to verify the output.
6. Observe the Waveforms
Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results
Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# Code
# RAM
verilog code 
```
module ram (clk,we,addr,din,dout);
input clk;
input we;                 
input [11:0] addr;         
input [7:0] din;          
output reg [7:0] dout;     
reg [7:0] mem [0:4095];        

always @(posedge clk) 
begin
    if (we)
        mem[addr] <= din;      
        dout <= mem[addr];        
end
endmodule
```
TESTBENCH
```
module ram_tb;
reg clk;
reg we;
reg [11:0] addr;
reg [7:0] din;
wire [7:0] dout;
integer i;
ram dut (clk,we,addr,din,dout);

initial 
begin
    clk = 0;
    forever #5 clk = ~clk;
end

initial 
   begin
    we = 0;
    addr = 0;
    din = 0;
    #10;

       for (i = 0; i < 20; i = i + 1) 
        begin
        @(posedge clk);
        addr = $random % 4096;   // Random address (0-4095)
        din  = $random % 256;    // Random 8-bit data (0-255)
        we   = 1;
        @(posedge clk);
        we   = 0;
        end
$finish;
end
endmodule
```

Output Waveform

<img width="1919" height="1079" alt="Screenshot 2025-11-09 233636" src="https://github.com/user-attachments/assets/cf61e9cb-7d6f-4c93-b933-fb578b039c53" />


# ROM
 verilog code for ROM using $random
 ```
module rom(
input clk,rst,
input[9:0]address,
output reg [9:0]dout );
reg[7:0] rom[1023:0];

initial 
begin
rom[10'd1000]=8'd100;
rom[10'd1021]=8'd125;
rom[10'd1023]=8'd105;
end
always@(posedge clk)
begin
if(rst)
dout<=8'b0;
else
dout<=rom[address];
end
endmodule
```
TEST BENCH
```
module rom_tb;
reg clk_t,rst_t;
reg [9:0] address_t;
wire [7:0] dout_t;
rom dut(.clk(clk_t),.rst(rst_t),.address(address_t),.dout(dout_t));
initial
begin
clk_t=1'b0;
rst_t=1'b1;
address_t=10'd0;
#50 rst_t=1'b0;
address_t=10'd1000;
#100;
address_t=10'd1021;
#100
address_t=10'd1023;
#100;
$finish;
end
always #10 clk_t=~clk_t;
endmodule
 ```

output Waveform

<img width="1919" height="1079" alt="Screenshot 2025-11-09 233900" src="https://github.com/user-attachments/assets/d5069be3-a3ea-43b4-a21a-ca54b8b9bf21" />


 # FIFO
 verilog code for FIFO
 ```
module fifo(clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count);
input clk;
input rst;
input wr_en;
input [7:0] data_in;
input rd_en;
output reg full;
output reg [7:0] data_out;
output reg empty;
output reg [4:0] count;

reg [7:0] mem [0:15];
reg [3:0] wr_ptr;
reg [3:0] rd_ptr;

always @(posedge clk) 
   begin
        if (rst) 
          begin
            wr_ptr   <= 0;
            rd_ptr   <= 0;
            count    <= 0;
            data_out <= 0;
            full     <= 0;
            empty    <= 1;
          end 
      else 
        begin
            full  <= (count == 16);
            empty <= (count == 0);
            if (wr_en && !full) 
            begin
                mem[wr_ptr] <= data_in;
                wr_ptr <= wr_ptr + 1'b1;
            end
            if (rd_en && !empty) 
            begin
                    data_out <= mem[rd_ptr];
                    rd_ptr <= rd_ptr + 1'b1;
            end 

case ({wr_en && !full, rd_en && !empty})
    2'b10: count <= count + 1'b1;
    2'b01: count <= count - 1'b1;
    default: count <= count;
endcase
full  <= (count == 16);
            empty <= (count == 0);
        end
    end
endmodule
```
TEST BENCH
```

`timescale 1ns/1ps

module fifo_tb;
reg clk;
reg rst;
reg wr_en;
reg rd_en;
reg [7:0] data_in;
wire [7:0] data_out;
wire full;
wire empty;
wire [4:0] count;

fifo uut (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count );

always #5 clk = ~clk;

initial 
begin
clk = 0;
rst = 1;
wr_en = 0;
rd_en = 0;
data_in = 8'h00;            #10;
rst = 0;                          #10;
rst = 1;                          #10;
rst = 0;                         

repeat (5) 
begin
    @(posedge clk);
    wr_en = 1;
    data_in = data_in + 1;
end
@(posedge clk);
wr_en = 0;
repeat (3) 
  begin
    @(posedge clk);
    rd_en = 1;
end
@(posedge clk);
rd_en = 0;
#20;
$finish;
end

endmodule
``` 

 output Waveform

 <img width="1919" height="1079" alt="Screenshot 2025-11-09 234234" src="https://github.com/user-attachments/assets/326ce7f7-eafd-4814-a846-7393e0c7a0a9" />


# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

