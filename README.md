**Coverity-Python3-Integration.MD**

Scripts to include Bandit SAST tool results into Coverity server
Bandit static analysis version 1.6.2 results in Connect

Coverity Third Party Integration Toolkit allows external data to be imported into the Coverity Connect Server and leverage existing functionalities available such email notification to developers, project leads, auto-assignment of defects to engineers, triaging defects, Reporting, etc. 
Adding third party static analysis results for Python3 in this case will certainly add value to the core Coverity analysis. Incorporating results from external tools fits nicely, providing another dimension to the overall measurement of a security posture. Additional custom languages could be added by reusing the techniques used here.

This page relates to running Bandit but could equally be used as the basis for accommodating any similar Open source SAST, Standards or Rules based enforcing tool.

HIgh level Workflow 
Bandit OSS SAST (https://bandit.readthedocs.io/en/latest/index.html) is called for each individual .py source file and a result data file is captured (named as .Bandit).

The output is in raw (configurable) text format which needs to be reformatted into proper 'Coverity' JSON file to be directly accepted into the Coverity Platform database (cov-import-results).

The 'Coverity' JSON file is in fact "imported" into an empty or existing Coverity intermediate directory from where a standard commit of the defects is performed into a stream in Connect.

A new or existing stream could be used for adding these Bandit results. In this way these new results can be combined with existing Python or other language analysis results too!!. A separate stream maybe recomended, as Coverity and Bandit may then be run at different rates.

To make the task simpler, scripts are provided to wrap the syntax of the native Bandit toolchain (to condition the output format for later consumption) to cover the different scripting shells.

The one attached uses the Bash Shell in Ubuntu 18.04 with Python 3.8.2 and Bandit 1.6.2 

Requirements:
Install Python 3.x
Install Bandit 1.6.2 or later, $ pip3 install bandit


Shell Script 1
Bandit1.sh  
This Bash shell is called to analyze all python files( *.py )  and subdirectories from the current root directory.

An optional parameter is available to analyze a single file for security audit purposes.
$ bandit1.sh /path/to/my/project/thisSource1.py

Edit bandit1.sh and update following toolchain variables:
analysisToolsRoot=""   # location of Bandit 1.6.2
analysisTool="bandit"   # name of the tool Bandit

This script will call the native Bandit toolchain with fixed arguments (-q : silent, -f custom template).

Output:
*.py.bandit files will be written in the same folder alongside the source files analyzed with the .bandit appended at the end of the file name.

Shell Script 2 
Bandit2.sh 
This shell script takes 0 (Default), 1 or 2 arguments
The first argument is the location of the intermediate directory that will be created where we are to import the results (optional, "idir-Bandit" default name in current directory )
The second is the project root where we are to scan from (optional, <current folder> assumed)

e.g. **Bandit2.sh myproject-Bandit-idir myprojectFolderRoot**

 .Bandit result files are scanned under the myprojectFolderRoot to be imported in the intermediate directory

They are initially read and converted into a 'Coverity' JSON format (see Bandit_import3.py) and then imported into the nominated Coverity int dir as analysis results (using the std cov-import-results command)

The script offers an option to also remove these .Bandit result files if their import was successful.

The int dir can then be committed to the Connect server
The same steps would be true for any type of external analysis tool

In this case the first script wraps the native tool to ensure the configurable native tool output format is exactly as we expect to be read by the "converter" script (actually written in python, see Bandit_import3.py and coverity_bandit_import3.py which it depends upon)
For other toolchains the Bandit_import3.py script would be swapped out for one specific to the fixed/configurable native format of that analysis tool.
There are some other platform configurations possible to further customize the display of the external analysis findings.


Install the script contents referenced above into a single folder and add this folder to your or server PATH.

**Artifacts**

*bandit1.sh*  // wrapper script to invoke Bandit analysis
*bandit2.sh*  // wrapper script that takes previous text analysis results inputs formats into 'Coverity' JSON  format and imports them into Coverity Intermediate directory
*Bandit_import3.py*   // Data transformation
*coverity_bandit_import3.py*  //Data Transformation and import json into int dir

**workflow**
1. cd <your project root>
2. bash ./bandit1.sh
3. bash ./bandit2.sh   [<int Dir> [<ProjectRootDir>]]
4. cov-commit-defects --dir iDir --host <hostname> --user admin --password coverity --stream "<project>-Bandit"

Note:
You will probably need to alter the first line in the scripts to the correct shell and path for your environment - I have prefixed the commands above with "sh" to force a known shell interpretation
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

