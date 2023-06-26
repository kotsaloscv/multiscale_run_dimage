FROM --platform=linux/amd64 cnsoist/steps/5.0.0a
MAINTAINER Christos KOTSALOS <christos.kotsalos@epfl.ch>

ADD bloodflow /opt/src/bloodflow
RUN cd /opt/src/bloodflow \
 && sed -i 's/"h5py"/#"h5py"/g' setup.py \
 && /opt/conda/bin/pip install --index-url https://bbpteam.epfl.ch/repository/devpi/bbprelman/dev/+simple/ -e .
ENV BACKEND_SOLVER_BFS='petsc'
ENV DEBUG_BFS=0
ENV VERBOSE_BFS=0

ADD test_parallelism /opt/src/test_parallelism

ARG JULIA_MAJOR=1.6
ARG JULIA_MINOR=7
RUN cd /usr/local/ \
 && mkdir julia \
 && wget https://julialang-s3.julialang.org/bin/linux/x64/${JULIA_MAJOR}/julia-${JULIA_MAJOR}.${JULIA_MINOR}-linux-x86_64.tar.gz \
 && tar zxf julia-${JULIA_MAJOR}.${JULIA_MINOR}-linux-x86_64.tar.gz -C julia --strip-components=1 \
 && rm julia-${JULIA_MAJOR}.${JULIA_MINOR}-linux-x86_64.tar.gz
ENV PATH "/usr/local/julia/bin:$PATH"

ADD julia_environment /usr/local/julia/julia_environment
ENV JULIA_DEPOT_PATH=/usr/local/julia/.julia
ENV JULIA_PROJECT=/usr/local/julia/julia_environment
RUN if [ -d "${JULIA_DEPOT_PATH}" ]; then \
      echo "julia packages already set"; \
    else \
      echo "setup julia"; \
      mkdir ${JULIA_DEPOT_PATH}; \
      if [ -d ${JULIA_PROJECT} ]; then \
        julia -e 'using Pkg; Pkg.instantiate(; verbose=true)'; \
      else \
        julia -e 'using Pkg; Pkg.add("IJulia")'; \
        julia -e 'using Pkg; Pkg.add("DifferentialEquations")'; \
        julia -e 'using Pkg; Pkg.add("DiffEqBase")'; \
        julia -e 'using Pkg; Pkg.add("ParameterizedFunctions")'; \
        julia -e 'using Pkg; Pkg.add("StaticArrays")'; \
        julia -e 'using Pkg; Pkg.add("RecursiveArrayTools")'; \
        julia -e 'using Pkg; Pkg.add("PyCall");Pkg.build("PyCall")'; \
      fi \
    fi \
 && /opt/conda/bin/pip install diffeqpy julia

ADD neurodamus-py /opt/src/neurodamus-py
RUN cd /opt/src/neurodamus-py \
 && touch /opt/conda/lib/python3.9/site-packages/numpy-1.24.3.dist-info/METADATA \
 && sed -i '29 s/^/#/' /opt/conda/lib/python3.9/site-packages/numpy/distutils/log.py \
 && /opt/conda/bin/pip install . \
 && /opt/conda/bin/pip install git+https://github.com/BlueBrain/libsonata
ENV HOC_LIBRARY_PATH "/opt/src/neurodamus-py/core/hoc:$HOC_LIBRARY_PATH"

RUN cd /opt/src \
 && apt-get --yes -qq update \
 && apt-get --yes -qq install bison flex \
 && git clone https://github.com/neuronsimulator/nrn.git \
 && cd nrn && mkdir build && cd build \
 && cmake .. -DNRN_ENABLE_INTERVIEWS=OFF -DNRN_ENABLE_RX3D=OFF -DCMAKE_INSTALL_PREFIX=`pwd`/install -DNRN_ENABLE_CORENEURON=OFF -DCORENRN_ENABLE_OPENMP=OFF -DNRN_ENABLE_TESTS=OFF -DPYTHON_EXECUTABLE=$(which python3) \
 && make -j 4 && make -j 4 install
ENV PATH "/opt/src/nrn/build/install/bin:$PATH"
ENV PYTHONPATH "/opt/src/nrn/build/install/lib/python:$PYTHONPATH"

ADD synapse-tool /opt/src/synapse-tool
RUN cd /opt/src/synapse-tool \
 && apt-get --yes -qq update \
 && apt-get --yes -qq install libboost-all-dev \
 && git clone --depth 1 --branch v2.7.1 https://github.com/BlueBrain/HighFive.git \
 && sed -i "s/find_package(HighFive REQUIRED)/add_subdirectory(HighFive)/g" CMakeLists.txt \
 && git clone --depth 1 --branch v0.1.20 https://github.com/BlueBrain/libsonata.git --recursive \
 && cd libsonata && mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE=Release -DEXTLIB_FROM_SUBMODULES=ON -DCMAKE_INSTALL_PREFIX="./install" .. && make -j 4 && make -j 4 install \
 && cd /opt/src/synapse-tool \
 && mkdir build && cd build && cmake -DSYNTOOL_WITH_MPI=ON -DCMAKE_INSTALL_PREFIX="./install" -DCMAKE_PREFIX_PATH=/opt/src/synapse-tool/libsonata/build/install .. \
 && make -j 4 && make -j 4 install

ADD neurodamus_neocortex_multiscale_mod_full /opt/src/neurodamus_neocortex_multiscale_mod_full
RUN cd /opt/src \
 && /opt/conda/bin/conda install -c conda-forge ncurses \
 && nrnivmodl -incflags "-DENABLE_SYNTOOL=1 -DDISABLE_REPORTINGLIB -I/opt/conda/include/ -I/opt/src/synapse-tool/build/install/include -I/opt/src/synapse-tool/build/install/include" -loadflags "-L/opt/conda/lib -L/opt/conda/lib64 -lmpi -L/opt/src/synapse-tool/build/install/lib -lsyn2 -L/opt/src/synapse-tool/libsonata/build/install/lib -lsonata" neurodamus_neocortex_multiscale_mod_full/
ENV PATH "/opt/src/x86_64:$PATH"
ENV LD_LIBRARY_PATH "/opt/src/synapse-tool/libsonata/build/install/lib:/opt/src/synapse-tool/build/install/lib:/opt/conda/lib:/opt/conda/lib64:$LD_LIBRARY_PATH"
ENV HDF5_DISABLE_VERSION_CHECK=1

ADD multiscale_run /opt/src/multiscale_run
ADD ngv_small_circuit/build /opt/src/ngv_small_circuit
