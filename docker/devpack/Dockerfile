################################################################################
# export REBEL_PYPI_USERNAME=XXXXXXXX REBEL_PYPI_PASSWORD=XXXXXXXX REBEL_R18S_PYPI_USERNAME=XXXXXXXX REBEL_R18S_PYPI_PASSWORD=XXXXXXXX
# export REPO=docker.e2e.sw2.rebellions.in/rebellions-sw/devpack TAG=r18s8
# docker build -t           "$REPO:$TAG" --target full      --secret id=REBEL_PYPI_USERNAME --secret id=REBEL_PYPI_PASSWORD --secret id=REBEL_R18S_PYPI_USERNAME --secret id=REBEL_R18S_PYPI_PASSWORD /
# docker build -t       "$REPO-app:$TAG" --target app       --secret id=REBEL_PYPI_USERNAME --secret id=REBEL_PYPI_PASSWORD --secret id=REBEL_R18S_PYPI_USERNAME --secret id=REBEL_R18S_PYPI_PASSWORD /
# docker build -t "$REPO-toolchain:$TAG" --target toolchain --secret id=REBEL_PYPI_USERNAME --secret id=REBEL_PYPI_PASSWORD --secret id=REBEL_R18S_PYPI_USERNAME --secret id=REBEL_R18S_PYPI_PASSWORD /
# docker build -t    "$REPO-python:$TAG" --target python    --secret id=REBEL_PYPI_USERNAME --secret id=REBEL_PYPI_PASSWORD --secret id=REBEL_R18S_PYPI_USERNAME --secret id=REBEL_R18S_PYPI_PASSWORD /
# docker build -t    "$REPO-driver:$TAG" --target driver    --secret id=REBEL_PYPI_USERNAME --secret id=REBEL_PYPI_PASSWORD --secret id=REBEL_R18S_PYPI_USERNAME --secret id=REBEL_R18S_PYPI_PASSWORD /

ARG PLATFORM=linux/amd64
FROM --platform=$PLATFORM ubuntu:22.04 AS base

ARG WORKDIR=/workspace
WORKDIR "$WORKDIR"

ARG ATOM_DRIVER_VERSION=2.0.1
ARG RBLN_CCL_VERSION="$ATOM_DRIVER_VERSION"
ARG BCM_DRIVER_VERSION=233.1.135.7
ARG BCM_LIBBNXT_RE_VERSION=233.0.152.2
ARG LLVM_VERSION=20.1.0-rbln-251111
ARG OPTIMUM_RBLN_VERSION=0.7.5rc3.dev3+g0e82798
ARG RAY_VERSION=2.46.0
ARG REBEL_COMPILER_VERSION=0.7.5.dev166+g8d596379
ARG TORCHVISION_VERSION=0.21.0+cpu
ARG TORCH_VERSION=2.6.0+cpu
ARG VLLM_RBLN_VERSION=0.8.0rc0
ARG REBEL_DEPENDENCY_DIR=/mnt/shared_data/users/daekyeong/nas_data/rebel_dependency
ARG ATOM_DRIVER_PACKAGE_DIR=/mnt/shared_data/users/daekyeong/nas_data/pkg
ARG RBLN_CCL_PACKAGE_DIR=/mnt/shared_data/users/daekyeong/nas_data/pkg

LABEL ATOM_DRIVER_VERSION="$ATOM_DRIVER_VERSION"
LABEL RBLN_CCL_VERSION="$RBLN_CCL_VERSION"
LABEL BCM_DRIVER_VERSION="$BCM_DRIVER_VERSION"
LABEL BCM_LIBBNXT_RE_VERSION="$BCM_LIBBNXT_RE_VERSION"
LABEL LLVM_VERSION="$LLVM_VERSION"
LABEL OPTIMUM_RBLN_VERSION="$OPTIMUM_RBLN_VERSION"
LABEL RAY_VERSION="$RAY_VERSION"
LABEL REBEL_COMPILER_VERSION="$REBEL_COMPILER_VERSION"
LABEL TORCHVISION_VERSION="$TORCHVISION_VERSION"
LABEL TORCH_VERSION="$TORCH_VERSION"
LABEL VLLM_RBLN_VERSION="$VLLM_RBLN_VERSION"
LABEL REBEL_DEPENDENCY_DIR="$REBEL_DEPENDENCY_DIR"
LABEL ATOM_DRIVER_PACKAGE_DIR="$ATOM_DRIVER_PACKAGE_DIR"
LABEL RBLN_CCL_PACKAGE_DIR="$RBLN_CCL_PACKAGE_DIR"


RUN sed -i 's/archive.ubuntu.com/mirror.kakao.com/g; s/kr.archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list && \
  sed -i 's/security.ubuntu.com/mirror.kakao.com/g; s/kr.security.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list

################################################################################
FROM base AS bnxt

RUN PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive apt update && \
  apt install -y \
    autoconf \
    automake \
    build-essential \
    gcc \
    iproute2 \
    kmod \
    libibverbs-dev \
    librdmacm-dev \
    librdmacm1 \
    libtool \
    pciutils \
    python3 \
    systemd \
    wget && \
  apt clean && \
  rm -rf /var/lib/apt/lists/*

RUN wget "https://docs.broadcom.com/docs-and-downloads/ethernet-network-adapters/NXE/BRCM_$BCM_DRIVER_VERSION/bcm_$BCM_DRIVER_VERSION.tar.gz" && \
  tar -xvzf "bcm_$BCM_DRIVER_VERSION.tar.gz" && \
  tar -C /tmp -xf "$WORKDIR/bcm_$BCM_DRIVER_VERSION/drivers_linux/bnxt_rocelib/libbnxt_re-$BCM_LIBBNXT_RE_VERSION.tar.gz" && \
  cd "/tmp/libbnxt_re-$BCM_LIBBNXT_RE_VERSION" && \
  sh autogen.sh && \
  ./configure && \
  make clean && \
  make && \
  make install


FROM base AS atom

ARG ATOM_DRIVER_PACKAGE="atom_internal_release_${ATOM_DRIVER_VERSION}_amd64.deb"
COPY "$ATOM_DRIVER_PACKAGE_DIR/$ATOM_DRIVER_PACKAGE" "/atom/$ATOM_DRIVER_PACKAGE"
RUN dpkg-deb --vextract "/atom/$ATOM_DRIVER_PACKAGE" /atom/ && rm -f "/atom/$ATOM_DRIVER_PACKAGE"

ARG RBLN_CCL_PACKAGE="librbln-ccl2_${RBLN_CCL_VERSION}_amd64.deb"
ARG RBLN_CCL_DEV_PACKAGE="librbln-ccl-dev_${RBLN_CCL_VERSION}_amd64.deb"
COPY "$RBLN_CCL_PACKAGE_DIR/$RBLN_CCL_PACKAGE" "/atom/$RBLN_CCL_PACKAGE"
COPY "$RBLN_CCL_PACKAGE_DIR/$RBLN_CCL_DEV_PACKAGE" "/atom/$RBLN_CCL_DEV_PACKAGE"
RUN dpkg-deb --vextract "/atom/$RBLN_CCL_PACKAGE" /atom/ && rm -f "/atom/$RBLN_CCL_PACKAGE"
RUN dpkg-deb --vextract "/atom/$RBLN_CCL_DEV_PACKAGE" /atom/ && rm -f "/atom/$RBLN_CCL_DEV_PACKAGE"


################################################################################
FROM base AS driver

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["/bin/bash"]

RUN PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive apt update && \
  apt install -y \
    tini \
    libbz2-1.0 \
    libcmocka0 \
    libdrm2 \
    libibverbs1 \
    librdmacm1 \
    rdma-core \
    zlib1g && \
  apt clean && \
  rm -rf /var/lib/apt/lists/*

COPY --from=bnxt /usr/local/lib/libbnxt_re-rdmav34.so /usr/local/lib/libbnxt_re-rdmav34.so
RUN mv /usr/lib/x86_64-linux-gnu/libibverbs/libbnxt_re-rdmav34.so /usr/lib/x86_64-linux-gnu/libibverbs/libbnxt_re-rdmav34.so.inbox && \
  ln -s /usr/local/lib/libbnxt_re-rdmav34.so /usr/local/lib/libbnxt_re.so && \
  ldconfig

COPY --from=atom /atom/etc/ /etc/
COPY --from=atom /atom/opt/ /opt/
COPY --from=atom /atom/usr/ /usr/
COPY --from=atom /atom/lib/ /usr/


FROM driver AS python

RUN PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive apt update && \
  apt install -y \
    python3-pip \
    python3-distutils \
    python3-venv \
    python-is-python3 && \
  apt clean && \
  rm -rf /var/lib/apt/lists/*


FROM python AS app

RUN PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive apt update && \
  apt install -y \
    wget rsync ssh && \
  apt clean && \
  rm -rf /var/lib/apt/lists/*

RUN --mount=type=secret,id=REBEL_PYPI_USERNAME,env=REBEL_PYPI_USERNAME \
  --mount=type=secret,id=REBEL_PYPI_PASSWORD,env=REBEL_PYPI_PASSWORD \
  pip install poetry==2.0.1 --no-cache && \
  poetry init -n --python '>=3.10,<3.14' && \
  poetry source add rbln-r18s --priority supplemental -- https://pypi.e2e.sw2.rebellions.in/simple && \
  poetry source add rbln --priority supplemental -- https://pypi.rebellions.in/simple && \
  poetry source add torch --priority supplemental -- https://download.pytorch.org/whl/cpu && \
  poetry config http-basic.rbln-r18s "$REBEL_R18S_PYPI_USERNAME" "$REBEL_R18S_PYPI_PASSWORD" && \
  poetry config http-basic.rbln "$REBEL_PYPI_USERNAME" "$REBEL_PYPI_PASSWORD" && \
  poetry config virtualenvs.create false && \
  poetry add --no-cache \
    "ray[serve]==$RAY_VERSION" \
    "rebel-compiler==$REBEL_COMPILER_VERSION" \
    "vllm-rbln==$VLLM_RBLN_VERSION" \
    "optimum-rbln==$OPTIMUM_RBLN_VERSION" \
    "torchvision==$TORCHVISION_VERSION" \
    "torch==$TORCH_VERSION" && \
  poetry sync --no-root --no-cache


FROM python AS toolchain

RUN PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive apt update && \
  apt install -y \
    ccache \
    g++-12 \
    gcc-12 \
    libbz2-dev \
    libcmocka-dev \
    libdrm-dev \
    libibverbs-dev \
    libnuma-dev \
    librdmacm-dev \
    rdma-core \
    software-properties-common \
    zlib1g-dev && \
  apt clean && \
  rm -rf /var/lib/apt/lists/*

RUN PYTHONUNBUFFERED=1 DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:ubuntu-toolchain-r/test && \
  apt update && \
  apt install -y \
    gcc-13 \
    g++-13 && \
  apt clean && \
  rm -rf /var/lib/apt/lists/*

RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 10 --slave /usr/bin/g++ g++ /usr/bin/g++-13 && \
  update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 20 --slave /usr/bin/g++ g++ /usr/bin/g++-12

COPY "$REBEL_DEPENDENCY_DIR/llvm-$LLVM_VERSION/" /tools/lib/rebel_dependency/llvm/
COPY "$REBEL_DEPENDENCY_DIR/GCC-12.1.0/" /tools/lib/rebel_dependency/GCC-12.1.0/
COPY "$REBEL_DEPENDENCY_DIR/gtest/" /tools/lib/rebel_dependency/gtest/


FROM app AS full

COPY --from=toolchain /tools/lib/ /tools/lib/
