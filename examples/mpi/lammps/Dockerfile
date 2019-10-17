# ch-test-scope: full
FROM openmpi
WORKDIR /usr/src

# Packages for building.
RUN dnf install -y --setopt=install_weak_deps=false \
    cmake \
    patch \
    python3-devel \
    python3-pip \
    python3-setuptools

# Building mpi4py from source to ensure it is built against our MPI build
# Building numpy from source to work around issues seen on Aarch64 systems
RUN pip3 install --no-binary :all: mpi4py numpy
# Build LAMMPS.
ENV LAMMPS_VERSION 19Sep19
RUN wget -nv http://lammps.sandia.gov/tars/lammps-$LAMMPS_VERSION.tar.gz
RUN tar xf lammps-$LAMMPS_VERSION.tar.gz
RUN mkdir lammps-${LAMMPS_VERSION}.build
RUN    cd lammps-${LAMMPS_VERSION}.build \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr \
             -DCMAKE_BUILD_TYPE=Release \
             -DBUILD_MPI=yes \
             -DBUILD_LIB=on \
             -DBUILD_SHARED_LIBS=on \
             -DPKG_DIPOLE=yes \
             -DPKG_KSPACE=yes \
             -DPKG_POEMS=yes \
             -DPKG_PYTHON=yes \
             -DPKG_USER-REAXC=yes \
             -DPKG_USER-MEAMC=yes \
             -DLAMMPS_MACHINE=mpi \
          ../lammps-${LAMMPS_VERSION}/cmake \
    && make -j $(getconf _NPROCESSORS_ONLN) install
RUN ln -s /usr/src/lammps-${LAMMPS_VERSION}/ /lammps
# Patch in.melt instead of including in.large.melt because LAMMPS is GPL.
# Patch simple.py to uncomment mpi4py calls and disable file output.
COPY melt-and-simple.patch /lammps
RUN patch -d / -p1 < /lammps/melt-and-simple.patch
