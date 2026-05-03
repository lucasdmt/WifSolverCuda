CXX=g++-10
CXXFLAGS=-std=c++14 -m64 -mssse3 -Wno-unused-result -Wno-write-strings -O2 -I. -I$(CUDA)/include

NVCC=nvcc
CUDA_HOME=/usr/local/cuda
CUDA_LIB=${CUDA_HOME}/lib64
CUDA_INCLUDE=${CUDA_HOME}/include
LFLAGS     = -lpthread -L$(CUDA_HOME)/lib64 -lcudart
CXXCUDA    = /usr/bin/g++-10

CUSRC:=$(wildcard *.cu)
MKDIR_P = mkdir -p
OBJDIR = obj

all:
	mkdir -p $(OBJDIR)
	$(CXX) $(CXXFLAGS) -o $(OBJDIR)/ripemd160.o -c lib/hash/ripemd160.cpp
	$(CXX) $(CXXFLAGS) -o $(OBJDIR)/ripemd160_sse.o -c lib/hash/ripemd160_sse.cpp
	$(CXX) $(CXXFLAGS) -o $(OBJDIR)/sha256cpp.o -c lib/hash/sha256.cpp
	$(CXX) $(CXXFLAGS) -o $(OBJDIR)/sha256_sse.o -c lib/hash/sha256_sse.cpp
	gcc -O3 -c lib/base58.c -o $(OBJDIR)/base58.o
	$(CXX) -O3 -c lib/util.cpp -o $(OBJDIR)/util.o
	$(CXX) -O3 -c lib/Bech32.cpp -o $(OBJDIR)/Bech32.o
	$(CXX) $(CXXFLAGS) -c lib/Int.cpp -o $(OBJDIR)/Int.o
	$(CXX) $(CXXFLAGS) -c lib/Point.cpp -o $(OBJDIR)/Point.o
	$(CXX) $(CXXFLAGS) -c lib/SECP256K1.cpp -o $(OBJDIR)/SECP256K1.o
	$(CXX) $(CXXFLAGS) -c lib/IntGroup.cpp -o $(OBJDIR)/IntGroup.o
	$(CXX) $(CXXFLAGS) -c lib/IntMod.cpp -o $(OBJDIR)/IntMod.o

	$(NVCC) --ptxas-options=-v -maxrregcount=0 --compile \
	--compiler-options "-fPIC -std=c++14" \
	-ccbin $(CXXCUDA) -m64 -O2 -std=c++14 -I$(CUDA)/include \
	-gencode=arch=compute_75,code=sm_75 -arch=sm_75 \
	-o $(OBJDIR)/sha256.o -c lib/hash/sha256.cu

	$(NVCC) --ptxas-options=-v -maxrregcount=0 --compile \
	--compiler-options "-fPIC -std=c++14" \
	-ccbin $(CXXCUDA) -m64 -O2 -std=c++14 -I$(CUDA)/include \
	-gencode=arch=compute_75,code=sm_75 -arch=sm_75 \
	-o $(OBJDIR)/Worker1.o -c Worker1.cu

	$(NVCC) --ptxas-options=-v -maxrregcount=0 --compile \
	--compiler-options "-fPIC -std=c++14" \
	-ccbin $(CXXCUDA) -m64 -O2 -std=c++14 -I$(CUDA)/include \
	-gencode=arch=compute_75,code=sm_75 -arch=sm_75 \
	-o $(OBJDIR)/main.o -c main.cu

	@echo Making WifSolver...
	$(CXX) $(OBJDIR)/ripemd160.o $(OBJDIR)/ripemd160_sse.o $(OBJDIR)/sha256cpp.o $(OBJDIR)/sha256_sse.o \
	$(OBJDIR)/base58.o $(OBJDIR)/util.o $(OBJDIR)/Bech32.o $(OBJDIR)/Int.o $(OBJDIR)/Point.o \
	$(OBJDIR)/SECP256K1.o $(OBJDIR)/IntGroup.o $(OBJDIR)/IntMod.o \
	$(OBJDIR)/sha256.o $(OBJDIR)/Worker1.o $(OBJDIR)/main.o $(LFLAGS) -o wifSolver

clean:
	@echo Cleaning...
	@rm -f $(OBJDIR)/*.o

