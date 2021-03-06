Strender
========

* Add debug mode so all commands and details are reported.
* Capture PIDs as well as FIDs and trace process executions (fork,execve)
* Pull in timing data.
* Make a proper class for the data and write out as xml.
* Move the primary script to python, and allow the command to be passed along the CLI.
* Rename the list-files script to strender-trace-parser out.log time.out.log
* Use proper temp files for those logs.

Aim is to understand all dependencies, process structure is of interest, but as a debug thing really.

HTTPS is rather difficult! (of course!)
Would require a proxy approach to catch the arguments
But we can still spot the HTTPS request to an IP Address

depends on strace, python, python-lxml, file, sha1sum, ...

Example output:

----
<?xml ?>
<trace pid="XXX">

 <process pid="XXX">
  <execve pid="XXX" command=""/>
  <times>
  </times>
 </process>

 <connections>
  <connect host="" port="" portProtocol="DNS" fromPid="pid" />
  <connect host="" port="" portProtocol="HTTP" fromPid="pid">
   <http method="GET" uri="http://" />
  </connect>
 </connections>

 <files>
  <file mode="READ" fromPackage="package" fromPid="pid" type="" sha1="">file.ext</file>
 </files>

</trace>

----


Link to package
---------------
dpkg -S /path/file

dlocate /path/file (may be faster)

http://collab-maint.alioth.debian.org/debtree/

### To maintainer ###
http://packages.debian.org/squeeze/devscripts

dd-list pretty-prints it
http://manpages.ubuntu.com/manpages/natty/man1/dd-list.1.html


Notes
=====

strace, scripts

strace -f -o trace.log /usr/bin/acroread /mnt/hgfs/Shared/Z39-87-2006.pdf

strace -f -o acroread-nofile-trace.log /usr/bin/acroread
more acroread-nofile-trace.log | grep open | grep -v ENOENT | grep -v EACCES > acroread-nofile-files.txt
sort acroread-nofile-files.txt | uniq > acroread-nofile-files.srted.uniq.txt

strace -f -o acroread-file-trace.log /usr/bin/acroread /mnt/hgfs/Shared/Z39-87-2006.pdf
more acroread-file-trace.log | grep open | grep -v ENOENT | grep -v EACCES > acroread-file-files.txt
sort acroread-file-files.txt | uniq > acroread-file-files.srted.uniq.txt

strace -f -o acroread-nofontfile-trace.log /usr/bin/acroread ~/drawing-no-fonts.pdf
more acroread-nofontfile-trace.log | grep open | grep -v ENOENT | grep -v EACCES > acroread-nofontfile-files.txt

export ACRO_DISABLE_FONT_CONFIG=true
strace -f -o acroread-file-adfc-trace.log /usr/bin/acroread /mnt/hgfs/Shared/Z39-87-2006.pdf 

more acroread-nofontfile-trace.log | grep RDO | grep -v ENOTDIR | grep -v ENOENT | grep -v EACCES | grep -v "O_DIRECTORY" | grep -i "PFB"

/usr/bin/acroread /a "page=22,view=Fit" /mnt/hgfs/Shared/Z39-87-2006.pdf 

strace -f -o strace-convert-file.log convert /mnt/hgfs/Shared/Z39-87-2006.pdf image.jpg

 - Use packages strace and lsof to inspect dependencies during rendering. See [http://www.netadmintools.com/art353.html][22]  strace -e trace=open -o strace-file.txt xpdf SimulationPaper.pdf Have to page through, and then Symbol and Times New Roman (standard) crop up: [08:08:18] [anj@li15-203 /home/anj]$ diff strace-file.txt strace-nofile.txt | grep -v tmp 110,158c110,111 &amp;lt; open("SimulationPaper.pdf", O_RDONLY|O_LARGEFILE) = 4 &amp;lt; open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 5 &amp;lt; open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 5 &amp;lt; open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 5 &amp;lt; open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 5 --- &amp;gt; --- SIGINT (Interrupt) @ 0 (0) --- &amp;gt; +++ killed by SIGINT +++

Acroread might be workable too, although it seems to have poked all the fonts when first started up. Mose sensible output the second time, although pop-ups/non-render stuff gets in the way.

See also direct forced font analysis.

[09:14:03] [anj@li15-203 /home/anj]$ strace -e trace=open -o render.txt pdftoppm -r 72 SimulationPaper.pdf pages [09:13:59] [anj@li15-203 /home/anj]$ more render.txt | grep ont open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4

Consistent with pdffonts SimulationPaper.pdf (Times-Roman, Symbol).


* Use packages strace and lsof to inspect dependencies during rendering. See http://www.netadmintools.com/art353.html strace -e trace=open -o strace-file.txt xpdf SimulationPaper.pdf Have to page through, and then Symbol and Times New Roman (standard) crop up: [08:08:18] [anj@li15-203 /home/anj]$ diff strace-file.txt strace-nofile.txt | grep -v tmp 110,158c110,111 &lt; open("SimulationPaper.pdf", O_RDONLY|O_LARGEFILE) = 4 &lt; open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 5 &lt; open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 5 &lt; open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 5 &lt; open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 5 --- &gt; --- SIGINT (Interrupt) @ 0 (0) --- &gt; +++ killed by SIGINT +++

Acroread might be workable too, although it seems to have poked all the fonts when first started up. Mose sensible output the second time, although pop-ups/non-render stuff gets in the way.

See also direct forced font analysis.

[09:14:03] [anj@li15-203 /home/anj]$ strace -e trace=open -o render.txt pdftoppm -r 72 SimulationPaper.pdf pages [09:13:59] [anj@li15-203 /home/anj]$ more render.txt | grep ont open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/n021003l.pfb", O_RDONLY) = 4 open("/usr/share/fonts/type1/gsfonts/s050000l.pfb", O_RDONLY) = 4

Consistent with pdffonts SimulationPaper.pdf (Times-Roman, Symbol).

NOTE: Switched from 64-bit to 32-bit to ensure I was using Adobe's distribution, but made little different to the results.

anj@debian:~/scape/pc-cc-strender/debian-strace$ pdffonts /mnt/hgfs/Shared/Z39-87-2006.pdf 
name                                 type              emb sub uni object ID
------------------------------------ ----------------- --- --- --- ---------
Arial-BoldMT                         TrueType          no  no  no     386  0
ArialMT                              TrueType          no  no  no     387  0
TimesNewRomanPSMT                    TrueType          no  no  no     340  0
Arial-ItalicMT                       TrueType          no  no  no     342  0
BBNPHD+SymbolMT                      CID TrueType      yes yes yes    344  0
Arial-BoldItalicMT                   TrueType          no  no  no     348  0

On linux, reads DejaVu fonts but quickly closes and uses Adobe instead. I suspect the DejaVu fonts are embedded in the UI, brought in via glib.

anj@debian:/mnt/hgfs/Shared$ grep open acroread-z39.87.log | grep -v ENOTDIR | grep -i "\.ttf"
3687  open("/usr/share/fonts/truetype/ttf-dejavu/DejaVuSans.ttf", O_RDONLY) = 6
3687  open("/usr/share/fonts/truetype/ttf-dejavu/DejaVuSans-Bold.ttf", O_RDONLY) = 13
anj@debian:/mnt/hgfs/Shared$ grep open acroread-z39.87.log | grep -v ENOTDIR | grep -i "\.pfb"
3687  open("/opt/Adobe/Reader9/Reader/../Resource/Font/ZX______.PFB", O_RDONLY) = 13


Convert on Debian.
/usr/share/fonts/type1/gsfonts/n019004l.pfb NimbusSanL-Bold
/usr/share/fonts/type1/gsfonts/n019003l.pfb NimbusSanL-Regu
/usr/share/fonts/type1/gsfonts/n019023l.pfb NimbusSanL-ReguItal
/usr/share/fonts/type1/gsfonts/n019024l.pfb NimbusSanL-BoldItal
/usr/share/fonts/type1/gsfonts/n021003l.pfb NimbusRomNo9L-Regu

Convert, gs does something odd. It translates ArialMT to Nimbus, and then ignores the fonts installed by libgs8 and uses the type1 fonts from the gsfonts package instead.

/usr/share/ghostscript/8.71/Resource/Init/Fontmap.GS
Above only maps ArialMT to Arial family.
/usr/share/ghostscript/8.71/Resource/Init/pdf_font.ps
Maps Arial to Helvetica
/var/lib/ghostscript/fonts/Fontmap
Maps Helvetica to Nimbus and then Nimbus to Type1 font files.


