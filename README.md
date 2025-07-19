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


Module 3 : Clock and input constraint scripting

In this module we will convert design constraints from .csv file to corresponding SDC (Synopsys Design Constraints) commands. This is done with the matrix-based search algorithm and differentiated between bus signals and individual bits. This process involves giving the .csv file as an input and extracting clock and input related data whether input type is bus or bit and converting it into SDC commands. For the clock-related constraints, the relevant entries were accurately extracted from the .csv file, and suitable SDC commands were generated and dumped into the output .sdc file. Classifying input ports using regular expressions.


<pre> ```#----------------------clock constraints-------------------------------------#
#----------------------clock latency constraints-----------------------------#

set clock_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_rise_delay] 0] 0]
set clock_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_fall_delay] 0] 0]
set clock_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_rise_delay] 0] 0]
set clock_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_fall_delay] 0] 0]
#----------------clock transition contraints---------------------------------#
set clock_early_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_rise_slew] 0] 0]
set clock_early_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_fall_slew] 0] 0]
set clock_late_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_rise_slew] 0] 0]
set clock_late_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_fall_slew] 0] 0]
set sdc_file [open $OutputDirectory/$DesignName.sdc "w"]
set i [expr {$clock_start+1}]
set end_of_ports [expr {$input_ports_start-1}]
puts "\nInfo-SDC: Working on clock constraints....."
while {$i < $end_of_ports} {
	puts "Working on clock [constraints get cell 0 $i]"
	puts -nonewline $sdc_file "\ncreate_clock -name [constraints get cell 0 $i] -period [constraints get cell 1 $i] -waveform \{0 [expr {[constraints get cell 1 $i]*[constraints get cell 2 $i]/100}]\} \[get_ports [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -rise -min  [constraints get cell $clock_early_rise_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -fall -min  [constraints get cell $clock_early_fall_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -rise -max  [constraints get cell $clock_late_rise_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -fall -max  [constraints get cell $clock_late_fall_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -rise  [constraints get cell $clock_early_rise_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -fall  [constraints get cell $clock_early_fall_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -rise  [constraints get cell $clock_late_rise_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -fall  [constraints get cell $clock_late_fall_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	
	set i [expr {$i+1}]
}``` </pre>


<img width="1587" height="741" alt="image" src="https://github.com/user-attachments/assets/519d0a47-c811-4381-a316-69e9c2d9f49e" />

<img width="767" height="356" alt="image" src="https://github.com/user-attachments/assets/c53d84ea-a84a-4322-a97b-1947c1d94df7" />

Successfully converted .csv file SDC format.


<pre> ```#--------------------input constraints-------------------------------#
set input_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_rise_delay] 0] 0]
set input_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_fall_delay] 0] 0]
set input_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_rise_delay] 0] 0]
set input_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_fall_delay] 0] 0]

set input_early_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_rise_slew] 0] 0]
set input_early_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_fall_slew] 0] 0]
set input_late_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_rise_slew] 0] 0]
set input_late_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_fall_slew] 0] 0]

set related_clock [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] clocks] 0 ] 0]
set i [expr {$input_ports_start+1}]
set end_of_ports [expr {$output_ports_start-1}]
puts "\nInfo-SDC: WOrking on IO constraints......."
puts "\nInfo-SDC: Categorizing input ports as bits and bussed"

while {$i < $end_of_ports} {
#------------------------------------optional----------differentiating input ports as bussed and bits-------------------#
set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]
foreach f $netlist {
	set fd [open $f]
	puts "reading file $f"
	while {[gets $fd line] != -1} {
		set pattern1 " [constraints get cell 0 $i];"
		if {[regexp -all -- $pattern1 $line]} {
			puts "pattern1 \"$pattern1\" found and matching in verilog file \"$f\" is \"$line\""
			set pattern2 [lindex [split $line ";"] 0]
			puts "creatng pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
			if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
				puts "out of all patterns, \"$pattern2\"has matching string \"input\". Sp preserving this line and ignoring others"
				set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
				puts "printing first 3 elements of pattern2 as \"$s1\" using space as delimiter"
				puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
				puts "replace mutiple spaces in s1 by single space and reformant as \"[regsub -all {\s+} $s1 " "]\""
			        }
			}
		
		}
	

close $fd
}
close $tmp_file

set tmp_file [open /tmp/1 r]
#puts "reading [read $tmp_file]"
#puts "reading /tmp/1 file as [split [read $tmp_file] \n]"
#puts "sorting /tmp/1 contents as [lsort -unique [split [read $tmp_file] \n ]]"
#puts "joining /tmp/1 as [join [lsort -unique [split [read $tmp_file] \n ]] \n]"
set tmp2_file [open /tmp/2 w]
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file

set tmp2_file [open /tmp/2 r]
#puts "Count is [llength [read $tmp2_file]]"
set count [llength [read $tmp2_file]]
#puts "Splitting content of tmp_2 using space and counting number of elements as $count"

if {$count > 2} {
	set inp_ports [concat [constraints get cell 0 $i]*]
	puts "bussed"
} else {
	set inp_ports [constraints get cell 0 $i]
	puts "not bussed"
}
	puts "input port name is $inp_ports since count is $count\n"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $input_early_fall_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $input_late_rise_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $input_late_fall_delay_start $i] \[get_ports $inp_ports\]"

	puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $input_early_fall_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $input_late_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $input_late_fall_slew_start $i] \[get_ports $inp_ports\]"

	set i [expr {$i+1}]
}
close $tmp2_file``` </pre>


<img width="1153" height="700" alt="image" src="https://github.com/user-attachments/assets/fce9431c-f993-4a70-88bd-479565475dab" />


<img width="1253" height="497" alt="image" src="https://github.com/user-attachments/assets/9e499d18-5efd-4017-86e0-a468afa50003" />


<img width="767" height="572" alt="image" src="https://github.com/user-attachments/assets/64715156-e5e2-4d9f-9646-9deb36394e86" />



Module 4: RTL Synthesis & Yosys Integration

In this module processing the output section of the constraints .csv file and generating the corresponding SDC commands, performing a sample synthesis using Yosys with an example memory block, checking the hierarchy in Yosys, and implementing error handling for hierarchy-related issues. This involves giving the .csv file to extract output-related constraints, differentiating between bit-level and bus-type outputs, and generating the SDC commands in the .sdc file. A Yosys script is developed to perform the hierarchy check, and error-handling code is also implemented to identify and manage hierarchy-related issues during synthesis. 


<pre> ```set output_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] early_rise_delay] 0] 0]
set output_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] early_fall_delay] 0] 0]
set output_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] late_rise_delay] 0] 0]
set output_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] late_fall_delay] 0] 0]
set output_load_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] load] 0 ] 0]

set related_clock [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] clocks] 0 ] 0]
set i [expr {$output_ports_start+1}]
set end_of_ports [expr {$number_of_rows-1}]
puts "\nInfo-SDC: Working on IO constraints......."
puts "\nInfo-SDC: Categorizing output ports as bits and bussed"

while {$i < $end_of_ports} {
set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]
foreach f $netlist {
        set fd [open $f]
        #puts "reading file $f"
        while {[gets $fd line] != -1} {
                set pattern1 " [constraints get cell 0 $i];"
                if {[regexp -all -- $pattern1 $line]} {
                        #puts "pattern1 \"$pattern1\" found and matching in verilog file \"$f\" is \"$line\""
                        set pattern2 [lindex [split $line ";"] 0]
                        #puts "creatng pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
                        if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
                                #puts "out of all patterns, \"$pattern2\"has matching string \"input\". Sp preserving this line and ignoring others"
                                set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
                                #puts "printing first 3 elements of pattern2 as \"$s1\" using space as delimiter"
                                puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
                                #puts "replace mutiple spaces in s1 by single space and reformant as \"[regsub -all {\s+} $s1 " "]\""
                                }
                        }

                }
close $fd
}
close $tmp_file

set tmp_file [open /tmp/1 r]
#puts "reading [read $tmp_file]"
#puts "reading /tmp/1 file as [split [read $tmp_file] \n]"
#puts "sorting /tmp/1 contents as [lsort -unique [split [read $tmp_file] \n ]]"
#puts "joining /tmp/1 as [join [lsort -unique [split [read $tmp_file] \n ]] \n]"
set tmp2_file [open /tmp/2 w]
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file

set tmp2_file [open /tmp/2 r]
#puts "Count is [llength [read $tmp2_file]]"
set count [llength [read $tmp2_file]]
#puts "Splitting content of tmp_2 using space and counting number of elements as $count"

if {$count > 2} {
        set op_ports [concat [constraints get cell 0 $i]*]
        #puts "bussed"
} else {
        set op_ports [constraints get cell 0 $i]
        #puts "not bussed"
}
        #puts "output port name is $op_ports since count is $count\n"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $output_early_rise_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $output_early_fall_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $output_late_rise_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $output_late_fall_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_load [constraints get cell $output_load_start $i] \[get_ports $op_ports\]"

        set i [expr {$i+1}]

}

close $tmp2_file
#return
close $sdc_file

puts "\nInfo: SDC created. PLease use constraints in path $OutputDirectory/$DesignName.sdc"``` </pre>

<img width="731" height="543" alt="image" src="https://github.com/user-attachments/assets/cbecdb03-7ddb-4aef-b0b2-7be974971540" />

The /tmp/1 and /tmp/2 files will have similar formats for both input and output ports.

<pre> ```module memory (CLK, ADDR, DIN, DOUT);

parameter wordSize = 1;
parameter addressSize = 1;

input ADDR, CLK;
input [wordSize-1:0] DIN;
output reg [wordSize-1:0] DOUT;
reg [wordSize:0] mem [0:(1<<addressSize)-1];

always @(posedge CLK) begin
	mem[ADDR] <= DIN;
	DOUT <= mem[ADDR];
	end

endmodule``` </pre>


The basic Yosys script memory.ys, used to synthesize the design and generate both the gate-level netlist and a 2D gate-level representation of the memory module, is provided below.

<pre> ```read_liberty -lib -ignore_miss_dir -setattr blackbox /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
read_verilog memory.v
synth top memory
splitnets -ports -format ___
dfflibmap -liberty /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
opt
abc -liberty /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
flatten
clean -purge
opt
clean
write_verilog memory_synth.v``` </pre>

The output view of netlist from the code is shown below.

<img width="950" height="537" alt="image" src="https://github.com/user-attachments/assets/04360f42-5597-40ce-af4e-50106472c680" />

memory write process is illustrated in the following images using a truth table

<img width="947" height="691" alt="image" src="https://github.com/user-attachments/assets/eeb89325-e917-460b-9f4c-4e275f82beac" />

first rising edge of the clock

<img width="948" height="536" alt="image" src="https://github.com/user-attachments/assets/91d4a00e-2954-4c0b-a90e-2a915dc1c4b9" />

first rising edge of the clock - write process done

<img width="945" height="526" alt="image" src="https://github.com/user-attachments/assets/c57e7b13-e79d-496a-b0e2-7dcccf7cdb8b" />

memory read process is demonstrated in the following images through a truth table

<img width="952" height="682" alt="image" src="https://github.com/user-attachments/assets/3e6a328d-1f6b-40ce-8999-8a3495ae53d0" />

After first rising edge and before second rising edge of the clock

<img width="940" height="533" alt="image" src="https://github.com/user-attachments/assets/6314eecf-c210-446e-811c-a1a0ee54214d" />

After second rising edge of the clock - read process done

<img width="952" height="523" alt="image" src="https://github.com/user-attachments/assets/28ccae59-d39d-486f-9f83-02f513f457b9" />

The hierarchy check script was successfully implemented. The developed code generates the required script for performing hierarchy verification.

<pre> ```#Hierarchy Check
puts "\nInfo: Creating hierarchy check script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
puts "data is \"$data\""
set filename "$DesignName.hier.ys"
puts "Filename is \"$filename\""
set fileId [open $OutputDirectory/$filename "w"]
puts -nonewline $fileId $dat
set netlist [glob -dir $NetlistDirectory *.v]
puts "Netlist is \"$netlist\""
foreach f $netlist {
	set data $f
	puts "Data is \"$f\""	
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -check"
close $fileId``` </pre>

<img width="1035" height="493" alt="image" src="https://github.com/user-attachments/assets/d2cc69c3-5ed4-4257-b354-2ef1a9074762" />

<img width="1041" height="321" alt="image" src="https://github.com/user-attachments/assets/5330c64b-0520-48e4-9e12-3229e855304a" />

Hierarchy Check and Error Handling is done in Yosys. The script detects and respond to any errors that occur during the hierarchy check.

<pre> ```set my_err [catch {exec yosys -s $OutputDirectory/$DesignName.hier.ys >& $OutputDirectory/$DesignName.hierarchy_check.log} msg]
puts "error flag is $my_err"
if { $my_err } {
set filename "$OutputDirectory/$DesignName.hierarchy_check.log"
	puts "Log file name is $filename"
	set pattern {referenced in module}
	puts "Pattern is $pattern"
	set count 0
	set fid [open $filename r]
	while {[gets $fid line] != -1} {
		incr count [regexp -all -- $pattern $line]
		if {[regexp -all -- $pattern $line]} {
			puts "\nError: Module [lindex $line 2] is not part of design $DesignName. Please correct RTL in the path '$NetlistDirectory'"
			puts "\nInfo: Hierarchy check has FAILED"
		}
	}
	close $fid
} else {
	puts "\nInfo: Hierarchy check is PASS"
}
puts "\nInfo: PLease find hierarchy check details in [file normalize $OutputDirectory/$DesignName.hierarchy_check.log] for more info"
cd $working_dir``` </pre>

<img width="1037" height="506" alt="image" src="https://github.com/user-attachments/assets/8c9392a4-bffa-41a0-b1e6-f0c05c8ebf3d" />

<img width="1036" height="505" alt="image" src="https://github.com/user-attachments/assets/ea47d506-2770-406a-b69f-495e468b0092" />



Module 5: QOR Report Generation

Using Yosys tool I have executed the main synthesis flow in module5, understood the working of the proc. Generating essential files like .conf, .spef, and .timing and complete opentimer script performing a static timing analysis (STA) run and extracting relevant Quality of Results (QoR) data from the .results file. This includes writing and dumping the main Yosys synthesis script (.ys file), creating the necessary OpenTimer input files, executing the STA run, and parsing the results to present QoR data in the expected format. 


<pre> ```# Synthesis Script
puts "\nInfo: Creating main synthesis script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
set filename "$DesignName.ys"
set fileId [open $OutputDirectory/$filename "w"]
puts -nonewline $fileId $data

set netlist [glob -dir $NetlistDirectory *.v]
foreach f $netlist {
	set data $f
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -top $DesignName"
puts -nonewline $fileId "\nsynth -top $DesignName"
puts -nonewline $fileId "\nsplitnets -ports -format ___\ndfflibmap -liberty ${LateLibraryPath}\nopt"
puts -nonewline $fileId "\nabc -liberty ${LateLibraryPath}"
puts -nonewline $fileId "\nflatten"
puts -nonewline $fileId "\nclean -purge\niopadmap -outpad BUFX2 A:Y -bits\nopt\nclean"
puts -nonewline $fileId "\nwrite_verilog $OutputDirectory/$DesignName.synth.v"
close $fileId
puts "\nInfo: Synthesis script created and can be accessed from path $OutputDirectory/$DesignName.ys"

puts "\nInfo: Running synthesis................"``` </pre>


<img width="1035" height="481" alt="image" src="https://github.com/user-attachments/assets/ceb3ee58-9787-406d-8a94-b3e8ec5aaf6c" />

<img width="1036" height="426" alt="image" src="https://github.com/user-attachments/assets/0434f4ce-511a-4d50-be18-babbf2d4f214" />

Yosys synthesis script has been successfully implemented, including error handling to terminate the process if any issues are encountered during execution.


<pre> ```if {[catch {exec yosys -s $OutputDirectory/$DesignName.ys >& $OutputDirectory/$DesignName.synthesis.log} msg]} {
	puts "\nError: Synthesis failed due to errors. Please refer to log $OutputDirectory/$DesignName.synthesis.log for errors"
	exit
} else {
	puts "\nInfo: Synthesis finished successfully"
}
puts "Please refer to log $OutputDirectory/$DesignName.synthesis.log"``` </pre>


<img width="1037" height="190" alt="image" src="https://github.com/user-attachments/assets/a21d8a1f-ad7e-44ea-b482-4faf5a9085f8" />

<img width="1035" height="491" alt="image" src="https://github.com/user-attachments/assets/9fb11160-0c48-4448-917d-6c3a6168bb6a" />


Editing the synth.v file where we have "*" and "\"

<pre> ```#Editing synth.v to be usable by Opentimer
set fileId [open /tmp/1 "w"]
puts -nonewline $fileId [exec grep -v -w "*" $OutputDirectory/$DesignName.synth.v]
close $fileId

set output [open $OutputDirectory/$DesignName.final.synth.v "w"]

set filename "/tmp/1"
set fid [open $filename r]
	while {[gets $fid line] != -1} {
		puts -nonewline $output [string map {"\\" ""} $line]
		puts -nonewline $output "\n"
	}
	close $fid
	close $output

	puts "\nInfo: Please find the synthesized netlist for $DesignName at below path. You can use this netlist for STA or PNR"
puts "\n$OutputDirectory/$DesignName.final.synth.v"``` </pre>


<img width="1038" height="503" alt="image" src="https://github.com/user-attachments/assets/2c035cda-d46e-4e8d-9a16-3ffc25cc9b96" />

<img width="1045" height="513" alt="image" src="https://github.com/user-attachments/assets/74c2b8c7-df46-495f-968b-28afbd4102bd" />

Started using proc methods to define custom commands for enhancing script flexibility and reusability.

<pre> ```#!/bin/tclsh
proc reopenStdout {file} {
    close stdout
    open $file w       
}``` </pre>

This is the proc for set_multi_cpu_usage

<pre> ```#!/bin/tclsh

proc set_multi_cpu_usage {args} {
        array set options {-localCpu <num_of_threads> -help "" }
        while {[llength $args]} {
                switch -glob -- [lindex $args 0] {
                	-localCpu {
				set args [lassign $args - options(-localCpu)]
				puts "set_num_threads $options(-localCpu)"
			}
                	-help {
				set args [lassign $args - options(-help) ]
				puts "Usage: set_multi_cpu_usage -localCpu <num_of_threads> -help"
				puts "\t-localCpu - To limit CPU threads used"
				puts "\t-help - To print usage"
                      	}
                }
        }
}``` </pre>

<img width="497" height="466" alt="image" src="https://github.com/user-attachments/assets/0c5e3cc6-e01a-4c69-850e-a62ff7eed201" />

read_lib.proc This proc generates the commands needed to load both early and late timing libraries required by the OpenTimer tool.

<pre> ```#!/bin/tclsh

proc read_lib args {
	# Setting command parameter options and its values
	array set options {-late <late_lib_path> -early <early_lib_path> -help ""}
	while {[llength $args]} {
		switch -glob -- [lindex $args 0] {
		-late {
			set args [lassign $args - options(-late) ]
			puts "set_late_celllib_fpath $options(-late)"
		      }
		-early {
			set args [lassign $args - options(-early) ]
			puts "set_early_celllib_fpath $options(-early)"
		       }
		-help {
			set args [lassign $args - options(-help) ]
			puts "Usage: read_lib -late <late_lib_path> -early <early_lib_path>"
			puts "-late <provide late library path>"
			puts "-early <provide early library path>"
			puts "-help - Provides user deatails on how to use the command"
		      }	
		default break
		}
	}
}``` </pre>

This is the proc for read_verilog

<pre> ```#!/bin/tclsh

# Proc to convert read_verilog to OpenTimer format
proc read_verilog {arg1} {
	puts "set_verilog_fpath $arg1"
}``` </pre>


read_sdc.proc This proc generates the necessary commands to load the .timing constraints file required by the OpenTimer tool. It performs a conversion of SDC file contents into a .timing file format compatible with OpenTimer.

<pre> ```#!/bin/tclsh

proc read_sdc {arg1} {

# 'file dirname <>' to get directory path only from full path
set sdc_dirname [file dirname $arg1]
# 'file tail <>' to get last element
set sdc_filename [lindex [split [file tail $arg1] .] 0 ]
set sdc [open $arg1 r]
set tmp_file [open /tmp/1 w]

# Removing "[" & "]" from SDC for further processing the data with 'lindex'
# 'read <>' to read entire file
puts -nonewline $tmp_file [string map {"\[" "" "\]" " "} [read $sdc]]     
close $tmp_file

# Opening tmp file to write constraints converted from generated SDC
set timing_file [open /tmp/3 w]

# Converting create_clock constraints
# -----------------------------------
set tmp_file [open /tmp/1 r]
set lines [split [read $tmp_file] "\n"]
# 'lsearch -all -inline' to search list for pattern and retain elementas with pattern only
set find_clocks [lsearch -all -inline $lines "create_clock*"]
foreach elem $find_clocks {
	set clock_port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
	set clock_period [lindex $elem [expr {[lsearch $elem "-period"]+1}]]
	set duty_cycle [expr {100 - [expr {[lindex [lindex $elem [expr {[lsearch $elem "-waveform"]+1}]] 1]*100/$clock_period}]}]
	puts $timing_file "\nclock $clock_port_name $clock_period $duty_cycle"
}
close $tmp_file``` </pre>


Code and Results for STA


<pre> ```puts "\nInfo: Timing Analysis Started....."
puts "\nInfo: Initializing number of threads, libraries, sdc, verilog netlist path...."
source /home/vsduser/vsdsynth/procs/reopenStdout.proc
source /home/vsduser/vsdsynth/procs/set_num_threads.proc
reopenStdout $OutputDirectory/$DesignName.conf
set_multi_cpu_usage -localCpu 2
#return

source /home/vsduser/vsdsynth/procs/read_lib.proc
read_lib -early /home/vsduser/vsdsynth/osu018_stdcells.lib

read_lib -late /home/vsduser/vsdsynth/osu018_stdcells.lib

source /home/vsduser/vsdsynth/procs/read_verilog.proc
read_verilog $OutputDirectory/$DesignName.final.synth.v

source /home/vsduser/vsdsynth/procs/read_sdc.proc
read_sdc $OutputDirectory/$DesignName.sdc
reopenStdout /dev/tty
#return
if {$enable_prelayout_timing == 1} {
	puts "\nInfo: enable_prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"
	set spef_file [open $OutputDirectory/$DesignName.spef w]
puts $spef_file "*SPEF \"IEEE 1481-1998\" "
puts $spef_file "*DESIGN \"$DesignName\" "
puts $spef_file "*DATE \"Sun May 11 20:51:50 2025\" "
puts $spef_file "*VENDOR \"PS 2025 Hackathon\" "
puts $spef_file "*PROGRAM \"Benchmark Parasitic Generator\" "
puts $spef_file "*VERSION \"0.0\" "
puts $spef_file "*DESIGN_FLOW \"NETLIST_TYPE_VERILOG\" "
puts $spef_file "*DIVIDER / "
puts $spef_file "*DELIMITER : "
puts $spef_file "*BUS_DELIMITER [ ] "
puts $spef_file "*T_UNIT 1 PS "
puts $spef_file "*C_UNIT 1 FF "
puts $spef_file "*R_UNIT 1 KOHM "
puts $spef_file "*L_UNIT 1 UH "
}

close $spef_file

set conf_file [open $OutputDirectory/$DesignName.conf a]
puts $conf_file "set_spef_fpath $OutputDirectory/$DesignName.spef"
puts $conf_file "init_timer "
puts $conf_file "report_timer "
puts $conf_file "report_wns "
puts $conf_file "report_worst_paths -numPaths 10000 "
close $conf_file

set tcl_precision 3

set time_elapsed_in_us [time {exec /home/vsduser/OpenTimer-1.0.5/bin/OpenTimer < $OutputDirectory/$DesignName.conf >& $OutputDirectory/$DesignName.results} 1]
set time_elapsed_in_sec "[expr {[lindex $time_elapsed_in_us 0]/100000}] sec"
puts "\nInfo: STA finished in $time_elapsed_in_sec seconds"
puts "\nInfo: Refer to $OutputDirectory/$DesignName.results for warning and errors"

puts "tcl_precision is $tcl_precision``` </pre>


<img width="567" height="1067" alt="image" src="https://github.com/user-attachments/assets/674081d7-b465-422a-b03c-dd3402c0a07c" />

<img width="1021" height="941" alt="image" src="https://github.com/user-attachments/assets/4766f93f-791e-4e8a-a5f6-9be3c140ed60" />

<img width="1007" height="895" alt="image" src="https://github.com/user-attachments/assets/6d86f089-0a47-4fef-8d62-3791b6c6a3b3" />

<img width="1108" height="949" alt="image" src="https://github.com/user-attachments/assets/39e5b19c-0dc2-47dc-a870-cdb5c72b2fab" />

<img width="1054" height="934" alt="image" src="https://github.com/user-attachments/assets/17b946ef-0666-48c3-a3f5-c77881180da7" />

<img width="1067" height="440" alt="image" src="https://github.com/user-attachments/assets/7c5e7e04-502f-49c5-8d94-b61772c3afeb" />

<img width="1058" height="345" alt="image" src="https://github.com/user-attachments/assets/670373ce-3b78-4c3d-b716-6c8eb7d702cf" />

I have successfully developed a script to extract all essential data points from the .results file and other relevant sources for Quality of Results (QoR) analysis. This includes capturing key metrics along with the total runtime of the entire .tcl script. The core implementation is presented below, accompanied by terminal screenshots that display variable outputs and debugging details using puts statements.

<pre> ```#---------------find WNS using RAT-------------------------#
set worst_RAT_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {RAT}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_RAT_slack "[expr {[lindex $line 3]/1000}]ns"
		puts "worst_RAT_slack is $worst_RAT_slack"
		break
	} else {
		continue
	}
}
close $report_file
#return

#--------------------------fine number of output violation------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_output_violations $count
puts "Number of output violations $Number_of_output_violations"
close $report_file


#---------------find WNS setup violation-------------------------#
set worst_negative_setup_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {Setup}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set worst_negative_setup_slack "[expr {[lindex $line 3]/1000}]ns"
                break
        } else {
                continue
        }
}
close $report_file

#---------------find number of setup violation-------------------------#

set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_setup_violations $count
close $report_file

#---------------find WNS hold violation-------------------------#
set worst_negative_hold_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {Hold}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set worst_negative_hold_slack "[expr {[lindex $line 3]/1000}]ns"
                break
        } else {
                continue
        }
}
close $report_file

#---------------find number of hold violation-------------------------#

set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_hold_violations $count
close $report_file


#---------------find number of instance---------------------------#
set pattern {Num of gates}
puts "pattern is $pattern"
set report_file [open $OutputDirectory/$DesignName.results r]
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set Instance_count [lindex [join $line " "] 4 ]
                break
        } else {
                continue
        }
}
close $report_file

set Instance_count "$Instance_count PS"
set time_elapsed_in_sec "$time_elapsed_in_sec PS"``` </pre>

<img width="1032" height="501" alt="image" src="https://github.com/user-attachments/assets/ab0853ff-e105-4b97-a82a-c5db4ea7ae47" />

The code for generating the Quality of Results (QoR) report has been successfully implemented.

<pre> ```set formatStr {%15s%15s%15s%15s%15s%15s%15s%15s%15s}

puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
puts [format $formatStr "Design Name" "Runtime" "Instance Count" " WNS Setup " " FEP Setup " " WNS Hold " " FEP Hold " "WNS RAT" "FEP RAT"]
puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
foreach design_name $DesignName runtime $time_elapsed_in_sec instance_count $Instance_count wns_setup $worst_negative_setup_slack fep_setup $Number_of_setup_violations wns_hold $worst_negative_hold_slack fep_hold $Number_of_hold_violations wns_rat $worst_RAT_slack fep_rat $Number_of_output_violations {
	puts [format $formatStr $design_name $runtime $instance_count $wns_setup $fep_setup $wns_hold $fep_hold $wns_rat $fep_rat]
}

puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
puts "\n"``` </pre>


<img width="1030" height="461" alt="image" src="https://github.com/user-attachments/assets/2b16ee5f-f8eb-4699-9df1-2f2026687317" />

<img width="636" height="652" alt="image" src="https://github.com/user-attachments/assets/61e9da86-ac11-4822-9d38-4042448ebcba" />

<img width="1037" height="346" alt="image" src="https://github.com/user-attachments/assets/cc0bdf68-b2bb-4971-940d-89b62eab9853" />

<img width="1037" height="482" alt="image" src="https://github.com/user-attachments/assets/40540462-dbcb-488c-9da7-5b1922059768" />

Tools Used
TCL Development Suite
Yosys – Open-source RTL synthesis
OpenTimer – Static timing analysis
VSD custom libraries



