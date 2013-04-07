# gmsGrep


## gmsGrep – A data reduction and graphing Unix script for GAMESS output

## Objective

The objective of the project is to generate an HTML report that summarizes a GAMESS output file and includes a graphical representation of the molecular system and optimization process. GAMESS (General Atomic and Molecular Structure System) is a freely distributed package of programs written in C and Fortran linked together with Unix scripts that normally utilizes multi-processor and/or cluster computing. The purpose of the program is to simulate the physical and chemical behavior of atoms and molecules utilizing an “ab initio” model, that is, a model that approximates the quantum mechanical behavior of electrons. The most common type of modeling performed is the calculation of the energy of a system before and after some geometry optimization process is carried out. The optimization process normally consists of an alternating search between nuclear coordinates and molecular orbital shapes that lowers or raises the total energy of the system using an algorithm such as the Newton-Raphson method. The GAMESS output file includes authorship information, data taken as input, initial parameters setup before the optimization process begins, a detailed sequential list of calculations during the optimization process, and a lengthy post-optimization section. Generally, the output file comprises 25,000 – 100,000 lines of text. To find output data of interest, the user must search through the text with regular expression searches using an editor such as the vi editor or read the output file into a program that can interpret it. The goal of this project is to design a script that searches for what the author considers the “essential” information from the output file and writes a report in HTML format that includes a graphical representation of the optimization process that adds value to the report by allowing the user to quickly see what changes occurred at each step and make measurements on the system at each step.

### Tested With This Software

- GAMESS release January 12, 2009
- Linux CentOS version 5.3 
- Linux Ubuntu version 9.04
- Jmol 11.7.43
- GnuPlot 4.2.5
- Methods

The Unix tools used for searching and extracting data  were the programs GNU GREP 2.5.1-cvs and MAWK Version 1.2. GREP is a well-known program for string searching included in all distributions of linux and MAWK is an intepreter for the AWK programming language. MAWK was also used to write HTML output embedded with data from GAMESS and invoke the applet Jmol. The GAMESS output file contains only ASCII characters and unique labels for most types of data. For many searches, it was sufficient to locate a regular expression with GREP or MAWK and select the field within MAWK, using a command sequence such as: 

    cat filename | grep 'LABEL' | mawk '{ printf(“%s”, $field }' or 
    cat filename | mawk '/LABEL/ { printf(“%s”, $field) }

The only complication of printing the fields in MAWK was including HTML tags in the printf commands, for example:

    mawk '{ printf(“<P><FONT STYLE=”” ><FONT SIZE=# >Description: %s</FONT>
    </FONT></P>\n”, $field) }

To count the number of iterations in the geometry search, all lines (records in AWK terminology) were found by searching for the label 'NSERCH' and then searching for 'ENERGY' within the search results of 'NSERCH', This was accomplished with two GREP commands. The record at the start of the geometry search has the format

    NSERCH= #     ENERGY=    -#.#

where the search step number is field 2 and the energy associated with the geometry at that step is field 4. The following command sequence can be used to write fields 2 and 4 to a file “temp.dat”:

    cat $1 | grep 'NSERCH' | grep 'ENERGY' | mawk '{print $2, $4}' > temp.dat

The “temp.dat” file was used as a temporary data file for graphing with GnuPlot.  A similar command sequence was used to print the final optimization step and the corresponding energy:

    cat $1 | grep 'NSERCH' | grep 'ENERGY' | mawk 'NR == $NR

This command sequence allowed fields 2 and 4 to printed in a printf() statement.

To display the input parameters for the calculations, a for loop was used to print every input line identified by the                    
    string 'INPUT CARD':

    cat $1 | grep 'INPUT CARD' | grep '\$END' | mawk '{line[NR] = $0}		 

    END { for(i=1; i<=NR; i++) { 
        printf("<><FONT SIZE=2>")		 
        printf("%s</FONT></FONT></P>\n", line[i])} }';

### Creation of a 2D plot with GnuPlot

A simple 2D plot is created for each report by using a script that inputs data into gnuplot for plotting. The data from the file “temp.dat” is read into the GnuPlot program using the script “plotscript” and field 1 is assigned the x-coordinate, field 2 the y-coordinate. A title and labels are easily added as shown in the following script:

    “plotscript”

    set xlabel "Search Step" 
    set ylabel "Energy (Hartrees)" 
    set title "Geometry Optimization" 
    plot "temp.dat" using 1:2 with lines 
    set terminal png 
    set output "temp.png" 
    replot

The plot is then exported as a png graphics file. The graphics file “temp.png” is renamed as the GAMESS output filename with a .png extension and the file “temp.dat” is erased so that “temp.dat” can be used again (this makes running multiple instances of gmsGrep concurrently dangerous, and may be changed in the future). The renaming of the png file, deletion of the data file and insertion of the graphic into the HTML report is accomplished by the following commands:

    gnuplot plotscript; 
    mv temp.png $1.png; rm temp.dat;

    echo $1 | mawk 'END {printf("<img src=\"./%s.png\n\" /><BR><BR><BR>", $1)}'

In this way a 2D plot of the energy of the system at each step of the optimization is embedded into the HTML report.

### Creation of an animated 3D plot with Jmol

The 3-D graphical representation of the molecular system is most useful when the atoms and bonds between them can visualized. For this level of detail, nuclear coordinates are sufficient only if they can be interpreted by a program that can calculate bonds by their relative proximities. Rather than use a plotting program such as GnuPlot for the 3-D representation, it seemed more practical and efficient to make use of the program Jmol, which is free and platform independent (as long as the system has a Java virtual machine and a web browser capable of handling applets). Jmol can interpret most types of molecular data files including GAMESS output. In addition, it has built-in functions that allow animation of a series of atomic coordinates (such as a minimization procedure or reaction). Adding a Jmol applet requires only a few lines of code and the addition of the required Jmol files to the web page directory or a subdirectory. 

Code for HTML HEAD section:
    
    {printf("\t<script type=\"text/javascript\" src=\"./Jmol.js\"></script>\n")}

Code for HTML BODY section
    
    echo $1 | mawk ' END { {printf("<script type=\"text/javascript\">")} 
    {printf("jmolInitialize(\"./\");\n")} 
    {printf("jmolApplet(400, \"load %s\");\n", $1)} 
    {printf("</script><BR><BR><BR><BR>\n")} 

The Jmol applet provides a pop-up control menu that is accessed by right-clicking anywhere in the molecule viewer window. This control menu includes options to view a frame-by-frame animation, make measurements of bond lengths and bond angles, and graphical choices for representations of atoms and bonds. The geometry optimization from any GAMESS output file can be viewed in forward and reverse by using the animation controls.

## Results

The gmsGrep script was tested against several GAMESS output files and behaved as intended, thanks to the predictability of the GAMESS output files. In all cases, the HTML document, 2D plot and 3D interactive plot were successfully generated. A complete report is appended to this document and a few examples of 2D and 3D plots are shown below (unfortunately without animations). The command syntax for calling the gmsGrep script is gmsGrep filname.ext. The following data is included in the report:

### General Information

1. Report Creation Date: Actual date and time the report was generated.
2. Host: The host server that GAMESS was running on for the given job.
3. Job start date/time: Time the job started on the host server. 
4. Job end date/time: Time the job ended on the host server.

### Input Parameters: The complete list of input controls and parameters.

#### Basis Options: The complete list of basis functions  used to model the orbitals.

Assigned Parameters

1. Point group of molecule: The symmetry-assigned point group of the molecule.
2. Order of principal axis: Principal axis according to the point group.
3. Total number of basis set shells: Number of orbital shells.
4. Number of Gaussian Cartesian basis functions
5. Number of electrons.
6. Charge of molecule.
7. Spin multiplicity.
8. Total number of atoms.
9. Nuclear repulsion energy.

## Optimization Results

1. Nuclear coordinate geometry search steps.
2. Optimized geometry energy.
3. Maximum gradient (last step).
4. RMS (root mean squared) of all gradients (last step).

## Energy Components (post calculations)

1. Wavefunction normalization.
2. One electron energy.
3. Two electron energy.
4. Nuclear repulsion energy (final).
5. Total energy.
6. Total potential energy.
7. Total kinetic energy.
8. Total wall clock time.
