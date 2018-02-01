# ClinSeq
Automatic Annotation and deposition of UW Virology clinical sequencing data
This repository contains scripts for analysis of assembled fasta files of known human viruses. This program annotates these viruses and then packages them up for NCBI deposition. All you need to get going on this is a computer and some fasta files of human viruses. Feed this program the viruses and .sqn files that you can just email to NCBI will be generated!
I can be contacted at uwvirongs@gmail.com for bug reports or feature requests - you can also make them here on github if you like.

The only information you need to supply is a fasta with as many viruses as you'd like to annotate in it as well as an optional metadata sheet and a .sbt file for author lists in the .sqn and .gbf files that NCBI likes. 

# Installation
Installation differs greatly for Unix systems and for Windows

Linux

1. Install all dependencies (Shoutout to the wonderful people who wrote these!)

Python - tested almost exclusively on 2.7.4 but there's nothing in here that shouldn't work on later versions.
The numpy and biopython module for Python which can generally be install by opening the terminal and typing 
pip install numpy 
pip install biopython

Then you'll need to install tbl2asn and put it on your path. (Another option is to download the appropriate version and simply unpack it directly into this repository once you've downloaded it. 
tbl2asn https://www.ncbi.nlm.nih.gov/genbank/tbl2asn2/

Windows

1. Install Microsoft Visual C++ for Python 2.7 https://www.microsoft.com/en-us/download/details.aspx?id=44266
2. Install Python 2.7.4 https://www.python.org/download/releases/2.7.4/ (Although 2.7.5 works as well)
  Make sure that when this is installing on the third step 'Customize your python installation' the box that says put python on your path is checked. This makes installing and running this annotator much easier. 

3. Install Numpy and Biopython - if python is installed correctly this can be done easily by opening a command prompt window and typing:
pip install numpy
pip install biopython

4. Then download and unzip this repository to your computer (anywhere will do)

5. Download tbl2asn for windows and unzip it, then copy and paste every single file in the tbl2asn folder (there should be like 7 .dlls and an .exe that is simply called tbl2asn 

After the above has completed there's still a bit of setup that needs to be done. 

1. A template .sbt file that contains your name and information about the orginization that you wish to submit your sequences to NCBI under. This .sbt file can be generated by filling out the form here https://submit.ncbi.nlm.nih.gov/genbank/template/submission/

2. Put the newly generated template.sbt file onto your computer. (Generally it's easist to just but it in the ClinVirusSeq folder) If you'll be submitting mulitple sequences from different people you can generate more than one, put them all in this folder, and choose which one you want to use at run time. 

3. (Optional)
You can generate a .csv file with most metadata that you wish to associate with your sequences. The file should have across the top Strain (the name of the fasta sequences that you'll import) followd by colloumns with NCBI aprooved metadata https://www.ncbi.nlm.nih.gov/Sequin/modifiers.html

An example file has been provided under the name example_metadata.csv 

# Usage - ClinVirusSeq.py
Create your fasta file with all of the sequences that you would like to annotate - you should make the names of the sequences (the things after >) what you would like the strain of the virus (In your NCBI Genbank records) to be. 

Then you would need to run the ClinVirusSeq.py script from the command line. i.e. cd to the directory that this is living it - if you cloned from github it'll be ClinVirusSeq/ 

You can run it with no arguments or the -h flag and it'll print out some usage information. But basically the way you run this is to type (without quotes) "ClinVirusSeq.py fasta_file metadata_info_sheet sbt_file_loc" where fasta_file is the location of all your sequences, metadata_info_sheet is the location of the semi-optional metadata sheet, and sbt_file_loc is the location of your .sbt file. I have it set to take a specified .sbt file so that you can create them with different author lists if you need to to make it easier.

Running the script without the -r flag all of your records will be in individual folders named by "strain" (what came after > in your fasta file). As a side note - please don't put whitespace or backslashes in your strain names. I would recommend doing this if you know you'll need to run this several times due to bugs or other issues. For example if you're debugging otherwise just use -r the first time

If you're feeling bold just put the -r flag on the first time and all your .sqn files will get consolidated. Your output will be in a folder with the date YEAR_MONTH_DAY/virus_species/ The script creates consolidated .sqn files by virus species because that's what the lovely people at Genbank asked us to do. So you'll have a date folder for each fasta file - then another sub folder for each virus species and inside these folders will be species.sqn which should be what you wanna submit to NCBI. You can find individual records inside the subfolders (named by strains)  (so if the first line of your fasta file is >SC12309 then there will be a folder called SC12309 with all of the .gbf .tbl and ect files that you need and love) Note that this does not (yet) protect against the same virus blasting to separate records that other people (sometimes me honestly) named two different things. (This is actually a pretty hard problem to solve without hard coding every virus, which I may end up doing)

Also - any records that had stop codons will not be moved into the species folders - they'll be left out in your main directory for you to examine - most often this comes from errors with coverage but sometimes it's because I introduced a catastrophic bug.
tbl2asn will also spit out some errors and warnings onto the command line which are obviously pretty helpful. One known bug is that if you pass a sequence that doesn't contain every orf in its entirety you'll get start codons of 1< which obviously tbl2asn doesn't like

# Implementation Details and Important Notes

Right now we accomplish the annotation by searching blast for the best hit that is a complete genome and using a maaft alignment to generate annotations for the supplied sequence. Basing the annotations off the best blast hit. We can handle some ribosomal slippage as well as RNA editing in HPIV/Measles/Mumps/Sendai with support for more and more viruses as I end up having to deal with them. Coronavirus 229E and HPIV3 are annotated off hardcoded records due to variability in accuracy for these records in blast.

------------------------------------------------------------------------------------------------------------------------------------------
IMPORTANT SECURITY NOTE: DO NOT PUT THIS CODE ON A SERVER AND LET IT FACE UNKNOWN CLIENTS!!! 
This code uses direct subprocess commands with user supplied input so it would be trivial to insert bash commands into the .fasta file and completely destroy a server. I don't see any reason why anyone would do this but if you are putting it on a server make sure that only people you know won't destroy your server will use it. Also - I guess don't name your strains really stupid things like & rm -rf / or & :(){ :|: & };: I doubt this would work as I've written it so you'll be fine as long as you name your strains normal alphanumeric strings or normal human words. Just don't let potentially malicious users run this code.
------------------------------------------------------------------------------------------------------------------------------------------

# Future directions
Currently developing a way of going directly from NGS reads coming straight off the Illumina to reference, assemble off the reference and then annotate, although this is in no way working right now stay tuned! :) Also going to try to have some way of consolidating gene product annotations because NCBI was complaining about that and I agree it's kinda bad and right now I'm manually reviewing all my annotations, which is slow.
