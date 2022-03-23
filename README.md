# VSDminiSOC 
This Project deals with RISC-V based microprocessor and perform the physical design to get the SoC ready for tapeout, it consist of combining PLL with the rvmyth core and PLL is connected with the input clock to RVMYTH core. A testbench is created to test and analyze the working of the integrated subsystem.

The first step consist of converting the RISC-V Processor TL-Verilog to to verilog file using Sandpiper-saas tool.

The sandpiper-saas tool is installed using the following commands in the following link:
https://pypi.org/project/sandpiper-saas/
        

                                $ sudo apt install make python python3 python3-pip git iverilog gtkwave docker.io
                                $ sudo chmod 666 /var/run/docker.sock
                                $ cd ~
                                $ pip3 install pyyaml click sandpiper-saas

After installing the sandpiper-saas tool the use the folloeing command to convert into the verilog file the generated files are stored in the same directory.
                                        
                                $ sandpiper-saas -i rvmyth.tlv -o rvmyth.v --bestsv --noline -p verilog 
     
This will generate the required HDL file, two files generated are 
![image](https://user-images.githubusercontent.com/97835399/159148758-adbd68cc-41ca-4938-8b67-8bc2a8ca9c4a.png)

## PLL Modelling-  
PLL is the analog block so to integrate it creating the verilog code becomes a necessity, the functional verilog code is created for the PLL designed in the previous section- 

                        
                                        module pll1 (
                                        output reg  CLK,
                                        input  wire VCO_IN,
                                        input  wire ENb_CP,
                                        input  wire ENb_VCO,
                                        input  wire REF
                                        );
                                         real period, lastedge, refpd;

                                         initial begin
                                         lastedge = 0.0;
                                        period = 25.0; // 25ns period = 40MHz
                                         CLK <= 0;
                                         end

                                        // Toggle clock at rate determined by period
                                        always @(CLK or ENb_VCO) begin
                                        if (ENb_VCO == 1'b1) begin
                                        #(period / 2.0);
                                        CLK <= (CLK === 1'b0);
                                        end
                                        else if (ENb_VCO == 1'b0) begin
                                        CLK <= 1'b0;
                                         end 
                                         else begin
                                         CLK <= 1'bx;
                                         end
                                         end
   
                                        // Update period on every reference rising edge
                                        always @(posedge REF) begin
                                        if (lastedge > 0.0) begin
                                        refpd = $realtime - lastedge;
                                          period =  (refpd / 8.0) ;
                                        end
                                        lastedge = $realtime;
                                         end
                                        endmodule


## Integration of PLL and rvmyth core:
Using the Clock Pin the the PLL and the RVMYTH is connected and the functionality of the code is verified by the testbench.
                                      
                                       
                                module vsdminisoc (
                                output wire OUT,
                                input  wire reset,
                                input  wire VCO_IN,
                                input  wire ENb_CP,
                                input  wire ENb_VCO,
                                input  wire REF,
                                //input  wire VREFL,
                                input  wire VREFH
                                 );

                                wire CLK;
                                wire [9:0] RV_TO_DAC;
   
                                 rvmyth core (
                                 .OUT(RV_TO_DAC),
                                 .CLK(CLK),
                                .reset(reset)
                                 );

                               pll1 pll (
                                .CLK(CLK),
                                .VCO_IN(VCO_IN),
                                .ENb_CP(ENb_CP),
                                .ENb_VCO(ENb_VCO),
                                 .REF(REF)
                                  );  
                                  endmodule

All the files are added to the test bench using include "file.v" in the testbench written for vsdminisoc. The testbench is run using "iverilog" and the dumped file is obtained.

![Screenshot 2022-03-20 120650](https://user-images.githubusercontent.com/97835399/159152280-9975d939-acf3-4534-8d5a-cbfedfccb0ee.png)


Now the waveform of the integrated PLL and RVMYTH we use gtkwave.

![image](https://user-images.githubusercontent.com/97835399/159637174-3c70157a-3df7-4d83-81d8-771d6e843748.png)

## PLL Modelling: 
Here we design the PLL as a clock multiplier and test the functionality. 



























   
