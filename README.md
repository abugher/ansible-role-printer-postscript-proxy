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

The printer is required to be plugged in, turned on, and awake at
deployment time.  If possible, it would be better to wake the printer as
an automated deployment step.

Add a new printer to the client system, set the make/model to
generic/postscript, and set the URI like so:

    ipp://printer.neuronpointer.net/printers/printer

... or just:

    ipp://printer/printers/printer


The driver packages for my printer are available from here:

    https://support.brother.com/g/b/downloadlist.aspx?c=us_ot&lang=en&prod=hll2300d_us_eu_as&os=128

One package includes some executable files compiled for i386 architecture.  I
adjusted the package to depend on `qemu-user` and `libc6:i386` and changed both
packages to declare compatibility with all architectures, using commands
similar to this set:

    cd tmp
    rm -rf *
    cp /path/to/hll2300dcupswrapper-3.2.0-1.i386.deb ./
    cp /path/to/hll2300dlpr-3.2.0-1.i386.deb ./
    for deb in *.deb; do 
      dir="$(sed 's/\.i386\.deb//' <<< "${deb}")"
      newdeb="${dir}.deb"
      dpkg-deb -R "${deb}" "${dir}"
      sed -i 's/Architecture: i386/Architecture: all/' "${dir}"/DEBIAN/control
      for path in $(find . -exec file {} \; | awk -F : '/ELF/ {print $1}'); do
        mv "${path}" "${path}_real}"
        ln -s /usr/local/bin/qemu-i386-wrapper "${path}"
      done
    done
    echo "Depends: qemu-user, libc6:i386" >> hll2300dlpr-3.2.0-1/DEBIAN/control
    mkdir -p hll2300dlpr-3.2.0-1/usr/local/bin
    echo -e "#!/bin/bash\nqemu-i386 '${0}_real' \"\${@}\"" > hll2300dlpr-3.2.0-1/usr/local/bin/qemu-i386-wrapper
    chmod 0755 hll2300dlpr-3.2.0-1/usr/local/bin/qemu-i386-wrapper
    for deb in *.deb; do 
      dir="$(sed 's/\.i386\.deb//' <<< "${deb}")"
      newdeb="${dir}.deb"
      dpkg-deb --root-owner-group -b "${dir}" "${newdeb}"
    done

This works on `armhf` architecture.
