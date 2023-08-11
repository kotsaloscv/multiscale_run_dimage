FROM cnsoist/steps:5.0.0_beta
MAINTAINER Christos KOTSALOS <christos.kotsalos@epfl.ch>

# Check that MPI works fine
ADD test_parallelism.tar.gz /opt/src/

RUN cd /opt/src \
 && git clone -b main https://github.com/BlueBrain/AstroVascPy.git bloodflow \
 && cd bloodflow \
 && sed -i 's/"mpi4py"/#"mpi4py"/g' setup.py \
 && sed -i 's/"h5py"/#"h5py"/g' setup.py \
 && sed -i 's/"scipy"/#"scipy"/g' setup.py \
 && sed -i 's/"cython"/#"cython"/g' setup.py \
 && sed -i 's/"matplotlib"/#"matplotlib"/g' setup.py \
 && sed -i 's/"libsonata"/#"libsonata"/g' setup.py \
 && sed -i 's/"morphio"/#"morphio"/g' setup.py \
 && /opt/conda/bin/pip install git+https://github.com/BlueBrain/libsonata.git@v0.1.22 \
 && /opt/conda/bin/pip install git+https://github.com/BlueBrain/MorphIO.git@v3.3.5 \
 && /opt/conda/bin/pip install -e .
ENV BACKEND_SOLVER_BFS='scipy'
ENV DEBUG_BFS=0
ENV VERBOSE_BFS=0

ARG JULIA_MAJOR=1.6
ARG JULIA_MINOR=7
RUN cd /usr/local/ \
 && mkdir julia \
 && wget https://julialang-s3.julialang.org/bin/linux/x64/${JULIA_MAJOR}/julia-${JULIA_MAJOR}.${JULIA_MINOR}-linux-x86_64.tar.gz \
 && tar zxf julia-${JULIA_MAJOR}.${JULIA_MINOR}-linux-x86_64.tar.gz -C julia --strip-components=1 \
 && rm julia-${JULIA_MAJOR}.${JULIA_MINOR}-linux-x86_64.tar.gz
ENV PATH "/usr/local/julia/bin:$PATH"

ADD julia_environment.tar.gz /usr/local/julia/
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

ADD common-2.8.0.tar.gz /opt/src/
RUN cd /opt/src \
 && git clone --recursive -b main https://github.com/BlueBrain/neurodamus.git neurodamus-py \
 && cd neurodamus-py \
 && cp /opt/src/common-2.8.0/hoc/* /opt/src/neurodamus-py/core/hoc \
 && /opt/conda/bin/pip install .
ENV HOC_LIBRARY_PATH "/opt/src/neurodamus-py/core/hoc:$HOC_LIBRARY_PATH"

RUN cd /opt/src \
 && apt-get --yes -qq update \
 && apt-get --yes -qq install bison flex \
 && git clone --recursive --depth 1 --branch 8.2.2 https://github.com/neuronsimulator/nrn.git \
 && cd nrn && mkdir build && cd build \
 && cmake .. -DNRN_ENABLE_MPI=ON -DNRN_ENABLE_INTERVIEWS=OFF -DNRN_ENABLE_RX3D=OFF -DCMAKE_INSTALL_PREFIX=`pwd`/install -DNRN_ENABLE_CORENEURON=OFF -DCORENRN_ENABLE_OPENMP=OFF -DNRN_ENABLE_TESTS=OFF -DPYTHON_EXECUTABLE=$(which python3) \
 && make -j 4 && make -j 4 install
ENV PATH "/opt/src/nrn/build/install/bin:$PATH"
ENV PYTHONPATH "/opt/src/nrn/build/install/lib/python:$PYTHONPATH"

RUN cd /opt/src \
 && apt-get --yes -qq update \
 && apt-get --yes -qq install libboost-all-dev \
 && git clone --recursive --depth 1 --branch v0.1.22 https://github.com/BlueBrain/libsonata.git \
 && cd libsonata \
 && mkdir build && cd build \
 && cmake -DCMAKE_BUILD_TYPE=Release -DEXTLIB_FROM_SUBMODULES=ON -DCMAKE_INSTALL_PREFIX="./install" .. \
 && make -j 4 && make -j 4 install

ADD neurodamus_neocortex_multiscale_mod_full.tar.gz /opt/src/
RUN cd /opt/src \
 && /opt/conda/bin/conda install -c conda-forge ncurses \
 && nrnivmodl -incflags "-DDISABLE_REPORTINGLIB -I/opt/conda/include/" -loadflags "-L/opt/conda/lib -L/opt/conda/lib64 -lmpi -lhdf5 -L/opt/src/libsonata/build/install/lib -lsonata" neurodamus_neocortex_multiscale_mod_full/
ENV PATH "/opt/src/x86_64:$PATH"
ENV LD_LIBRARY_PATH "/opt/src/x86_64:/opt/src/libsonata/build/install/lib:/opt/conda/lib:/opt/conda/lib64:$LD_LIBRARY_PATH"
ENV NRNMECH_LIB_PATH "/opt/src/x86_64/libnrnmech.so"
ENV HDF5_DISABLE_VERSION_CHECK=1

RUN /opt/conda/bin/pip install gdown
