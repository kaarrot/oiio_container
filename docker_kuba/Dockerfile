FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Install required packages
RUN apt-get update && apt-get install -y --no-install-recommends  \
                                        build-essential           \
                                        git                       \
                                        cmake                     \
                                        unzip                     \
                                        python3-dev               \
                                        python3-venv              \
                                        ninja-build               \
                                        && rm -rf /var/lib/apt/lists/*

# When creating a workspace from VScode server running on windows, the editor changes line endings to CRLF
# This break some awk files (ie options.awk in PNG) which is really hard to detect the root issue.
# This problem does not happen when using Emacs tramp mode
RUN git config --global core.autocrlf false

ENV DISPLAY=:1

WORKDIR /opt
RUN git clone  https://github.com/glennrp/libpng
WORKDIR /opt/libpng/
RUN git checkout v1.5.13
RUN mkdir build
WORKDIR /opt/libpng/build
RUN cmake ..
RUN make -j 14 install

WORKDIR /opt
RUN mkdir /opt/venvs
RUN python3 -m venv /opt/venvs/oiio
# instead of activate
ENV PATH=/opt/venvs/oiio/bin:$PATH
RUN python -m pip --default-timeout=1000 install numpy==1.26.4

WORKDIR /opt
RUN git clone https://github.com/kaarrot/oiio.git
WORKDIR /opt/oiio/

# oiiotool requires a default font which is used in some tests, see: src/libOpenImageIO/imagebufalgo_draw.cpp
RUN mkdir -p /usr/share/fonts/
RUN cp src/fonts/Droid_Sans/*.ttf /usr/share/fonts/
RUN cp src/fonts/Droid_Serif/*.ttf /usr/share/fonts/
RUN cp src/fonts/Droid_Sans_Mono/*.ttf /usr/share/fonts/

RUN mkdir build 
WORKDIR /opt/oiio/build

RUN git checkout master

ENV CMAKE_BUILD_PARALLEL_LEVEL=14
RUN cmake -G Ninja .. -DOIIO_DOWNLOAD_MISSING_TESTDATA=ON -DOpenImageIO_BUILD_MISSING_DEPS=all -DINSTALL_FONTS=ON
RUN ninja

# RUN echo "Run Sekected Test:"
# RUN echo "PYTHONPATH=$(pwd)/lib/python/site-packages:$PYTHONPATH ctest -E broken --force-new-ctest-process --output-on-failure -R oiiotool-copy -V"

# RUN echo "Test fonts"
# RUN echo "bin/oiiotool  -pattern constant:color=.5,.1,.1 256x256 3 -text:size=50:x=75:y=140 1001 --attrib Make pet --attrib Model dog -d uint8 -otex file.1001.tx"

# #PYTHONPATH=$(pwd)/lib/python/site-packages:$PYTHONPATH ctest -E broken --force-new-ctest-process --output-on-failure -R oiiotool-copy -V
# # bin/oiiotool  -pattern constant:color=.5,.1,.1 256x256 3 -text:size=50:x=75:y=140 1001 --attrib Make pet --attrib Model dog -d uint8 -otex file.1001.tx
# #  could not set default font face
# # PYTHONPATH=$(pwd)/lib/python/site-packages:$PYTHONPATH ctest -E broken --force-new-ctest-process --output-on-failure -R oiiotool-text -V

CMD ["bash"]
