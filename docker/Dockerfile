# SPDX-License-Identifier: CC-BY-4.0
# SPDX-FileCopyrightText: © 2021 Arm Limited <kristof.beyls@arm.com>
# syntax=docker/dockerfile:1
FROM debian:trixie
# 1. install pandoc and texlive dependencies
#    Also install git as we need that for computing version numbers.
#    Also install librsvg2-bin to enable pandoc to convert SVG images to PDF.
#    Also install python as the pandoc-fignos filter depends on it.
RUN apt-get update && \
    DEBIAN_FRONTEND="noninteractive" apt-get install -y \
            wget perl pandoc make git librsvg2-bin python3-pip && \
    rm -rf /var/lib/apt/lists/*
# 2. Install LaTeX directly from stable Texlive 2023 final repo
# (We could alternatively install a base Texlive using "apt-get install
#  texlive-latex-extra", and then add extra packages using tlmgr.
#  The drawbacks of that approach are:
#  - bigger image size over 1GB versus 443MB;
#  - using 2 package distribution systems to install LaTeX functionality:
#    apt-get and texlive, rather than just texlive.
#  The drawback from installing directly from Texlive is a bit more
#  complexity, explained directly below.)
# The below downloads the stable Texlive 2023 distribution.
# The options written to texlive.profile indicate to do a minimal install,
# which keeps the docker image size smaller. It also tells not to install
# documentation or source files (which we are very unlikely to need in a
# docker container).
# The texlive distribution will be installed in /texlive/2023/bin/{target triple}
# A symbolic link to that path is added so that in the next Docker command, we
# can add it to the PATH variable. Docker ENV commands do not allow using wildcards,
# hence we need to create a link with a stable name (/texlive/bin) to make the
# ENV PATH=... command work.
RUN cd /root && \
    wget https://ftp.tu-chemnitz.de/pub/tug/historic/systems/texlive/2023/install-tl-unx.tar.gz && \
    mkdir install-tl-unx && \
    tar -zxvf install-tl-unx.tar.gz -C install-tl-unx --strip-components=1 && \
    echo "selected_scheme scheme-minimal" >> texlive.profile && \
    echo "tlpdbopt_install_docfiles 0" >> texlive.profile && \
    echo "tlpdbopt_install_srcfiles 0" >> texlive.profile && \
    echo "tlpdbopt_autobackup 0" >> texlive.profile && \
    mkdir texlive && \
    TEXLIVE_INSTALL_PREFIX=/texlive ./install-tl-unx/install-tl \
        -profile texlive.profile \
        -repository https://ftp.tu-chemnitz.de/pub/tug/historic/systems/texlive/2023/tlnet-final && \
    ln -s /texlive/2023/bin/* /texlive/bin
# 3. Add texlive binaries to PATH so they can be used easily.
ENV PATH=/texlive/bin:$PATH
# 4. install latex too (the minimal scheme selected above does not install latex)
RUN tlmgr install latex latex-bin latexmk
# 5. install extra latex packages needed for pandoc:
RUN tlmgr install amsfonts amsmath lm unicode-math iftex listings fancyvrb \
      booktabs graphics hyperref xcolor ulem geometry setspace babel upquote \
      microtype parskip xurl bookmark footnotehyper mdwtools kvoptions \
      etoolbox pdftexcmds infwarerr grffile cleveref caption multirow
# 6. install extra packages needed for the book specifically:
RUN tlmgr install todonotes epstopdf-pkg bclogo pstricks pst-grad pst-coil \
      pst-node pst-blur mdframed zref needspace mptopdf

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
