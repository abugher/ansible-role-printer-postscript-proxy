This role enables a host to print via an elaborate game of mousetrap.
At one end of the Rube Goldberg machine, there is a physical printer,
driven by a closed driver.  At the other end is a print queue reachable
via IPP and driven by PostScript, which is all but to say if your device
can print, it can print to this queue.  In the middle we have the
cups-pdf driver, which is configured to fling the output of every print
job into a certain file location on the server, which is watched by
pdfprintd, which in turn flings the file at the default print queue,
which is the physical printer.

CUPS may have a simpler way to expose printers to the network without
requiring hardware-specific drivers, these days.
