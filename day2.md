## Floorplanning

After running synthesis we can use the following command to create a floorplan of picorv32a

  * 
    ```tcl
    run_floorplan
    ```
![](./Images/floorplanrun.png)

We can then check the info on our floorplan run inside the config file

![](./Images/floorplaninfo.png)

After navigating to the run file of the floorplan we can use this command to open magic inside this file

  * 
    ```tcl
    magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
    ```

![](./Images/magicfloorplan.png)

Using s to select the floorplan, v to center ourselves, and then z and Shift+z to zoom in and out; We can zoom in on the individual cells

![](./Images/magiccloseuptapcells.png)

Then navigating to the bottom left corner we can find our preplaced cells

![](./Images/preplacedcells.png)