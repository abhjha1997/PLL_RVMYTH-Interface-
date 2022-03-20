# VSDminiSOC 
This Project deals with RISC-V based microprocessor and perform the physical design to get the Soc ready for tapeout, it consist of combining PLL with the rvmyth core and PLL is connected with the input clock to RVMYTH core. A testbench is created to test and analyze the working of the integrated subsystem.

The first step consist of converting the RISC-V Processor TL-Verilog to to verilog file using Sandpiper-saas tool.

The sandpiper-saas tool is installed using the following commands in the following link:
https://pypi.org/project/sandpiper-saas/
        

                                $ sudo apt install make python python3 python3-pip git iverilog gtkwave docker.io
                                $ sudo chmod 666 /var/run/docker.sock
                                $ cd ~
                                $ pip3 install pyyaml click sandpiper-saas

After installing the sandpiper-saas tool the use the folloeing command to convert into the verilog file the generated files are stored in the same directory.
                                        
                                $ sandpiper-saas -i rvmyth.tlv -o rvmyth.sv --bestsv --noline -p verilog 
     
This will generate the required HDL file.

   
