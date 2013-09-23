Based on the observations, a list of critical files & methods that every BSP must include and use was compiled. These files & methods belonging to a component for a BSP can be deemed as critical or non-critical. The rule-checking is accomplished by the following way :

1)Check if a file is being compiled by searching it's name in the Makefile.am for a BSP.
2)If file not found, then report it.
3)Check if the file being compiled is present in the correct directory as defined by the rules.
4)If location of file is not correct, report it.
5)Check if the corresponding functions for a file are defined in there or not. This takes place by searching for the definition of a method in that file with the help of a simple regex [a-z|A-Z|0-9|_]+[ ]*$i[ ]*\(
6)If the function is not located in it's specified file, then search whether it's defined in some other file maybe. Report it
7)Check whether a required header is being installed by searching it's name in the Makefile.am
8)Compile a list of all RTEMS internal functions(i.e starting with an '_' ) defined in cpukit & libcpu.
9)Compile another list of functions starting with an '_' being executed in BSP by searching for a ';' to ensure that function is actually being called. Searching for a function being called from a file can be a bit intimidating if the function calling spans multiple lines, meaning that the function name is on one line & the ';' on another. Since grep only parses line by line, so it cannot do this job very easily. So all newlines(\n) were trimmed while searching a file. The whole file now being concatenated in a single line, so grep is ideal for searching the pattern [ |^][_]+[a-z|A-Z|0-9|_]*[ ]*[\n]*\([^;]*\)[ ]*;
10)Find common function names in the above two lists in order to ensure that the function being called in a bsp is infact been defined in cpukit or libcpu. Report it.
11)In order to obtain the results of the script successfully, either call it from a libbsp, cpu family or a bsp directory or pass it a path for a single or multiple bsp directories so that it can validate the location of a bsp in the RTEMS tree. If no path is passed, the pwd is evaluated and searched for bsps.
12)Passing only the directory name for bsp, cpu family or libbsp displays the discrepancies found only for critical files & functions. Following parameters need to be passed to expand the horizon

--warnings : gives all the critical & non-critical discrepancies found in a bsp.
--verbose : gives the whole story of a BSP. Any discrepancy & a correct rule implementation is also displayed.
--tests : gives test specific checks for each given BSP.

Some of the interesting results found with the script were :

arm/gumstix : bspreset.c not compiled
arm/gumstix : bsp_reset() present in file startup/bspstart.c
powerpc/beatnik : bspreset.c not compiled
powerpc/beatnik : bsp_reset() present in file include/bsp.h startup/reboot.c
powerpc/mpc55xxevb : start.S not present in correct path
powerpc/mpc55xxevb : bspreset.c not compiled
powerpc/mpc55xxevb : bsp_reset() present in file startup/reset.c
powerpc/mpc55xxevb : bspgetworkarea.c not compiled
powerpc/mpc55xxevb : bsp_work_area_initialize() present in file startup/bspworkareainit.c