#
# Copyright 2021 Xilinx, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

ECHO=@echo

.PHONY: help

help::
	$(ECHO) "Makefile Usage:"
	$(ECHO) "  make gen_ip"
	$(ECHO) "      Command to generate the IPs used in this design"
	$(ECHO) ""
	$(ECHO) "  make pack_kernel"
	$(ECHO) "      Command to pack the module krnl_aes to Vitis kernel"
	$(ECHO) ""
	$(ECHO) "  make runsim"
	$(ECHO) "      Command to run the simulation"
	$(ECHO) ""
	$(ECHO) "  make build_hw"
	$(ECHO) "      Command to build xclbin files for Alveo platform, including krnel_aes and axis dump and issue HLS kernels"
	$(ECHO) ""
	$(ECHO) "  make build_sw"
	$(ECHO) "      Command to build host software for xclbin test"
	$(ECHO) ""
	$(ECHO) "  make clean"
	$(ECHO) "      Command to remove all the generated files."

# PART setting: uncomment the line matching your Alveo card
PART := xcu200-fsgd2104-2-e
#PART := cu250-figd2104-2L-e
#PART := xcu50-fsvh2104-2-e
#PART := xcu280-fsvh2892-2L-e

# PLATFORM setting: uncomment the lin matching your Alveo card
PLATFORM := xilinx_u200_xdma_201830_2
#PLATFORM := xilinx_u250_xdma_201830_2
#PLATFORM := xilinx_u50_gen3x16_xdma_201920_3
#PLATFORM := xilinx_u280_xdma_201920_3

# TARGET: set the build target, can be hw or hw_emu
TARGET := hw


.phony: clean traces

################## resource generation and simulation 
gen_ip:
	rm -rf ip_generation; mkdir ip_generation; vivado -mode batch -source ./gen_ip.tcl -tclargs $(PART)


pack_kernel:
	rm -rf vivado_pack_krnl_project; mkdir vivado_pack_krnl_project; cd vivado_pack_krnl_project; vivado -mode batch -source ../pack_kernel.tcl -tclargs $(PART)

runsim:
	./runsim_krnl_aes_xsim.sh


################## hardware build 
NCPUS := $(shell grep -c ^processor /proc/cpuinfo)
JOBS := $(shell expr $(NCPUS) - 1)

XOCCFLAGS := --platform $(PLATFORM) -t $(TARGET)  -s -g
XOCCLFLAGS := --link --optimize 3 --vivado.synth.jobs $(JOBS) --vivado.impl.jobs $(JOBS)
# You could uncomment following line and modify the options for hardware debug/profiling
#DEBUG_OPT := --debug.chipscope krnl_aes_1 --debug.protocol all --profile_kernel data:all:all:all:all

build_hw: strm_issue.xo strm_dump.xo
	v++ $(XOCCLFLAGS) $(XOCCFLAGS) $(DEBUG_OPT) --config krnl_aes_test.cfg -o krnl_aes_test_$(TARGET).xclbin krnl_aes.xo strm_dump.xo strm_issue.xo 

strm_dump.xo: ./hls/strm_dump.cpp
	v++ $(XOCCFLAGS) --kernel strm_dump -c -o $@ $<

strm_issue.xo: ./hls/strm_issue.cpp
	v++ $(XOCCFLAGS) --kernel strm_issue -c -o $@ $<

all: gen_ip pack_kernel build_hw

################## software build 
CXXFLAGS := -std=c++14 -Wno-deprecated-declarations
CXXFLAGS += -I$(XILINX_XRT)/include
LDFLAGS := -L$(XILINX_XRT)/lib
LDFLAGS += -lxrt_coreutil -lssl -lcrypto

EXECUTABLE:= host_krnl_aes_test

HOST_SRCS := ./host/host_krnl_aes_test.cpp

build_sw: $(EXECUTABLE)_$(TARGET)

$(EXECUTABLE)_$(TARGET): $(HOST_SRCS)
	$(CXX) -o $(EXECUTABLE) $^ $(CXXFLAGS) $(LDFLAGS)


################## clean up
clean:
	$(RM) -rf ip_generation vivado_pack_krnl_project
	$(RM) -rf *.xo *.xclbin *.xclbin.info *.xclbin.link_summary *.jou *.log *.xo.compile_summary _x
	$(RM) -rf *.dat *.pb xsim.dir *.xml *.ltx *.csv *.json *.protoinst *.wdb *.wcfg host_krnl_aes_test
