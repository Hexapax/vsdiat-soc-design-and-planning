## Converting Grid info to Track info

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