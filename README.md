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

if ($#argv != 1) then
	echo "Info: Please provide the csv file"
	exit 1
endif

if (! -f $argv[1] || $argv[1] == "-help") then
	if ($argv[1] != "-help") then
		echo "Error: Cannot find csv file $argv[1]. Exiting..."
		exit 1
	else
		echo USAGE: ./vsdsynth \<csv file\>
		echo
		echo        where \csv file\> consists of 2 columns, below keyword being in 1st column and is Case Sensitive. PLease request PS for sample csv file
		echo
		echo        \<Design Name\> is the name of the top level module
                echo
                echo        \<Output Directory\> is the name of the output directory where you want to dump synthesis script, synthesized netlist and timing reports
                echo
                echo        \<Netlist Directory\> is the name of  directory where all the RTL netlist are present
                echo
                echo        \<Early Library Path\> is the file path of the early cell library to be used for STA
                echo
                echo        \<Late Library Path\> is the file path of the late cell library to be used for STA
                echo
                echo        \<Constraints file\> is csv file path of contraints to be used for STA
		echo
		exit 1
	endif
else tclsh ./vsdsynth.tcl $argv[1]
endif



