#  _         _____   __  __
# | |    __ |_   _|__\ \/ /
# | |   / _` || |/ _ \\  /
# | |__| (_| || |  __//  \
# |_____\__,_||_|\___/_/\_\
#
ARG pandoc_version=edge
FROM pandoc/core:${pandoc_version}-ubuntu

# NOTE: to maintainers, please keep this listing alphabetical.
RUN apt-get -q --no-allow-insecure-repositories update \
  && DEBIAN_FRONTEND=noninteractive \
     apt-get install --assume-yes --no-install-recommends \
        fontconfig \
        gnupg \
        gzip \
        libfontconfig1 \
        libfreetype6 \
        perl \
        tar \
        wget \
        xzdec \
  && rm -rf /var/lib/apt/lists/*

# Installer scripts and config
COPY common/latex/texlive.profile    /root/texlive.profile
COPY common/latex/install-texlive.sh /root/install-texlive.sh
COPY common/latex/packages.txt       /root/packages.txt

# TeXLive binaries location
ARG texlive_bin="/opt/texlive/texdir/bin"

# TeXLive version to install (leave empty to use the latest version).
ARG texlive_version=

# TeXLive mirror URL (leave empty to use the default mirror).
ARG texlive_mirror_url=

# Modify PATH environment variable, prepending TexLive bin directory
ENV PATH="${texlive_bin}/default:${PATH}"

RUN cd /root && \
    TEXLIVE_ARCH="$(uname -m)-$(uname -s | tr '[:upper:]' '[:lower:]')" && \
    mkdir -p ${texlive_bin} && \
    ln -sf "${texlive_bin}/${TEXLIVE_ARCH}" "${texlive_bin}/default" && \
# Request musl precompiled binary access
    echo "binary_${TEXLIVE_ARCH} 1" >> /root/texlive.profile && \
    ( \
     [ -z "$texlive_version"    ] || printf '-t\n%s\n"' "$texlive_version"; \
     [ -z "$texlive_mirror_url" ] || printf '-m\n%s\n' "$texlive_mirror_url" \
    ) | xargs /root/install-texlive.sh && \
    sed -e 's/ *#.*$//' -e '/^ *$/d' /root/packages.txt | \
        xargs tlmgr install && \
    rm -f /root/texlive.profile \
          /root/install-texlive.sh \
          /root/packages.txt && \
    TERM=dumb luaotfload-tool --update && \
    chmod -R o+w /opt/texlive/texdir/texmf-var && \
    mkdir -p ${texlive_bin} && \
    ln -sf "${texlive_bin}/${TEXLIVE_ARCH}" "${texlive_bin}/default"

WORKDIR /data
