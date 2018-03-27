# Understanding how density (utilization) is calculated by the different commands in Innovus

## Problem

How is density (utilization) calculated by the various commands in Innovus?


## Solution

### Commands

The commands typically used to report density are:

timeDesign
optDesign
checkPlace
checkFPlan -reportUtil
reportDensityMap
queryDensityInBox
summaryReport
report_qor (Common UI)

The following is an explanation of how density is calculated along with examples and exceptions for each command.

### Density Calculation
 
The basic density calculation for the commands mentioned earlier is:

stdcell_area / alloc_area

Where:

stdcell_area = sum of the standard cell area of instances in the design
alloc_area = total area available for the standard cells to be placed (that is, area of the sites that are unblocked)

There are several factors that effect these two values:

Cell padding (specifyCellPad) is added to the stdcell_area value.
Stripes on layers specified as obstructions (setPlaceMode -prerouteAsObs) are treated as placement blockages. So, any sites blocked by these are subtracted from alloc_area.
Sites blocked by hard placement blockages are not counted in alloc_area.
The area of hard macros or standard cells with a placement status of FIXED or COVER are not included in stdcell_area or alloc_area.
Block halos are included in the hard macro area if their status is FIXED or COVER.
If a cell has a PLACED status, its area is counted in stdcell_area as well as in alloc_area, even if it is a hard macro.
The area of the physical cells, such as filler cells, are included in stdcell_area. Some commands such as timeDesign report the density with and without the physical-only cells.

### Command Details and Examples
 
**optDesign, timeDesign and placement**
timeDesign and optDesign (including place_opt_design) report the density after the timing summary table:

Density: 49.716%

placeDesign reports the density in a similar way, except that it may reduce the alloc_area further. For example, if setPlaceMode -maxDensity value is set, alloc_area is reduced by this set value (alloc_area = alloc_area * value). Also, placeDesign removes the small channels from the alloc_area calculation. Overall, placeDesign uses a complex, internal algorithm to calculate the density and therefore, you should rely on checkFPlan -reportUtil instead.

c**heckPlace**
checkPlace reports the density with and without the fixed standard cells. 120 is a standard cell area and 241 is a placeable area.

Placement Density:49.72%(120/241)
Placement Density (including fixed std cells):49.72%(120/241)

**checkFPlan -reportUtil and report_qor**
checkFPlan -reportUtil reports the core utilization and effective utilization. This replaces the queryPlaceDensity command in Innovus 15.2 and later.

Core utilization is the basic utilization without considering blocked sites. Density for the design is the effective density considering block sites, cell padding, and so on.

One important difference is that checkFplan -reportUtil does not include the soft blockage area in alloc_area. Likewise, a partial placement blockage subtracts the percentage of area blocked from alloc_area.

Core utilization  = 49.715909
Effective Utilizations
Average module density = 0.497.
Density for the design = 0.497.
       = stdcell_area 350 sites (120 um^2) / alloc_area 704 sites (241 um^2).

The density value reported by report_qor, when using the common UI, is equal to the effective utilization reported by checkFPlan -reportUtil.


Note:
The density reported by command place_opt_design at the begin of global placement might be higher than the one that is reported in the timing summary or that is reported by command  “checkFPlan –reportUtil".

Begin of global placement:
…
Density for the design = 0.547.
       = stdcell_area 4770581 sites (64117 um^2) / alloc_area 8717962 sites (117169 um^2).
…
Iteration  1: Total net bbox = 1.133e+04 (9.42e+03 1.92e+03)
              Est.  stn bbox = 1.596e+04 (1.35e+04 2.49e+03)
              cpu = 0:00:05.2 real = 0:00:05.0 mem = 9848.3M
…

Density, later reported in the timing summary:
Density: 54.362%

This difference can be caused be application of auto density screens. Those auto density screens are not visible to the user and are generated automatically by place_opt_design as a method to address congestion problems. They will increase the density number that is reported right before global placement.

You can review the impact of auto density screens in the verbose log file (e.g. innovus.logv). There, this information can be found - example:

[12/17 16:52:44    199s] Apply auto density screen in pre-place stage.
[12/17 16:52:46    201s] Auto density screen increases utilization from 0.544 to 0.547
[12/17 16:52:46    201s] Auto density screen runtime: cpu = 0:00:02.0 real = 0:00:02.0 mem = 9620.2M

 

**queryDensityInBox**
queryDensityInBox reports the density for a specific area.

StdInstArea/freeSpace = 49.7%(119.70/240.77)
macroInstArea/totArea = 0.0%(0.00/240.77)
powerMetalArea/totArea = 0.0%(0.00/240.77)
PlacementObsArea/totArea = 0.0%(0.00/240.77)
0.4972

Where:

StdInstArea = area_of_standard_cells + area_of_physical_cells (filler, Endcap, welltaps, power shutoff (PSO) and Decaps cells)
freeSpace = core_area - blocked_core_area
Note: blocked_core_area is the area blocked by the hard macros, power stripes (determined by setPlaceMode -prerouteAsObs setting), and placement blockages.

IMPORTANT: FIXED cells and cell padding do not effect the calculation.

macroInstArea = area_of_hard_macros
totArea = total_core_area
powerMetalArea = area_of_power_stripes​
Note: setPlaceMode -prerouteAsObs is used to determine if the power stripes block the cells from being placed under them.

PlacementObsArea = area_of_placement_blockages

**​reportDensityMap**
​This generates a map that uses colors to represent the placement density and also generates a placement density report. By default, reportDensityMap does not take into account the fixed instances while queryDensityInBox does. To let reportDensityMap take into account the fixed std cells (but not the macros), you need to run the following before the actual command.

setPlaceMode -includeFixed true

This should result in similar placement density reports between reportDensityMap and queryDensityInBox.

Note: The text report from reportDensityMap is only useful for reporting the number of hotspots. You should use queryPlaceDensity in conjunction with the GUI to get a real sense of where the density is a problem.

**summaryReport**
summaryReport provides a detailed breakdown of the density.

    ==============================
    Floorplan/Placement Information
    ==============================
    Total area of Standard cells: 119.700 um^2
    Total area of Standard cells(Subtracting Physical Cells): 119.700 um^2
    Total area of Macros: 0.000 um^2
    Total area of Blockages: 0.000 um^2
    Total area of Pad cells: 0.000 um^2
    Total area of Core: 240.768 um^2
    Total area of Chip: 240.768 um^2
    Effective Utilization: 4.9716e-01
    Number of Cell Rows: 8
    % Pure Gate Density #1 (Subtracting BLOCKAGES): 49.716%
    % Pure Gate Density #2 (Subtracting BLOCKAGES and Physical Cells): 49.716%
    % Pure Gate Density #3 (Subtracting MACROS): 49.716%
    % Pure Gate Density #4 (Subtracting MACROS and Physical Cells): 49.716%
    % Pure Gate Density #5 (Subtracting MACROS and BLOCKAGES): 49.716%
    % Pure Gate Density #6 ((Unpreplaced Standard Inst + Unpreplaced Block Inst + Unpreplaced Black Blob Inst + Fixed Clock Inst Area) / (Free Site Area + Fixed Clock Inst Area) for insts are placed): 49.716%
    % Core Density (Counting Std Cells and MACROs): 49.716%
    % Core Density #2(Subtracting Physical Cells): 49.716%
    % Chip Density (Counting Std Cells and MACROs and IOs): 49.716%
    % Chip Density #2(Subtracting Physical Cells): 49.716%