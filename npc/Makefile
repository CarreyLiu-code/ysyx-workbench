# ==============================================================================
# 基础工具链配置
# ==============================================================================
VERILATOR = verilator
# 共享的基础编译参数：开启警告，生成 C++ 代码，使用 O3 优化，加速仿真
VERILATOR_BASE_FLAGS = -Wall --cc --build -O3 --x-assign fast --x-initial fast --noassert

# 目录结构与共享源文件
TOP_MODULE = top
BUILD_DIR = build
VSRC_DIR = vsrc
CSRC_DIR = csrc

# 自动收集所有的 Verilog 源码
VERILOG_SOURCES = $(shell find $(abspath ./$(VSRC_DIR)) -name "*.v")

# ==============================================================================
# 场景一：纯 RTL 仿真 (面向波形分析与自动化测试)
# ==============================================================================
SIM_BIN = $(BUILD_DIR)/$(TOP_MODULE)_sim
SIM_OBJ_DIR = $(BUILD_DIR)/obj_dir_sim
# 【关键】这里只指定用于波形仿真的 C++ 文件
SIM_CPP = $(abspath $(CSRC_DIR)/main_tb.cpp) 

VCD_FILE = waveform.vcd
FST_FILE = waveform.fst
# 开启波形追踪参数
VCD_FLAGS = --trace --trace-fst --trace-structs

$(SIM_BIN): $(VERILOG_SOURCES) $(SIM_CPP)
	@mkdir -p $(BUILD_DIR)
	@rm -rf $(SIM_OBJ_DIR)
	$(VERILATOR) $(VERILATOR_BASE_FLAGS) $(VCD_FLAGS) \
		--top-module $(TOP_MODULE) $(VERILOG_SOURCES) $(SIM_CPP) \
		--Mdir $(SIM_OBJ_DIR) --exe -o $(abspath $(SIM_BIN))

# ==============================================================================
# 场景二：NVBoard 交互式仿真 (面向图形化虚拟开发板)
# ==============================================================================
NVB_BIN = $(BUILD_DIR)/$(TOP_MODULE)_nvboard
NVB_OBJ_DIR = $(BUILD_DIR)/obj_dir_nvboard
# 【关键】这里只指定用于 NVBoard 交互的 C++ 文件
NVB_CPP = $(abspath $(CSRC_DIR)/main_nvboard.cpp) 

NXDC_FILES = constr/$(TOP_MODULE).nxdc
SRC_AUTO_BIND = $(abspath $(BUILD_DIR)/auto_bind.cpp)

# NVBoard 引脚绑定代码生成规则
$(SRC_AUTO_BIND): $(NXDC_FILES)
	@mkdir -p $(BUILD_DIR)
	python3 $(NVBOARD_HOME)/scripts/auto_pin_bind.py $^ $@

# 引入 NVBoard 的底层构建规则
include $(NVBOARD_HOME)/scripts/nvboard.mk

# NVBoard 专用的 C++ 编译与链接选项
INC_PATH ?=
INCFLAGS = $(addprefix -I, $(INC_PATH))
CXXFLAGS += $(INCFLAGS) -DTOP_NAME="\"V$(TOP_MODULE)\""

# 注意：NVBoard 仿真通常不需要加 --trace 开启波形，以节省性能开销
$(NVB_BIN): $(VERILOG_SOURCES) $(NVB_CPP) $(SRC_AUTO_BIND) $(NVBOARD_ARCHIVE)
	@mkdir -p $(BUILD_DIR)
	@rm -rf $(NVB_OBJ_DIR)
	$(VERILATOR) $(VERILATOR_BASE_FLAGS) \
		--top-module $(TOP_MODULE) $(VERILOG_SOURCES) $(NVB_CPP) $(SRC_AUTO_BIND) $(NVBOARD_ARCHIVE) \
		$(addprefix -CFLAGS , $(CXXFLAGS)) $(addprefix -LDFLAGS , $(LDFLAGS)) \
		--Mdir $(NVB_OBJ_DIR) --exe -o $(abspath $(NVB_BIN))

# ==============================================================================
# 伪目标与快捷执行命令
# ==============================================================================
default:
	@echo "Usage:"
	@echo "  make sim      - 编译并运行纯 RTL 仿真 (适合写激励、查波形)"
	@echo "  make nvboard  - 编译并运行 NVBoard 交互式仿真 (适合拨开关、看效果)"
	@echo "  make wave     - 查看生成的 VCD 波形"
	@echo "  make clean    - 清理所有生成的文件"

# 运行纯 RTL 仿真
sim: $(SIM_BIN)
	$(SIM_BIN)
	$(call git_commit, "sim pure RTL") # DO NOT REMOVE THIS LINE!!!

# 运行 NVBoard 仿真
nvboard: $(NVB_BIN)
	$(NVB_BIN)
	$(call git_commit, "sim RTL with NVBoard") # DO NOT REMOVE THIS LINE!!!

wave: $(VCD_FILE)
	gtkwave $(VCD_FILE) &

wave-fst: $(FST_FILE)
	gtkwave $(FST_FILE) &

# 依赖关系：看波形前必须先跑过纯 RTL 仿真
$(VCD_FILE) $(FST_FILE): sim

clean:
	rm -rf $(BUILD_DIR) *.vcd *.fst *.log

.PHONY: default sim nvboard wave wave-fst clean

include ../Makefile