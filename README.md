# PES_DUAL_PORT_RAM_DESIGN
<details>
<summary>RTL and GLS</summary>
  
+ Design Module
```
// Dual Port RAM module design

module pes_ram_design(
  input [7:0] data_a, data_b, //input data
  input [5:0] addr_a, addr_b, //Port A and Port B address
  input we_a, we_b, //write enable for Port A and Port B
  input clk, //clk
  output reg [7:0] q_a, q_b //output data at Port A and Port B
);
  
  reg [7:0] ram [63:0]; //8*64 bit ram

 
  always @ (posedge clk)
    begin
      if(we_a)
        ram[addr_a] <= data_a;
      else
        q_a <= ram[addr_a]; 
    end
  
  always @ (posedge clk)
    begin
      if(we_b)
        ram[addr_b] <= data_b;
      else
        q_b <= ram[addr_b]; 
    end
  
endmodule
```

+ Testbench 
```
// Dual Port RAM testbench

module dual_port_ram_tb;
  reg [7:0] data_a, data_b; //input data
  reg [5:0] addr_a, addr_b; //Port A and Port B address
  reg we_a, we_b; //write enable for Port A and Port B
  reg clk; //clk
  wire [7:0] q_a, q_b; //output data at Port A and Port B
  
  pes_ram_design dpr1(
    .data_a(data_a),
    .data_b(data_b),
    .addr_a(addr_a),
    .addr_b(addr_b),
    .we_a(we_a),
    .we_b(we_b),
    .clk(clk),
    .q_a(q_a),
    .q_b(q_b)
  );
  
  initial
    begin
      $dumpfile("dump.vcd");
      $dumpvars(1, dual_port_ram_tb);       
      
      clk=1'b1;
      forever #5 clk = ~clk;
    end
  
  initial
    begin
      data_a = 8'h33;
      addr_a = 6'h01;
      
      data_b = 8'h44;
      addr_b = 6'h02;
      
      we_a = 1'b1;
      we_b = 1'b1;
      
      #10;
      
      data_a = 8'h55;
      addr_a = 6'h03;
      
      addr_b = 6'h01;
      
      we_b = 1'b0;
      
      #10;          
            
      addr_a = 6'h02;
      
      addr_b = 6'h03;
      
      we_a = 1'b0;
      
      #10;
      
      addr_a = 6'h01;
      
      data_b = 8'h77;
      addr_b = 6'h02;
      
      we_b = 1'b1;
      
      #10;
    end
  
  initial	
    #40 $stop;
  
endmodule
```

## RTL Simulation
+ To simulate the HDL code before synthesis enter the following command
```
iverilog dual_port_design.v dual_port_tb.v
```
+ To generate the .vcd file type the following command
```
./a.out
```
+ To view the simulation waveform type the following command
```
gtkwave dump.vcd
```

![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/1cd39e48-6ae6-45b8-96f7-c261f9410e8d)

+ Pre-Synthesis Waveform looks like this
![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/3bafac69-209b-4d20-9c83-66956fb84486)


## Synthesis
+ Open yosys and read the .lib file. Then read the verilog file and synthesize the top module.
```
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dual_port_design.v
synth -top pes_ram_design
```

![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/9a5b331d-217a-4d67-8b04-14d25db91b01)

![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/0252b86b-be27-4661-9400-6a219429a0bd)

+ Perform the abc step by typing the following command
```
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```
![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/7c571612-0eda-483e-946d-8941aa8885b5)

+ To view the synthesized design type the following command
```
show
```

![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/08685658-49dc-4bed-bf21-f3fc8a6221c8)

![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/ef59bdcb-a54f-456b-a84d-bbc4fdcf42e2)

+ The following shows that the library files have been used.

![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/91a2e1e6-0751-4e70-bfa8-055979667639)

## GLS
+ The following shows netlist simulation
![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/90331fba-516a-43ae-8e27-7e882d12aed9)

The netlist simulation has some delay compared to the pre-synthesis simulation. However the final write results are the same.
![image](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/a9ac330c-4e57-4f9c-ab9f-dabe51a5cd87)

</details>

## Physical Design

### Installing magic and OpenLane
<details>
<summary>Prerequisites</summary> 
<blockhead>
  
+ Magic requires some pre requisites:
+ M4 preprocessor
```
sudo apt-get install m4
```
+ python 3
```
sudo apt-get install python
```
+ Xlib.h
```
sudo apt-get install libx11-dev
```
+ If you wish to have the Tcl/Tk wrapper around magic (recommended) you will need to install the Tcl/Tk libraries. Version 8.5 or higher is highly recommended.
+ Tcl/Tk
```
sudo apt-get install tcl-dev tk-dev
```
+ The best graphics for Magic is the OpenGL interface ("magic -d OGL"), but since that is problematic for off-screen rendering on many systems, a good alternative is the Cairo graphics interface ("magic -d XR"). This is optional, but if you want to use it, you need the Cairo library development package:

+ Cairo
```
sudo apt-get install libcairo2-dev 
```

+ The OpenGL interface itself may need these dependencies:

+ OpenGL
```
sudo apt-get install mesa-common-dev libglu1-mesa-dev
```

+ For the non-Tcl/Tk version only: The readline source makes reference to the `tputs` function which is provided by the ncurses library. Although the ncurses library is installed in Ubuntu, the include files to build against it are not, so the development version is required.

+ ncurses
```
sudo apt-get install libncurses-dev 
```
</blockhead>
</details>

<details>
<summary>Magic installation</summary>
<blockhead>
  
+ Next part is to clone from the magic repository. Magic requires writing into hidden folders which may sometimes require using root privileges. Therefore, before cloning the magic type the following:
```
sudo su
```
+ Now type in
```
git clone git://opencircuitdesign.com/magic
./configure
make
make install
```
</blockhead>  
</details>

<details><summary>Installing OpenLane</summary>
<blockhead>
  
+ OpenLane needs docker to run. So that needs to be installed first. Follow the steps given in the original documentation. It is very simple - https://openlane.readthedocs.io/en/latest/getting_started/installation/installation_ubuntu.html
</blockhead></details>

### Using OpenLane
<details>
<summary>Creating design directory</summary>
<blockhead>
  
+ Create your design folder through openlane. Change the <my_design_name> to the your verilog design name.
```
cd
cd OpenLane
make mount
./flow.tcl -design <my_design_name> -init_design_config
```
![Screenshot from 2023-11-04 22-07-32](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/4987c776-9abb-4d39-8ebf-d7650fd765ee)

+ A folder will be created in the openlane directory.
</blockhead>
</details>

<details>
<summary>Adding the design code</summary>
<blockhead>
  
+ Now we need to make our design code available in the source part which is the 'src' folder in our design file. This folder is not available initially. It needs to be created. So type the following in a new tab:
```
cd ~/OpenLane/openlane/<my_design_name>
mkdir src
cd src
gedit <design>.v
```
![Screenshot from 2023-11-04 22-05-23](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/20043bb0-64a5-4156-b941-e1b57a2ccb6a)

+ After opening the '.v' file, paste your design code in this. This will be used as your design file.
+ There will be a config.json file in your design directory. It will look like this after you add the path to the verilog design file for the field VERILOG_FILES like below:
```
dir::src/<design>.v
```
![Screenshot from 2023-11-04 22-34-44](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/51719195-f351-4c35-bfa5-2521e313ec37)
</blockhead>
</details>

### Running a basic automated flow of the Openlane

<details>
<summary>Running Automated Flow</summary>
<blockhead>
  
+ Type the following to start the automated openlane flow in the tab where openlane container was opened using docker
```
./flow.tcl -design openlane/<design_folder_name> -tag <name_for_a_specific_run>
```
![Screenshot from 2023-11-04 22-33-14](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/418614c6-ddd2-42fc-9966-647e524fecd7)

+ Make sure you change the 'pes_ram_design' to your design file name before executing the command. Also make sure you change the name of the openlane run before executing the command.
</blockhead>
</details>

<details>
<summary>Checking for negative slack</summary>
<blockhead>
  
+ After running the automated flow, check for slack in a file called '2-syn_sta.summary' or a similar sta summary file in the following location
```
/home/vishnu/OpenLane/openlane/pes_ram_design/runs/run_3_auto/reports/synthesis
```

+ As it can be seen from the sta summary, the slack is positive and therefore no timing violations. Therefore no need to do slack correction by replacing cells.

![Screenshot from 2023-11-04 22-41-22](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/aab54a4a-2fa8-4f7a-9723-85de10099221)

</details> 
</blockhead>

### Running the Interactive OpenLane

<details>
<summary>Starting Interactive Openlane</summary>
<blockhead>

+ To start the OpenLane in interactive mode:
```
./flow.tcl -interactive
```
</blockhead>
</details>

<details>
<summary>Prepare the design file for the flow</summary>
<blockhead>
  
+ Type the following command to prepare design code for the flow.
```
prep -design openlane/<design_folder_name> -tag <name_for_a_specific_run>
```
![Screenshot from 2023-11-04 22-44-42](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/0dc00922-db84-4043-b2c5-cc54e61c3b01)

</blockhead>
</details>

<details>
<summary>Synthesis</summary>
<blockhead>
  
+ Type the following to perform synthesis of the design.
```
run_synthesis
```
![Screenshot from 2023-11-04 22-45-41](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/8448f2ee-28d4-4de5-8236-7c390865bb5e)

+ To check for slack violations, check the end of a file called '2-sta.log' or a similar 'sta.log' in the followng location
```
/home/vishnu/OpenLane/openlane/pes_ram_design/runs/run_4_inter/logs/synthesis
```

+ As it can be seen from the sta log, slack is positive and therefore there are no timing violations.
![Screenshot from 2023-11-04 22-48-31](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/5c2da2f6-431d-4458-bc71-df23089f2c47)

</blockhead>
</details>

<details>
<summary>Floorplan</summary>
<blockhead>

  + Type the following to perform floorplan of the design
```
run_floorplan
```
![Screenshot from 2023-11-04 22-50-44](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/06f97712-4522-499c-9bc9-9e979b1dbc9b)

+ To view the design after floorplan we can use magic.
+ Open a new tab and type the following to go to the floorplan directory. Make sure you change the name of the design directory to your design directory.
```
cd /home/vishnu/OpenLane/openlane/pes_ram_design/runs/run_4_inter/results/floorplan
magic -T /home/vishnu/.volare/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read pes_ram_design.def &
```
+ This will open magic and show you the floorplan. Press 's' and then 'v' to set the design to the centre. Then keep pressing 'z' to zoom in.
![Screenshot from 2023-11-04 23-22-56](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/0ae2c811-4be7-412d-838c-e6c0e5ed9332)

![Screenshot from 2023-11-04 23-23-27](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/34f6085e-185a-40a2-8e8f-0b497266a8d4)

![Screenshot from 2023-11-04 23-24-33](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/470d44f9-a48b-428d-9b4e-8ad01ecc8615)

</blockhead>
</details>

<details>
<summary>Placement</summary>
<blockhead>

+ Placement - Type the following to perform placement of the design
```
run_placement
```
![Screenshot from 2023-11-04 23-29-08](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/cfc11cb5-b7cb-4882-8820-d9e5bae4e4c2)

+ To view the design after placement we can again use magic.
+ Open a new tab and type the following to go to the floorplan directory. Make sure you change the name of the design directory to your design directory.
```
cd /home/vishnu/OpenLane/openlane/pes_ram_design/runs/run_4_inter/results/placement
magic -T /home/vishnu/.volare/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read pes_ram_design.def &
```
![Screenshot from 2023-11-04 23-26-52](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/72c3711b-ebe4-4d96-acc7-74c599b54e32)

![Screenshot from 2023-11-04 23-28-17](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/fef5e3c6-d53a-4ac6-8d1c-c1806616e6fc)
  
</blockhead>
</details>

<details>
<summary>Clock Tree Synthesis</summary>
<blockhead>
  
+ Type the following to perform clock-tree-synthesis of the design
```
run_cts
```
![Screenshot from 2023-11-04 23-34-42](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/0f2a430b-e0c8-47f4-9434-f10cc6f196aa)

</blockhead></details>

<details>
<summary>Power Distribution Network</summary>
<blockhead>
  
+ To Generate Power Distribution Network, type the following
```
gen_pdn
```
![Screenshot from 2023-11-04 23-35-48](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/897caaa7-9104-460e-9eee-9652bfe282d3)
</blockhead></details>

<details>
<summary>Routing</summary>
<blockhead>
  
+ Type the following to perform routing of the design
```
run_routing
```
![Screenshot from 2023-11-04 23-54-06](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/8b79af8f-f658-4ea6-90bb-cef6a5506a89)

+ To view the design after routing we can again use magic.
+ Open a new tab and type the following to go to the routing directory. Make sure you change the name of the design directory to your design directory.
```
cd /home/vishnu/OpenLane/openlane/pes_ram_design/runs/run_4_inter/results/routing
magic -T /home/vishnu/.volare/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read pes_ram_design.def &
```
![Screenshot from 2023-11-04 23-55-35](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/6e07f62b-ac06-4958-96e7-6d39c2011f38)

![Screenshot from 2023-11-04 23-56-13](https://github.com/Vishnu1426/pes_ram_design/assets/79538653/f4bbf7ac-04fe-4bcb-a96a-b1acc08bc306)

</blockhead></details>
