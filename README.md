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

</pre>```tclset filename [lindex $argv 0]
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



