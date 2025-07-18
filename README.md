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


