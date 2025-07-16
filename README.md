# TCL-Workshop
VSD TCL Workshop 11th to 20th July 2025

Overview

This repository hosts the complete set of course materials, lab scripts, and practical exercises from the VSD TCL & Linux Workshop. It is designed to develop strong proficiency in TCL scripting and Linux-based automation tailored for digital semiconductor design flows.

The workshop content emphasizes automating design constraints, integrating TCL with open-source tools such as Yosys and OpenTimer, and conducting Quality of Results (QoR) analysis through custom TCL procedures.

Modulewise Content

Module 1: Introduction to TCL and VSDSYNTH Toolbox usage

1. TCL task and sub-task fundamentals
2. VSDSYNTH Toolbox scenarios and help flow
3. Handling user input and CSV formats

This is the .CSV file

<img width="743" height="685" alt="image" src="https://github.com/user-attachments/assets/02f71b79-da5b-429d-8cfb-9ad72e8c7bc1" />

+ Sub-Task One : VSDSYNTH Toolbox usage scenarios
Scenario 1 : User doesn't provide an input csv file.

-> Create command (ex: vsdsynth) and pass .csv from UNIX shell to TCL script
General scenarios : From user point of view.
1. Not provide .csv file as input
   <img width="928" height="60" alt="image" src="https://github.com/user-attachments/assets/f29b2767-ad15-4d06-9bfb-908c8ca43ce6" />
2. Provide a .csv file which doesn't exist
   <img width="922" height="57" alt="image" src="https://github.com/user-attachments/assets/bea89be3-ea51-4259-931d-1859703a9fca" />
3. Type "-help" to find out usage
   <img width="922" height="65" alt="image" src="https://github.com/user-attachments/assets/6f9a9cb3-1108-40db-8a1b-9d319a7001c1" />


The below code is used to check whether csv file is provided, does csv file provided by the user exist and the usage of the command.

<img width="1056" height="575" alt="image" src="https://github.com/user-attachments/assets/0207435c-fe20-4a39-9336-b307a18c3155" />

The output of the above code :

<img width="1596" height="672" alt="image" src="https://github.com/user-attachments/assets/f3eb90af-c3de-423e-a9da-1f0927cee6f7" />

Module 2: Variable Creation & Constraint Processing

In this module we will be converting all the inputs to format [1] & SDC format, and pass to synthesis tool 'Yosys'

-> Tasks involved to achieve this
1. Create variables
2. Check if directories and files mentioned in .csv exist or not
3. Read "Constraints File" for above .csv and convert to SDC format
4. Read all files in "Netlist Directory"
5. Create main synthesis script in format[2]
6. Pass this script to Yosys

Working with arrays, matrices, and loop constructs
Parsing and validating CSV/SDC constraint files Inside the openMSP430_design_constraints.csv file

<img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/adbf4dd7-0ad7-4003-a604-c0c6fddaf41c" />


vsdsynth.tcl (using the matric package and creating a matrix from the details.csv file.

<pre> ```set filename [lindex $argv 0]
package require csv
package require struct::matrix
struct::matrix m
set f [open $filename]
csv::read2matrix $f m , auto
close $f
set columns [m columns]
#m add columns $columns
m link my_arr
set num_of_rows [m rows] ``` </pre>

By converting the .csv file into a matrix, the number of rows and columns was determined, and variables for each entry were dynamically created using the my_arr structure.

<pre> ```set i 0
	while {$i < $num_of_rows} {
		puts "\nInfo: Setting $my_arr(0,$i) as '$my_arr(1,$i)'"
		if {$i == 0} {
			set [string map {" " ""} $my_arr(0,$i)] $my_arr(1,$i)
		} else {
			set [string map {" " ""} $my_arr(0,$i)] [file normalize $my_arr(1,$i)]
		}
		set i [expr {$i+1}]
	}
}

puts "\nInfo: Below are the list of initial variables and their values. User can use these variables for further debug. Use 'puts <variable name>' command to query value of below variables"
puts "DesignName = $DesignName"
puts "OutputDirectory = $OutputDirectory"
puts "NetlistDirectory = $NetlistDirectory"
puts "EarlyLibraryPath = $EarlyLibraryPath"
puts "LateLibraryPath = $LateLibraryPath"
puts "ConstraintsFile = $ConstraintsFile"``` </pre>

<img width="1292" height="773" alt="image" src="https://github.com/user-attachments/assets/e0a05458-bcff-48ef-97f1-831bf816dfe6" />

The names got converted into variables by removing spaces and paths got assigned to these variables.

<img width="772" height="560" alt="image" src="https://github.com/user-attachments/assets/18af75de-a415-43af-a0a8-af621d464365" />

This script verifies the existence of all required files and directories. If file or directory missing, the script throws an error message to prevent further execution. The only exception is the output directory, which is automatically created if it does not already exist.

<pre> ```if {![file isdirectory $OutputDirectory]} {
	puts "\nInfo: Cannot find output directory $OutputDirectory. Creating $OutputDirectory"
	file mkdir $OutputDirectory
} else {
	puts "\nInfo: Output directory found in path $OutputDirectory"
}
if {![file isdirectory $NetlistDirectory]} {
        puts "\nInfo: Cannot find RTL netlist directory $NetlistDirectory. Exiting..."
        exit
} else {
        puts "\nInfo: RTL Netlist directory found in path $NetlistDirectory"
}
if {![file exists $EarlyLibraryPath]} {
        puts "\nInfo: Cannot find early cell library in path $EarlyLibraryPath. Exiting..."
        exit
} else {
        puts "\nInfo: Early cell library found in path $EarlyLibraryPath"
}
if {![file exists $LateLibraryPath]} {
        puts "\nInfo: Cannot find late cell library in path $LateLibraryPath. Exiting..."
        exit
} else {
        puts "\nInfo: Late cell library found in path $LateLibraryPath"
}
if {![file exists $ConstraintsFile]} {
        puts "\nInfo: Cannot find constraints file in path $ConstraintsFile. Exiting..."
        exit
} else {
        puts "\nInfo: Contraints file found in path $ConstraintsFile"
}``` </pre>



<img width="591" height="412" alt="image" src="https://github.com/user-attachments/assets/22e5e33c-98c2-4879-aa3c-78232aa2c736" />

The file was successfully read and converted into a matrix structure. The script identified the total number of rows and columns, and determined the starting indices for clock, input, and output constraints. Below is the core TCL code used for processing

<pre> ```puts "\nInfo: Dumping SDC contraints file for $DesignName"
::struct::matrix constraints
set  chan [open $ConstraintsFile]
csv::read2matrix $chan constraints  , auto
close $chan
set number_of_rows [constraints rows]
puts "number_of_rows are $number_of_rows"
set number_of_columns [constraints columns]
puts "number_of_columns are $number_of_columns"
set clock_start [lindex [lindex [constraints search all CLOCKS] 0 ] 1]
puts "clock_start = $clock_start"
set clock_start_column [lindex [lindex [constraints search all CLOCKS] 0 ] 0]
puts "clock_start_column = $clock_start_column"
#----check row number for "inputs" section in constraints.csv------------#
set input_ports_start [lindex [lindex [constraints search all INPUTS] 0 ] 1]
puts "input_ports_start = $input_ports_start"
#----check row number for "inputs" section in constraints.csv------------#
set output_ports_start [lindex [lindex [constraints search all OUTPUTS] 0 ] 1]
puts "output_ports_start = $output_ports_start"``` </pre>

<img width="556" height="505" alt="image" src="https://github.com/user-attachments/assets/199a0e99-5dcf-4ce2-b28d-d8bdb88cae5f" />


