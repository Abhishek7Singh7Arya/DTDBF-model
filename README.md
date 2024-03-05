
# follow these steps to run this code
<!--  this code is for DTDBFs version only as we mentioned in the paper we updated Proteus code -->

 


## Directory Layout
`include/` - contains all the files necessary to the implementation of the Dtdbf filter and the CPFPR model.

`bench/` - contains all the files used for in-memory standalone filter benchmarks and a script to run RocksDB benchmarks.

`workloads/` - contains all the files used for generating synthetic workloads and downloading real-world datasets.

`rocksdb-6.20.3/` - contains a modified version of RockDB v6.20.3 with dtdbf and SuRF integrated.

`SuRF/` - modified version of SuRF. 

## Requirements

- `jq` (to download Internet domains dataset)
- (RocksDB) `CMake`, `gtest`, `lz4`, `gflags`, `zstandard`

```	
# For Ubuntu
sudo apt-get install jq build-essential cmake libgtest.dev liblz4-dev libzstd-dev libgflags-dev
```

## Setup

	cd Proteus/workloads
	./setup.sh

	
`setup.sh` compiles string and integer workload generators and retrieves real-world datasets - `books_800M_uint64` and `fb_200M_uint64` from https://github.com/learnedsystems/SOSD, `.org` domains from https://domainsproject.org/


# Standalone In-Memory Filter Benchmarks

Configure the workload setup as detailed in `bench.sh`.

	cd Proteus/bench
	make bench
	./bench.sh &> bench.txt

To see more modeling information, uncomment the relevant macro definitions in `include/modeling.cpp`.

# RocksDB Filter Integration and Benchmarks

Changes to the RocksDB source code can be grepping the tag `ProteusMod`. Since RockDB ["does not report false positive rate for prefix in seeks"](https://github.com/facebook/rocksdb/issues/3680#issuecomment-384786975), we implement custom range query FPR statistics recording. Note that this is only valid for experiments that use forward iterators only, not for general purpose. We use RocksDB v6.20.3 for our benchmarks.

## Build RocksDB

	cd rocksdb
	mkdir build && cd build
	cmake -DCMAKE_BUILD_TYPE=Release -DWITH_LZ4=ON -DWITH_ZSTD=ON ..
	make -j$(nproc) filter_experiment

## Run dtdbf experiment with RocksDB 

	cd Proteus/bench
	./rocksdb.sh rocksdb_experiment
