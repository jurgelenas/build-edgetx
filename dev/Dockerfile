ARG OS_CODENAME=bionic

FROM ubuntu:${OS_CODENAME}

# need to redeclare inside FROM with no value assignment
# refer https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
ARG OS_CODENAME

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update -qq && \
        apt-get install --yes \
        software-properties-common \
        build-essential gpg wget \
        libfox-1.6-dev libclang-6.0-dev libsdl1.2-dev

RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | \
        gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null && \
        echo "deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ ${OS_CODENAME} main" | \
        tee /etc/apt/sources.list.d/kitware.list >/dev/null && \
        apt-get update -qq && apt-get install --yes cmake

RUN add-apt-repository ppa:git-core/ppa --yes && \
        apt-get update -qq && \
        apt-get install --yes -qq \
        git wget zip unzip \
        python3-pip

RUN python3 -m pip install \
        asciitree jinja2 \
        pillow==7.2.0 \
        clang==6.0.0

# Qt installation adapted from
# see https://github.com/miurahr/aqtinstall/
# see https://github.com/vslotman/docker-aqtinstall

RUN python3 -m pip install \
        aqtinstall

ARG QT_VERSION=5.12.9
# if modules use syntax -m MODULE [MODULE]
ARG QT_MODULES=
ARG QT_HOST=linux
ARG QT_TARGET=desktop
ARG QT_INSTALL_DIR=/opt/qt

# Install some dependencies required by Qt libs
RUN apt-get install --yes -qq \
        libxkbcommon-x11-0 gstreamer1.0-plugins-base

RUN aqt install --outputdir ${QT_INSTALL_DIR} ${QT_MODULES} ${QT_VERSION} ${QT_HOST} ${QT_TARGET}

ARG QT_BASE_DIR=${QT_INSTALL_DIR}/${QT_VERSION}/gcc_64

ENV PATH=${QT_BASE_DIR}/bin:$PATH
ENV QT_PLUGIN_PATH=${QT_BASE_DIR}/plugins/
ENV QML_IMPORT_PATH=${QT_BASE_DIR}/qml/
ENV QML2_IMPORT_PATH=${QT_BASE_DIR}/qml/
ENV LD_LIBRARY_PATH=${QT_BASE_DIR}/lib:$LD_LIBRARY_PATH
ENV PKG_CONFIG_PATH=${QT_BASE_DIR}/lib/pkgconfig:$PKG_CONFIG_PATH

# Workaround for "libQt5Core.so.5 not found" issue
# see https://www.gitmemory.com/issue/Microsoft/WSL/3023/488329451
RUN strip --remove-section=.note.ABI-tag ${QT_BASE_DIR}/lib/libQt5Core.so.${QT_VERSION}

RUN wget --quiet https://developer.arm.com/-/media/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2 -O - \
        | tar -xj -C /opt

ENV PATH=/opt/gcc-arm-none-eabi-10-2020-q4-major/bin/:${PATH}

VOLUME ["/src"]

COPY entrypoint.sh /opt

ENTRYPOINT ["/opt/entrypoint.sh"]
