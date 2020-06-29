# Coverity-Python3-Integration
Scripts to include Bandit SAST tool results into Coverity server
Bandit static analysis results in Connect

Adding third party static analysis results for Python will certainly add value to the core Coverity analysis.
Incorporating results from such an external tool thus fits nicely providing another dimension to the overall measurement of a quality posture.
This page relates to running Bandit but could equally be used as a basis for accommodating any of the other similar tools.
Bandit (https://bandit.readthedocs.io/en/latest/index.html) is called for each individual .py source file and a result data file is captured (I have named these as .Bandit)
The output is in raw (configurable) text format which needs to be reformatted into JSON for Coverity to directly accept (cov-import-results).
The JSON is in fact "imported" into a normal intermediate directory from where a standard commit of the defects is performed into a stream in Connect.
A new stream would be used for these Bandit results and can be part of a Coverity Project which most likely includes the Coverity Quality Python analysis results too. 
Results can be added to a new stream or to an existing one.
To make the task simpler, a couple of scripts to wrap the syntax to the native Bandit toolchain (to condition the output format for us to later consume) to cover the different scripting shells.
The one attached uses the Bash Shell runs in windows Subsystem Linux version 1 implementation of Ubuntu 20.x with Python3.8.2
Tool Chain:
Install Bandit:
Pip3 Bandit install
Script 1   Bandit1.sh  (Bash Shell)
This Bash shell is called by default to analyze all files and subdirectories from the current directory (PWD=project Dir).
i.e. default bandit1.sh  #will analyze all files under current PWD
One optional parameter to analyze a single file for security audit purposes.
 e.g.    bandit1.sh /path/to/my/project/thisSource1.py


1.Edit bandit1.sh and update following toolchain variables:
analysisToolsRoot=""   # location of Bandit 1.6.2
analysisTool="bandit"   # name of the tool Bandit
This script will call the native Bandit toolchain with fixed arguments (-q : silent, -f custom template).
Output:
.py.bandit files will be written in the same folder alongside the source files analyzed 
  
Script 2   Bandit2.sh
 
This shell script requires up to 1 or 2 arguments
1              is the Coverity  int dir where we are to import the  results (optional, "idir-Bandit" assumed)
2              is the project root where we are to scan from (optional, <current folder> assumed)
e.g. Bandit2.sh myproject-Bandit-idir # idir-bandit default
 
Any .Bandit result files are scanned for under the project root
They are initially read and converted into a temporary JSON format (see Bandit_import.py) and then imported into the nominated Coverity int dir as "other domain" analysis results (using the std cov-import-results command)
The script offers an option to also remove these .Bandit result files if their import was successful.
 The int dir can then be committed to the Connect server
 The same steps would be true for any type of external analysis tool 
In this case my first script wraps the native tool to ensure the configurable native tool output format is exactly as we expect to be read by the "converter" script (actually written in python, see Bandit_import3.py and coverity_bandit_import3.py which it depends upon)
For other toolchains the Bandit_import3.py script would be swapped out for one specific to the fixed/configurable native format of that analysis tool.
There are some other platform configurations possible to further customize the display of the external analysis findings. 
Install the script contents referenced above into a single folder and add this to the PATH:
Bandit1.sh  // Python wrapper to invoke analysis
Bandit2.sh   //Take input results from previous formatting steps and import them into Coverity Intermediate directory
Bandit_import3.py   // Data transformation
coverity_bandit_import3.py  //Data Transformation
 
workflow
1.	cd <your project root>
2.	sh Bandit1.sh
3.	sh Bandit2.sh
4.	cov-commit-defects --dir idir-Bandit --host <host> --user admin --password coverity --stream "<project>-Bandit"
 
Note:
You will probably need to alter the #! line in the scripts to the correct shell and path for your environment - I have prefixed the commands above with "sh" to force a known shell interpretation
 If you are interested in also including SCM (blame/annotation) data for the python source files (for display in Connect and auto defect assignment) then perform these extra steps before committing the results.
  # import any derivable SCM information, as required - driven by the existing TU content
  cov-import-scm  --dir idir-Bandit  --scm <scm tool name>
The above cov-import-scm is a short cut for the following underlying commands which could be run instead but with more granular control - see the filter line
      # derive a list of sources we need to fetch SCM data for
      cov-manage-emit --dir idir list-scm-unknown --output files-scm-unknown.txt
      <<TODO>>  filter this list of files (without scm information) as required - some might be headers or generated code?
      # get SCM data for this modified (more appropriate) list of files
      cov-extract-scm --input files-scm-unknown.txt --output myscm.data
      # add the derived scm data into the idir
      cov-manage-emit --dir idir add-scm-annotations --input myscm.data
 
# import an "empty results" JSON file (empty.json) to clear the "dirty" flag set by the SCM addition (checked at the commit stage)
cov-import-results  --dir idir-Bandit  --other-domain  --append empty.json  --strip-path <project root>
 
Installation tweaking / controls 
Define the path location where the native Bandit tool can be found (see Bandit1.sh,  analysisToolsRoot= i.e. "C:/Python27/Scripts/")
If the Coverity Analysis tools (bin folder) is not in the PATH, define covAnalysisInstallRoot="<path to Coverity analysis/bin>" (see Bandit2.sh)
If the above supplied .py scripts are not in the PATH, similarly define covImportToolsRoot="<path to import scripts>"
 
Other useful shell config options to note for testing/debugging:
keep                                   for keeping (1) temporary files or not, usually zero (0).
debugLevel                        for showing the script inner workings (3 is a good number to use), usually zero (0).
dryRun                               for performing the majority of the logic except for the main purpose 'call' where it will be stepped over (1), usually zero (0) for normal operation - see scripts for a better explanation!
removeResult                     (1) if the .Bandit result file is also to be removed after it's importing else leave for manual clean up (0)
 
Enjoy!



