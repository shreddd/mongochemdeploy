FROM girder/girder:latest-py3

# Build CMake
RUN git clone https://github.com/Kitware/CMake.git /cmake && \
  cd /cmake && \
  git checkout v3.9.1 && \
  ./bootstrap && \
  make install

# Build pybind11
RUN git clone https://github.com/pybind/pybind11.git /pybind11/source && \
  mkdir /pybind11/build && \
  mkdir /pybind11/install && \
  cd /pybind11/build && \
  cmake -DCMAKE_INSTALL_PREFIX:PATH=/pybind11/install -DPYBIND11_TEST:BOOL=NO ../source && \
  cmake --build . --target install && \
  rm -rf /pybind11/build && rm -rf /pybind11/source

# Install deps for avogadro
RUN apt-get update && apt-get -y install \
  libeigen3-dev \
  libpython3-dev

# Build avogadrolibs
RUN git clone https://github.com/openchemistry/avogadrolibs.git /avogadrolibs/source && \
  mkdir /avogadrolibs/build && \
  cd /avogadrolibs/build && \
  cmake -DCMAKE_BUILD_TYPE:STRING=Release -DUSE_PYTHON:BOOL=YES -DENABLE_TESTING:BOOL=NO \
    -Dpybind11_DIR:PATH=/pybind11/install/share/cmake/pybind11 -DUSE_LIBSPG:BOOL=NO \
    -DUSE_QT:BOOL=NO -DUSE_OPENGL:BOOL=NO ../source && \
  cmake --build . && \
  mkdir -p /usr/lib/python3.4/site-packages/ && \
  cp /avogadrolibs/build/lib/* /usr/local/lib/python3.4/dist-packages/ && \
  rm -rf /avogadrolibs/source

# Install cumulus
RUN git clone https://github.com/Kitware/cumulus.git /cumulus && \
  pip install --no-cache-dir -r /cumulus/requirements.txt && \
  pip install --no-cache-dir -e /cumulus && \
  girder-install plugin /cumulus/girder/cumulus && \
  girder-install plugin /cumulus/girder/sftp && \
  girder-install plugin /cumulus/girder/newt && \
  girder-install plugin /cumulus/girder/taskflow

# Set the broker URL
RUN sed -i s/localhost/rabbitmq/g /cumulus/cumulus/celery/commonconfig.py

COPY config.json /cumulus/cumulus/conf/config.json

RUN git clone https://github.com/OpenChemistry/openchemistrypy.git /openchemistrypy

# Install deps for pybel
RUN apt-get update && apt-get -y install \
  swig \
  libopenbabel-dev

# Enable proxy support
COPY girder/girder.local.conf /girder/girder/conf/girder.local.cfg

# Install mongochemserver plugin
RUN git clone https://github.com/OpenChemistry/mongochemserver.git /mongochemserver && \
  pip install --no-cache-dir -r /mongochemserver/requirements.txt && \
  girder-install plugin /mongochemserver/girder/molecules

# Install clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
