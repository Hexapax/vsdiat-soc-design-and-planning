# Converting Grid info to Track info

### Find the tracks.info of sky130_fd_sc_hd and check your track dimensions

![](./Images/opentracks.png)

![](./Images/tracksinfo.png)

### Now inside the inverter file from before you can press G to show the grid and a small black dot should appear in the bottom left of the design

![](./Images/gridactivate.png)

### To align our grid to the correct size to match the track dimensions from earlier type this in the tkcon

![](./Images/gridupscale.png)

### Now the grid is the correct size and the inverter is 3 squares wide, verifying the conditions for the track (We assigned .46 to be the width of a grid cell and the width of the standard cell if you select it is 1.38 or three times that number)

![](./Images/largergrid.png)

### To practice creating a port with a label, select the A port, open this window and type in the following

![](./Images/createportlabel.png)

### You can also select the Y port and type in these commands to assign the port a class and use

![](./Images/yportassign.png)

### Now save this to a .lef file

![](./Images/writeleffile.png)

### You can see the port information is in the .lef file

![](./Images/openleffile.png)

### Now we copy the .lef file to src

![](./Images/copylef.png)

### Then inside the libs folder in the vsdstdcelldesigns also copy the lib files

```
# Copy lef file
cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# List and check whether it's copied
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# Copy lib files
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# List and check whether it's copied
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```


### Next we need to edit config.tcl

![](./Images/editconfig.png)

### Paste in these lines

```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

![](./Images/newconfig.png)

## Opening the new inverter in picorv32a

### Now we need to run openlane to see our new inverter in the picorv32a

![](./Images/runningopenlane.png)

### paste in a few lines after prepping to run picorv32a with the inverter and then run synthesis

```
# Adiitional commands to include newly added lef to openlane flow
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```

![](./Images/editedpicorv32arun.png)

### After running the synthesis we see that there is an issue with the slack and take note of these numbers and the size

![](./Images/slackissue.png)
![](./Images/picorvsizebefore.png)

### Now run these commands to view and edit the synthesis settings

```
# Command to display current value of variable SYNTH_STRATEGY
echo $::env(SYNTH_STRATEGY)

# Command to set new value for SYNTH_STRATEGY
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Command to display current value of variable SYNTH_BUFFERING to check whether it's enabled
echo $::env(SYNTH_BUFFERING)

# Command to display current value of variable SYNTH_SIZING
echo $::env(SYNTH_SIZING)

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Command to display current value of variable SYNTH_DRIVING_CELL to check whether it's the proper cell or not
echo $::env(SYNTH_DRIVING_CELL)
```

### Run synthesis again

![](./Images/slackfix.png)

### see that our slack issue has been resolved and our size has increased

![](./Images/reducedslack.png)
![](./Images/picorvsizeafter.png)

### When we run magic for the placement we can see the new cell in our picorv32a

![](./Images/magicvsdinv.png)

# Timing Analysis

## Central Concepts

* Timing Analysis of a circuit ensures it operates at the correct speed and minimizes delay
  * **Data Arrival Time** is when the data reaches the destination
  * **Data Required Time** is when the data **must** be stable at the destination for the correct capture
  * The **Slack** is the difference between these two 
    * Slack = Data Required Time - Data Arrival Time
    * You want to aim for positive slack to avoid logic delays
  * **Skew** is the difference in arrival time across different points in the circuit
    * Aim for 0 skew since too far positive or negative leads to an imbalance in operation of the circuit

![](./Images/timinganalysis.png)

### We will perform synthesis with the new lef

```
prep -design picorv32a

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_SIZING) 1

run_synthesis
```

### Create a new pre_sta.conf for STA analysis inside our main openlane directory

![](./Images/pre_staconf.png)

### also make a base.sdc file in picorv32a/src based on the base.sdc in openlane/scripts

![](./Images/basesdc.png)

### now from inside the openlane directory we will run sta

```
sta pre_sta.conf
```

![](./Images/runsta.png)

### Notice our sta shows lots of slack so we will start to optimize it

![](./Images/staslack.png)

### We can locate OR gates driving more fanouts and not enough strength and replace them with ones with higher strength

![](./Images/staissueexample.png)

### these commands replace our weak OR gate and perform analysis

```
# Reports all the connections to a net
report_net -connections _11672_

# Checking command syntax
help replace_cell

# Replacing cell
replace_cell _14513_ sky130_fd_sc_hd__or3_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

### And our slack has been slightly reduced

![](./Images/stalessslack.png)

### Now repeat this, looking for gates that cause an issue until you have significantly reduced slack
### Then replace your old netlist with the new one that has been updated. Make a copy of the synthesis.v file

```
cp picorv32a.synthesis.v picorv32a.synthesis_old.v
```

![](./Images/copysynthfile.png)

### Now back inside your STA run this command to write a new verilog file (replace the run name with your own)

```
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/27-07_00-26/results/synthesis/picorv32a.synthesis.v
```

### Now after running placement we can run the clock timing synthesis (CTS)

```
run_cts
```

![](./Images/runcts.png)

## OpenROAD timing analysis

### After running cts, run the following commands to use openroad (Remember to change run name)

```
# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/24-07_00-26/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/24-07_00-26/results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/24-07_00-26/results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/base.sdc

# Setting all clocks as propagated clocks
set_propagated_clock [all_clocks]

# Check syntax of 'report_checks' command
help report_checks

# Generating timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
```

![](./Images/openroadrun.png)

### Now we will try making changes to the variable CTS_CLK_BUFFER_LIST in order to remove the sky130_fd_sc_hd__clkbuf_1 cell

### Follow these commands to update the variable and run openROAD

```
# We will use this a lot to check the variables
echo $::env(CTS_CLK_BUFFER_LIST)

# Removing sky130_fd_sc_hd__clkbuf_1
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

echo $::env(CTS_CLK_BUFFER_LIST)

echo $::env(CURRENT_DEF)

# Setting def as placement def
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/24-07_00-26/results/placement/picorv32a.placement.def

run_cts

echo $::env(CTS_CLK_BUFFER_LIST)

openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/24-07_00-26/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/24-07_00-26/results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts1.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/24-07_00-26/results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all clocks as propagated clocks
set_propagated_clock [all_clocks]

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Report hold skew
report_clock_skew -hold

# Report setup skew
report_clock_skew -setup

exit

echo $::env(CTS_CLK_BUFFER_LIST)

# Inserts sky130_fd_sc_hd__clkbuf_1 to first index of list
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]

echo $::env(CTS_CLK_BUFFER_LIST)
```

![](./Images/openroadrun2.png)