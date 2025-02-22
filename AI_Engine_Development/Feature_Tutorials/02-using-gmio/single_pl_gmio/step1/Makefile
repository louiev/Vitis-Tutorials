#
#Copyright 2020 Xilinx, Inc.
#Licensed under the Apache License, Version 2.0 (the "License");
#you may not use this file except in compliance with the License.
#You may obtain a copy of the License at
#    http://www.apache.org/licenses/LICENSE-2.0
#Unless required by applicable law or agreed to in writing, software
#distributed under the License is distributed on an "AS IS" BASIS,
#WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#See the License for the specific language governing permissions and
#limitations under the License.
#
TARGET   = hw
MODE	 = linux
PLATFORM = ${PLATFORM_REPO_PATHS}/xilinx_vck190_es1_base_202020_1/xilinx_vck190_es1_base_202020_1.xpfm
XCLBIN   = vck190_aie_base_graph_${TARGET}.xclbin
XSA	 = vck190_aie_base_graph_${TARGET}.xsa
XPFM	 = vck190_xpfm
HOST_EXE = host.exe

GRAPH    = aie/graph.cpp
LIBADF  = libadf.a
AIE_CMPL_CMD = aiecompiler -platform=${PLATFORM} -include="./aie" -include="./aie/hls/" --pl-axi-lite=true --pl-freq=250 -workdir=./Work ${GRAPH}
AIE_SIM_CMD = aiesimulator --pkg-dir=./Work

##########################################################################################################################################################
### DO NOT MODIFY BELOW THIS LINE UNLESS NECESSARY
################################################################################################################################################

GRAPH_XOS= $(subst .cpp,.xo,$(wildcard aie/hls/*.cpp))
VCC      = v++
VPP_SPEC =system.cfg
VPP_FLAGS=--save-temps --verbose --config ${VPP_SPEC}  

.PHONY: clean

###
# Guarding Checks. Do not modify.
###
check_defined = \
	$(strip $(foreach 1,$1, \
		$(call __check_defined,$1,$(strip $(value 2)))))

__check_defined = \
	$(if $(value $1),, \
		$(error Undefined $1$(if $2, ($2))))

guard-PLATFORM_REPO_PATHS:
	$(call check_defined, PLATFORM_REPO_PATHS, Set your where you downloaded xilinx_vck190_es1_base_2020210_1)

guard-ROOTFS:
	$(call check_defined, ROOTFS, Set to: xilinx-versal-common-v2020.2/rootfs.ext4)

guard-IMAGE:
	$(call check_defined, IMAGE, Set to: xilinx-versal-common-v2020.2/Image)

guard-CXX:
	$(call check_defined, CXX, Run: xilinx-versal-common-v2020.2/environment-setup-aarch64-xilinx-linux)

guard-SDKTARGETSYSROOT:
	$(call check_defined, SDKTARGETSYSROOT, Run: xilinx-versal-common-v2020.2/environment-setup-aarch64-xilinx-linux)

###

all: ${XCLBIN} ${HOST_EXE} package

aie: guard-PLATFORM_REPO_PATHS ${LIBADF}
${LIBADF}: ${GRAPH}
	${AIE_CMPL_CMD}

${GRAPH_XOS}: guard-PLATFORM_REPO_PATHS
	make -C aie/hls/

aiesim: ${LIBADF}
	${AIE_SIM_CMD}

xclbin: guard-PLATFORM_REPO_PATHS ${XCLBIN}

${XCLBIN}: ${LIBADF} ${VPP_SPEC} ${GRAPH_XOS}
	${VCC} -g -l --platform ${PLATFORM} ${GRAPH_XOS} ${LIBADF} -t ${TARGET} ${VPP_FLAGS} -o $@

host: guard-CXX guard-SDKTARGETSYSROOT ${HOST_EXE}
${HOST_EXE}: ${GRAPH} ./Work/ps/c_rts/aie_control_xrt.cpp
	make -C sw/
package: guard-ROOTFS guard-IMAGE guard-PLATFORM_REPO_PATHS package_${TARGET}
package_${TARGET}: ${LIBADF} ${XCLBIN} ${HOST_EXE} 
	${VCC} -p -t ${TARGET} -f ${PLATFORM} \
		--package.rootfs ${ROOTFS} \
		--package.kernel_image ${IMAGE} \
		--package.boot_mode=sd \
		--package.image_format=ext4 \
		--package.defer_aie_run \
		--package.sd_file ${HOST_EXE} ${XCLBIN} ${LIBADF}

clean:
	rm -rf _x v++_* ${XOS} ${OS} ${LIBADF} *.o.* *.o *.xpe *.xo.* \
	       vck190_aie_base*.xclbin* *.xsa *.log *.jou xnwOut Work Map_Report.csv \
	       ilpProblem* sol.db drivers .Xil
	make -C aie/hls clean
	make -C sw clean
