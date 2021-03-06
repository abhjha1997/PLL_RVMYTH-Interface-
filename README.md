# RVMYTH Integration with PLL:
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

## RYMYTH Simulation-   
PLL is the analog block so to integrate it creating the verilog code becomes a necessity, the functional verilog code is created for the PLL designed in the previous section- 


All the files are added to the test bench using include "file.v" in the testbench written for vsdminisoc. The testbench is run using "iverilog" and the dumped file is obtained.

![Screenshot 2022-03-20 120650](https://user-images.githubusercontent.com/97835399/159152280-9975d939-acf3-4534-8d5a-cbfedfccb0ee.png)


Now the waveform of the integrated PLL and RVMYTH we use gtkwave.



![image](https://user-images.githubusercontent.com/97835399/159637174-3c70157a-3df7-4d83-81d8-771d6e843748.png)

## PLL Modelling: 
Here we design the PLL as a clock multiplier and test the functionality. 
## Integration of PLL and rvmyth core:
Using the Clock Pin the the PLL and the RVMYTH is connected and the functionality of the code is verified by the testbench. The verilog code for the above PLL is given below:


                        `timescale 1ns / 1ps
                        module pll( CLK, VCO_IN, VDDA, VDDD, VSSA, VSSD, EN_VCO, REF);

                        input VSSD;
                        input EN_VCO;
                        input VSSA;
                        input VDDD;
                        input VDDA;
                        input VCO_IN;
                        output CLK;
                        input REF;
                        reg CLK;
                        real period, lastedge, refpd;
                        wire  VSSD, VSSA, VDDD, VDDA;
 

                        initial begin
                        lastedge = 0.0;
                        period = 25.0; // 25ns period = 40MHz
                        CLK <= 0;
                        end

                         // Toggle clock at rate determined by period
                        always @(CLK or EN_VCO) begin
                        if (EN_VCO == 1'b1) begin
                         #(period / 2.0);
                         CLK <= (CLK === 1'b0);
                        end else if (EN_VCO == 1'b0) begin
                        CLK <= 1'b0;
                        end else begin
                        CLK <= 1'bx;
                        end
                        end
   
                        // Update period on every reference rising edge
                        always @(posedge REF) begin
                        if (lastedge > 0.0) begin
                        refpd = $realtime - lastedge;
                        // Adjust period towards 1/8 the reference period
                         //period = (0.99 * period) + (0.01 * (refpd / 8.0));
                        period =  (refpd / 8.0) ;
                         end
                        lastedge = $realtime;
                        end
                        endmodule

The Testbench for the PLL is given below:
                                       
                       `timescale 1ns / 1ps
                        module pll_tb();
                        reg VSSD;
                        reg EN_VCO;
 
                        reg VSSA;
                        reg VDDD;
                        reg VDDA;
                        reg VCO_IN;
                        reg REF;
  
                        wire CLK;


                        pll dut(CLK, VCO_IN, VDDA, VDDD, VSSA, VSSD, EN_VCO, REF);
  
                        initial
                         begin
                        {REF,EN_VCO}=0;
                        VCO_IN = 1'b0 ;
                        VDDA = 1.8;
                        VDDD = 1.8;
                        VSSA = 0;
                        VSSD = 0;
   
                        end
   
                        initial
                        begin
                         $dumpfile("test.vcd");
                        $dumpvars(0,pll_tb);
                        end
 
                        initial
                        begin
                        repeat(400)
                        begin
                        EN_VCO = 1;
                        #100 REF = ~REF;
                        #(83.33/2)  VCO_IN = ~VCO_IN;
                        end
                        $finish;
                        end
                        endmodule
                        
                        
![image](https://user-images.githubusercontent.com/97835399/160181556-fc4d8d49-409f-4ddd-b09c-f58159fd980b.png)

                  
 This generates the .vcd file not functionality of the PLL can be checked by waveform using gtkwave.
 
 
![image](https://user-images.githubusercontent.com/97835399/160181691-e7f2aa48-4549-4804-85e1-320e17b274b2.png)

## Combining both PLL and rvmyth:
The next aim is to combine the PLL and RVMYTH core, top combine this we need to again write the verilog and corresponding test bench for the verilog file to check the functionality combined output.

Use the following command to analyze the waveform of the integrated PLL and RVMYTH using iverilog and gtkwave tools:

![image](https://user-images.githubusercontent.com/97835399/160224915-3e31d923-6d18-4010-bcfd-ac700e101211.png)

The output waveform of the RVMYTH_PLL is given below:


![image](https://user-images.githubusercontent.com/97835399/160224759-a3fe2fe0-a4b7-4cea-b2f2-701af4880849.png)

# Installlation of the openLANE and sky130 installation:

Synthesis in yosys

In OpenLANE the RTL synthesis, technology mapping and timing reports are performed by different tools- RTL synthesis by yosys, Technology mapping by abc.
Timing reports are generated by OpenSTA. Verilog file (.v) of the avsdpll and its LIB (.lib) file.

To generate the LIB file run the below perl script.

perl verilog_to_lib.pl avsdpll.v avsdpll

To perform synthesys in yosys

Just type yosys in linux shell and follow the script.


                  read_verilog rvmyth_pll.v 
                  read_liberty -lib avsd_pll_1v8.lib 
                  read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib 
                  synth -top rvmyth_pll_interface 
                  dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib 
                  opt 
                  abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D} 
                  flatten 
                  setundef -zero 
                  clean -purge 
                  rename -enumerate
                  stat 
                  write_verilog -noattr rvmyth_pll.synth.v 
                  The synthesized netlist is rvmyth_pll.synth.v
                  

The snapshot of the synthesized netlist.


![Screenshot from 2022-05-29 11-11-42 (1)](https://user-images.githubusercontent.com/97835399/170855345-d74031e6-58c7-4b7e-bb6c-eef9db010074.png)
## Pre-Synthesis simulation:
                                    iverilog rvmyth_pll.v rvmyth_pll_tb.v
                                    ./a.out
                                    gtkwave test1.vcd

![Screenshot from 2022-05-29 18-57-04](https://user-images.githubusercontent.com/97835399/170871496-39937bea-7c10-4026-baf1-4a527c9fac05.png)


## Post-Synthesis simulation
Use the following commands for post-synth simulation output.

                        iverilog gls.v 
                        ./a.out
                        gtkwave gls.vcd 
    
![Screenshot 2022-05-29 122450](https://user-images.githubusercontent.com/97835399/170856163-18d7b44e-d51f-456f-9e1e-8b300288f8bb.png)














































   
