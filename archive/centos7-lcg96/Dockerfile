FROM cern/cc7-base:latest

LABEL description="CERN CentOS 7 with Acts dependencies from LCG 96"
LABEL maintainer="Paul Gessinger <paul.gessinger@cern.ch>"
# increase whenever any of the RUN commands change
LABEL version="5"

ENV LCG_RELEASE 96
ENV LCG_PLATFORM_BASE x86_64-centos7
ENV LCG_PLATFORM ${LCG_PLATFORM_BASE}-gcc8-opt
# branch/tag/commit reference for lcgcmake. we need a specific version
# to be able to create an RPM-based view.
ENV LCGCMAKE_REF 16176752

# Install all LCG dependencies from the system package manager by using the
# HEP_OSlibs metapackage as described here
#
#     https://gitlab.cern.ch/linuxsupport/rpms/HEP_OSlibs
#
# Add the LCG rpm repository as described here
#
#     https://lcgpackages.web.cern.ch
#
# and use it to install all explicit Acts dependencies from an LCG release.
RUN yum-config-manager --add-repo https://lcgpackages.web.cern.ch/lcgpackages/rpms/etc/yum.repos.d/lcg.repo \
  && yum -y install \
    HEP_OSlibs \
    LCG_${LCG_RELEASE}_gcc_8.2.0_${LCG_PLATFORM_BASE//-/_}.noarch \
    LCG_${LCG_RELEASE}_Boost_1.70.0_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_CMake_3.14.3_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_DD4hep_01_10_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_eigen_3.3.7_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_Geant4_10.05.p01_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_hepmc3_3.1.1_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_ninja_1.9.0_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_pythia8_240_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_ROOT_6.18.00_${LCG_PLATFORM//-/_}.noarch \
    LCG_${LCG_RELEASE}_tbb_2019_U7_${LCG_PLATFORM//-/_}.noarch \
  && yum -y clean all

# Add missing directory link so the view creating script finds the compiler
RUN ln -rsv /opt/lcg/gcc/8.2.0-3fa06 /opt/lcg/gcc/8
# Bugfix for broken setup script
ADD fixed-gcc-setup.sh /opt/lcg/gcc/8.2.0-3fa06/x86_64-centos7/setup.sh

# Create the LCG view. Use a fixed location since there is only one
# release/platform combination. This way, images with different
# configurations (including different compilers) can be easily substituted
# without needing to know the specific contents.
RUN git clone https://gitlab.cern.ch/sft/lcgcmake.git lcgcmake \
  && (cd lcgcmake; git checkout ${LCGCMAKE_REF}) \
  && lcgcmake/cmake/scripts/create_lcg_view.py -v \
    -l /opt/lcg/LCG_${LCG_RELEASE} \
    -p ${LCG_PLATFORM} \
    /opt/lcg_view \
  && rm -rf lcgcmake
