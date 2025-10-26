# **Week-5 RISC-V Tapeout Program: OpenROAD Flow Setup and Physical Design**

<div align="center">

<img src="./screenshots/setup/openroad_logo.jpeg" alt="OpenROAD Logo" width="250" height="50"/>


**Previous Week Repository** --> [click_here](https://github.com/Meenakshi-2627/Week-4_RISC-V_Tapeout_Program)

---

# Table of Contents

 **1. Introduction**  
    [Overview](#overview)  
    [Learning Objectives](#learning-objectives)  
    [Prerequisites](#prerequisites)  

 **2. Physical Design Flow**  
    [What is Physical Design?](#what-is-physical-design)  
    [OpenROAD Flow Stages](#openroad-flow-stages)  

 **3. OpenROAD Installation**   
    [System Requirements](#system-requirements)  
    [Git Cloning & Dependencies](#git-cloning--dependencies)  
    [Building OpenROAD](#building-openroad)  

 **4. Floorplanning**   
    [Overview & Key Metrics](#overview--key-metrics)  
    [Running Floorplan Stage](#running-floorplan-stage)  
    [Understanding Outputs](#understanding-outputs)  

 **5. Placement**   
    [Overview & Goals](#overview--goals)  
    [Running Placement Stage](#running-placement-stage)  
    [Understanding Outputs](#understanding-outputs-1)  

 **6. Visualization**  
    [GUI for Floorplan & Placement](#gui-for-floorplan--placement)  

 **7. Error Resolving**    
      [Troubleshooting & Common Errors](#troubleshooting--common-errors)  

 **8. Summary of commands used**  
     [Quick Reference Commands](#quick-reference-commands)  

 **9. Conclusion of Week-5**  
     [Conclusion](#conclusion)
</div>

---

## 1. Introduction

### Overview

Welcome to Week 5 of the RISC-V Tapeout Program! This week marks an exciting transition from transistor-level circuit design to **physical design implementation**—the process of converting logical circuits into actual silicon layout.

**What you'll accomplish this week:**
- ✅ Install OpenROAD Flow Scripts (complete RTL-to-GDSII toolchain)
- ✅ Execute **Floorplanning** to define chip dimensions and placement regions
- ✅ Execute **Placement** to position standard cells optimally
- ✅ Visualize your design at each stage


---

### Prerequisites

#### Knowledge Prerequisites
- ☑️ Basic Linux command-line skills
- ☑️ Understanding of digital logic design (gates, flip-flops)
- ☑️ Familiarity with Verilog/VHDL (helpful but not required)
- ☑️ Completion of Week 4 (timing analysis with SPICE)

#### System Prerequisites
- **Operating System**: Ubuntu 20.04 / 22.04 LTS or similar Linux distribution
- **RAM**: Minimum 8GB (16GB recommended)
- **Disk Space**: At least 20GB free
- **Processor**: Multi-core CPU (4+ cores recommended)
- **Internet**: For downloading dependencies

#### Software Prerequisites Check
```bash
# Check your Ubuntu version
lsb_release -a

# Check available disk space
df -h

# Check RAM
free -h

# Check CPU cores
nproc
```

---

## 2. Understanding the Physical Design Flow

### What is Physical Design?

**Physical Design** is the process of transforming a logical circuit description (RTL - Register Transfer Level) into a physical layout that can be manufactured on silicon.


`RTL Code` → `Synthesis` → `Floorplan` → `Placement` → `CTS` → `Routing` → `Signoff` → `GDSII`


**Explanation**
- **RTL** = Architectural blueprint
- **Synthesis** = Choosing building materials (logic gates)
- **Floorplan** = Defining plot boundaries and room locations
- **Placement** = Positioning furniture (standard cells)
- **Routing** = Installing wiring and plumbing (metal interconnects)

---

### OpenROAD Flow Stages

The OpenROAD flow consists of 6 major stages. **This week focuses on stages 2 and 3.**

| Stage | Name | Purpose | Week 5 Status |
|-------|------|---------|---------------|
| **1** | **Synthesis** | Converts RTL Verilog to gate-level netlist | ✅ Will Run |
| **2** | **Floorplan** | Define die area, IO pins, power grid | 🎯 **Focus** |
| **3** | **Placement** | Position standard cells optimally | 🎯 **Focus** |
| **4** | CTS | Build clock distribution network | ❌ Skip This Week |
| **5** | Routing | Connect cells with metal wires | ❌ Skip This Week |
| **6** | Signoff | DRC/LVS checks, GDS generation | ❌ Skip This Week |

#### Stage 1: Synthesis (Yosys)
- **Input:** RTL Verilog code (`.v` files)
- **Process:** Logic optimization, technology mapping
- **Output:** Gate-level netlist (`1_synth.v`)

#### Stage 2: Floorplan (OpenROAD) 🎯
- **Input:** Synthesized netlist, design constraints
- **Process:** Define chip dimensions, place IO pads, insert tap cells, create power grid
- **Output Files:**
  - `2_1_floorplan.odb` - Initial floorplan
  - `2_4_floorplan_pdn.odb` - With power grid
  - `2_floorplan.odb` - **Final floorplan database**

#### Stage 3: Placement (OpenROAD) 🎯
- **Input:** Floorplan with placement rows
- **Process:** Global placement, IO placement, cell resizing, detail placement
- **Output Files:**
  - `3_1_place_gp_skip_io.odb` - Global placement
  - `3_5_place_dp.odb` - Detail placement
  - `3_place.odb` - **Final placement database**

---

## 3. OpenROAD Installation

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| OS | Ubuntu 20.04 | Ubuntu 22.04 LTS |
| RAM | 8 GB | 16 GB |
| Disk | 15 GB | 30 GB |
| CPU | 2 cores | 4+ cores |

---

### Git Cloning

#### Step 1: Create Working Directory
```bash
# Create project directory
cd ~
mkdir -p OpenROAD
cd OpenROAD
```

#### Step 2: Clone Repository
```bash
# Clone with all submodules (IMPORTANT!)
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts.git

# Enter directory
cd OpenROAD-flow-scripts

# Verify clone success
ls -la
```

> **⚠️ Critical:** The `--recursive` flag ensures all submodules (Yosys, OpenROAD, etc.) are cloned. Without it, tools won't be available!

---

### Dependency Installation

#### Step 3: Install Base Dependencies
```bash
# Run automated dependency installer
sudo ./etc/DependencyInstaller.sh
```

**This installs:**
- Build tools (gcc, g++, cmake, make)
- Libraries (Boost, TCL, Qt5, CUDD, spdlog)
- Python packages
- Other VLSI tools

**Expected time:** 10-30 minutes

---

#### Common Dependency Issues and Fixes

**Issue 1: Missing Boost 1.78+**
```bash
# Install from source
cd ~/Downloads
wget https://sourceforge.net/projects/boost/files/boost/1.78.0/boost_1_78_0.tar.gz/download -O boost_1_78_0.tar.gz
tar -xzf boost_1_78_0.tar.gz
cd boost_1_78_0
./bootstrap.sh --prefix=/usr/local
sudo ./b2 install
```

**Issue 2: Missing CUDD Library**
```bash
# Build from source
cd ~/Downloads
git clone https://github.com/ivmai/cudd.git
cd cudd
autoreconf -i
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
sudo ldconfig
```

**Issue 3: Missing spdlog**
```bash
sudo apt-get update
sudo apt-get install -y libspdlog-dev
```

**Issue 4: Missing GoogleTest**
```bash
sudo apt-get install -y libgtest-dev
cd /usr/src/gtest
sudo cmake .
sudo make
sudo cp lib/*.a /usr/lib
```

---

### Building OpenROAD

#### Step 4: Build All Tools
```bash
# Navigate to repository root
cd ~/OpenROAD/OpenROAD-flow-scripts

# Build OpenROAD and all tools
./build_openroad.sh --local
```

**This compiles:**
- Yosys (synthesis)
- OpenROAD (physical design)
- OpenSTA (timing analysis)
- KLayout (layout viewer)

**Expected time:** 30-60 minutes

---

#### Step 5: Set Environment Variables
```bash
# Source environment
source ./env.sh

# Verify tools
which yosys
which openroad

# Check versions
yosys -V
openroad -version
```

**Expected output:**
```
$ yosys -V
Yosys 0.38+42 (git sha1 ...)

$ openroad -version
OpenROAD v2.0-...
```

---

### Verification

#### Step 6: Test Installation
```bash
# Navigate to flow directory
cd ~/OpenROAD/OpenROAD-flow-scripts/flow

# List available designs
ls designs/

# Test help
make help
```

✅ **If you see the Makefile help menu, installation is successful!**

---

## 4. Running the Flow: Floorplanning

### What is Floorplanning?

Floorplanning defines:
1. **Die Size** - Total chip area (including IO pads)
2. **Core Area** - Usable area for logic cells
3. **IO Placement** - Where input/output pins go
4. **Power Grid** - Power (VDD) and ground (VSS) distribution
5. **Placement Rows** - Legal positions for standard cells

**Key Metrics:**
```
Utilization = (Total Cell Area) / (Core Area) × 100%
Aspect Ratio = Core Height / Core Width
```

---

### Executing Floorplan Stage

#### Step 7: Navigate to Flow Directory
```bash
cd ~/OpenROAD/OpenROAD-flow-scripts/flow
```

#### Step 8: Choose GCD Design
```bash
# View available designs
ls designs/nangate45/

# We'll use GCD (Greatest Common Divisor) - simple design for learning
```

#### Step 9: Set Design Configuration
```bash
# Set design config
export DESIGN_CONFIG=./designs/nangate45/gcd/config.mk

# View config
cat $DESIGN_CONFIG
```

#### Step 10: Run Floorplan
```bash
# Clean previous builds
make clean_all

# Run synthesis + floorplan
make DESIGN_CONFIG=./designs/nangate45/gcd/config.mk floorplan
```

**Expected output:**
```
[INFO] Running synthesis...
[INFO] Yosys synthesis complete
[INFO] Starting floorplan...
[INFO] Core area: 100 x 100 um
[INFO] Utilization: 40%
[INFO] Floorplan complete ✓
```

**Expected time:** 2-5 minutes

---

### Understanding Floorplan Outputs

#### Step 11: Locate Output Files
```bash
cd results/nangate45/gcd/base/

# List floorplan files
ls -lh 2_*.odb
```

**Files created:**
- `2_1_floorplan.odb` - Initial floorplan
- `2_2_floorplan_macro.odb` - With macro placement
- `2_3_floorplan_tapcell.odb` - With tap cells
- `2_4_floorplan_pdn.odb` - With power grid
- `2_floorplan.odb` - **Final floorplan**

#### Step 12: View Reports
```bash
# View floorplan report
cat ../../reports/nangate45/gcd/base/2_floorplan.rpt
```

**Sample report:**
```
Design: gcd
Die Area:     110.4 × 111.2 um² (12,276 um²)
Core Area:    100.4 × 101.2 um² (10,160 um²)
Utilization:  40.0%
Aspect Ratio: 1.01
Rows:         88
```

---

## 5. Running the Flow: Placement

### What is Placement?

Placement positions each standard cell (AND, OR, flip-flop, etc.) into legal locations on the floorplan.

**Goals:**
- ✅ Minimize wire length (reduces delay/power)
- ✅ Meet timing constraints
- ✅ Avoid congestion
- ✅ Balance cell distribution

**Placement steps:**
1. **Global Placement** - Coarse positioning (cells can overlap)
2. **IO Placement** - Position input/output pins
3. **Resizing** - Adjust cell sizes for timing
4. **Detail Placement** - Legalization (no overlaps)

---

### Executing Placement Stage

#### Step 13: Run Placement
```bash
cd ~/OpenROAD/OpenROAD-flow-scripts/flow

# Run placement
make DESIGN_CONFIG=./designs/nangate45/gcd/config.mk place
```

**Expected output:**
```
[INFO] Starting placement...
[INFO] Global Placement: HPWL = 15,234 um
[INFO] IO Placement: 16 pins placed
[INFO] Resizing: 12 cells upsized
[INFO] Detail Placement: HPWL = 14,756 um
[INFO] Placement complete ✓
```

**Expected time:** 3-10 minutes

---

### Understanding Placement Outputs

#### Step 14: Locate Files
```bash
cd results/nangate45/gcd/base/

# List placement files
ls -lh 3_*.odb
```

**Files created:**
- `3_1_place_gp_skip_io.odb` - Global placement (no IO)
- `3_2_place_iop.odb` - IO pins placed
- `3_3_place_gp.odb` - Global placement (with IO)
- `3_4_place_resized.odb` - After cell resizing
- `3_5_place_dp.odb` - Detail placement
- `3_place.odb` - **Final placement**

#### Step 15: View Reports
```bash
# Placement statistics
cat ../../reports/nangate45/gcd/base/3_place.rpt

# Timing report
cat ../../reports/nangate45/gcd/base/3_place_timing.rpt
```

**Sample report:**
```
Total Cells:           145
Cell Area:             4,285 um²
HPWL:                  14,756 um
Worst Slack:           +0.23 ns (MET)
Total Negative Slack:  0.00 ns
```

---

## 6. Visualizing Results

### Opening the GUI

#### Step 16: Launch GUI for Floorplan
```bash
cd ~/OpenROAD/OpenROAD-flow-scripts/flow

# Open floorplan in GUI
make DESIGN_CONFIG=./designs/nangate45/gcd/config.mk gui_floorplan
```

**Alternative (manual):**
```bash
openroad -gui
# In console:
read_lef ../platforms/nangate45/lef/NangateOpenCellLibrary.tech.lef
read_db results/nangate45/gcd/base/2_floorplan.odb
```

---

### Viewing Floorplan

**What to look for:**
- ✅ Die Boundary (outer rectangle)
- ✅ Core Area (inner rectangle)
- ✅ Placement Rows (horizontal lines)
- ✅ IO Pads/Pins (around perimeter)
- ✅ Power Stripes (VDD/VSS grid)

**GUI Controls:**
- `F` - Fit view
- `Z` / Scroll - Zoom in/out
- Middle-click drag - Pan
- Ruler icon - Measure distance

---

### Viewing Placement

#### Step 18: Open Placement in GUI
```bash
make DESIGN_CONFIG=./designs/nangate45/gcd/config.mk gui_place
```

**What to observe:**
- ✅ Standard cells filling rows
- ✅ No overlaps (legal placement)
- ✅ Different cell types (colors)
- ✅ IO pins connected
- ✅ Related cells placed nearby

**Zoom in to see individual gates (AND, OR, DFF, etc.)**

---

## 7. Troubleshooting Common Errors

### Yosys Errors

#### Error 1: `No such frontend: verilog`

**Problem:** Yosys missing Verilog frontend

**Fix:**
```bash
cd ~/OpenROAD/OpenROAD-flow-scripts/yosys
git submodule update --init --recursive
make clean
make config-clang
make -j$(nproc)
sudo make install PREFIX=~/OpenROAD/OpenROAD-flow-scripts/tools/install/yosys
```

**Verify:**
```bash
yosys -p "help" | grep verilog
```

---

#### Error 2: `init_share_dirname: unable to determine share/ directory`

**Problem:** Missing Yosys share directory

**Fix:**
```bash
cd ~/OpenROAD/OpenROAD-flow-scripts/yosys
sudo make install PREFIX=~/OpenROAD/OpenROAD-flow-scripts/tools/install/yosys

# Verify
ls ~/OpenROAD/OpenROAD-flow-scripts/tools/install/yosys/share/yosys/
```

---

#### Error 3: `read_liberty: unknown option`

**Problem:** ABC (synthesis backend) version mismatch

**Fix:**
```bash
cd ~/OpenROAD/OpenROAD-flow-scripts/yosys
git submodule update --init --recursive
make clean
make -j$(nproc)
sudo make install PREFIX=~/OpenROAD/OpenROAD-flow-scripts/tools/install/yosys
```

---

### Permission Issues

#### Error 4: `Permission denied`

**Fix:**
```bash
# Change ownership to your user
sudo chown -R $USER:$USER ~/OpenROAD/OpenROAD-flow-scripts/tools/install/

# Add write permission
sudo chmod -R u+w ~/OpenROAD/OpenROAD-flow-scripts/tools/install/
```

---

#### Error 5: `Text file busy`

**Problem:** Yosys binary is running

**Fix:**
```bash
# Kill running processes
ps aux | grep yosys
pkill -9 yosys

# Wait and retry
sleep 2
```

---

### Flow Errors

#### Error 6: `Core area too small`

**Problem:** Cell area exceeds core area (utilization > 100%)

**Fix:**
```bash
# Edit config
nano designs/nangate45/gcd/config.mk

# Change:
export CORE_UTILIZATION = 40
# To:
export CORE_UTILIZATION = 30

# Clean and retry
make clean_all
make floorplan
```

---

#### Error 7: `Cannot legalize placement`

**Problem:** Too many cells, not enough space

**Fix:**
```bash
# Reduce utilization
nano designs/nangate45/gcd/config.mk
export CORE_UTILIZATION = 35

# Or increase core margin
export CORE_MARGIN = 5

# Retry
make clean_all
make place
```

---

### Debugging Tips

**Always check logs first:**
```bash
# View errors in synthesis
cat logs/nangate45/gcd/base/1_1_yosys.log

# View floorplan errors
cat logs/nangate45/gcd/base/2_1_floorplan.log

# View placement errors
cat logs/nangate45/gcd/base/3_1_place_gp.log

# Search for errors
grep -i "error" logs/nangate45/gcd/base/*.log
```

**Clean before retrying:**
```bash
make clean_synth      # Clean synthesis only
make clean_floorplan  # Clean floorplan only
make clean_place      # Clean placement only
make clean_all        # Clean everything
```

---

## 8. Understanding Output Files

### File Structure

```
OpenROAD-flow-scripts/
├── flow/
│   ├── designs/nangate45/gcd/
│   │   ├── config.mk          ← Design configuration
│   │   └── constraint.sdc     ← Timing constraints
│   ├── results/nangate45/gcd/base/
│   │   ├── 1_synth.v          ← Gate-level netlist
│   │   ├── 2_floorplan.odb    ← Floorplan database
│   │   ├── 3_place.odb        ← Placement database
│   │   ├── 2_floorplan.def    ← Floorplan DEF
│   │   └── 3_place.def        ← Placement DEF
│   ├── logs/nangate45/gcd/base/
│   │   ├── 1_1_yosys.log
│   │   ├── 2_1_floorplan.log
│   │   └── 3_1_place_gp.log
│   └── reports/nangate45/gcd/base/
│       ├── 2_floorplan.rpt
│       ├── 3_place.rpt
│       └── 3_place_timing.rpt
```

---

### Key Output Files

#### Synthesis Outputs
| File | Format | Description |
|------|--------|-------------|
| `1_synth.v` | Verilog | Gate-level netlist |
| `1_synth.sdc` | SDC | Timing constraints |

#### Floorplan Outputs
| File | Format | Description |
|------|--------|-------------|
| `2_1_floorplan.odb` | ODB | Initial floorplan |
| `2_4_floorplan_pdn.odb` | ODB | With power grid |
| `2_floorplan.odb` | ODB | **Final floorplan** |
| `2_floorplan.def` | DEF | Floorplan (text format) |

#### Placement Outputs
| File | Format | Description |
|------|--------|-------------|
| `3_1_place_gp_skip_io.odb` | ODB | Global placement |
| `3_5_place_dp.odb` | ODB | Detail placement |
| `3_place.odb` | ODB | **Final placement** |
| `3_place.def` | DEF | Placement (text format) |

---

### File Format Explanations

**ODB (OpenROAD Database)**
- Binary format storing complete design state
- Fast to read/write
- Opens directly in OpenROAD GUI

**DEF (Design Exchange Format)**
- ASCII text format
- Industry standard
- Human-readable
- Compatible with many EDA tools

**SDC (Synopsys Design Constraints)**
- Timing constraints file
- Clock definitions, input/output delays

---

## 9. Deliverables Checklist

### Required Submissions

Create a GitHub repository with this structure:
```
Week5-OpenROAD-Floorplan-Placement/
├── README.md
├── images/
│   ├── installation/
│   │   ├── 01-system-info.png
│   │   ├── 02-git-clone.png
│   │   ├── 03-build-complete.png
│   │   └── 04-tool-versions.png
│   ├── floorplan/
│   │   ├── 01-floorplan-gui-full.png
│   │   ├── 02-floorplan-zoom.png
│   │   └── 03-floorplan-report.png
│   └── placement/
│       ├── 01-placement-gui-full.png
│       ├── 02-placement-zoom.png
│       └── 03-placement-report.png
└── reports/
    ├── 2_floorplan.rpt
    ├── 3_place.rpt
    └── 3_place_timing.rpt
```

---

### Required Screenshots

#### Installation Evidence (4 minimum)
1. ✅ System info with username (`uname -a`, `whoami`)
2. ✅ Git clone with username visible
3. ✅ Build success message
4. ✅ Tool versions (`yosys -V`, `openroad -version`)

#### Execution Evidence (4 minimum)
5. ✅ Floorplan command execution
6. ✅ Floorplan completion logs
7. ✅ Placement command execution
8. ✅ Placement completion logs

#### Visual Evidence (4 minimum)
9. ✅ Floorplan GUI - full view
10. ✅ Floorplan GUI - zoomed view
11. ✅ Placement GUI - full view
12. ✅ Placement GUI - zoomed view (cells visible)

#### Reports (3 files)
13. ✅ `2_floorplan.rpt`
14. ✅ `3_place.rpt`
15. ✅ `3_place_timing.rpt`

---

## Installation Evidence
![System Info](images/installation/01-system-info.png)
![Build Complete](images/installation/03-build-complete.png)

## Floorplan Results
![Floorplan Full](images/floorplan/01-floorplan-gui-full.png)
![Floorplan Zoom](images/floorplan/02-floorplan-zoom.png)

## Placement Results
![Placement Full](images/placement/01-placement-gui-full.png)
![Placement Zoom](images/placement/02-placement-zoom.png)

## Reports
- [Floorplan Report](reports/2_floorplan.rpt)
- [Placement Report](reports/3_place.rpt)
- [Timing Report](reports/3_place_timing.rpt)

---

### Quick Reference Commands

**Check status:**
```bash
# Where am I in the flow?
ls results/nangate45/gcd/base/*.odb | tail -n 1

# Check for errors
grep -i "error" logs/nangate45/gcd/base/*.log
```

**View reports:**
```bash
# Floorplan metrics
cat reports/nangate45/gcd/base/2_floorplan.rpt

# Placement metrics
cat reports/nangate45/gcd/base/3_place.rpt
```

**Clean specific stages:**
```bash
make clean_synth
make clean_floorplan
make clean_place
make clean_all
```

---

### Glossary

| Term | Definition |
|------|------------|
| **ASIC** | Application-Specific Integrated Circuit |
| **GDSII** | Final layout format for manufacturing |
| **LEF** | Library Exchange Format - physical abstract |
| **DEF** | Design Exchange Format - physical layout |
| **ODB** | OpenROAD Database - internal format |
| **SDC** | Synopsys Design Constraints |
| **RTL** | Register Transfer Level |
| **Utilization** | % of core area filled with cells |
| **Slack** | Timing margin (positive = met) |

---

## Conclusion

🎉 **Congratulations!** You have successfully:

✅ Installed OpenROAD Flow Scripts  
✅ Executed floorplanning (defined chip boundaries)  
✅ Executed placement (positioned standard cells)  
✅ Visualized physical design in OpenROAD GUI  
✅ Analyzed design metrics  

---

## Appendix: Complete Installation Script

```bash
#!/bin/bash
# Complete OpenROAD installation for Ubuntu 22.04

# Update system
sudo apt-get update

# Install base tools
sudo apt-get install -y \
    build-essential cmake git g++ wget curl \
    automake autoconf libtool m4 perl \
    tcl-dev libreadline-dev python3 python3-dev \
    zlib1g-dev swig libboost-all-dev

# Install libraries
sudo apt-get install -y libgtest-dev libspdlog-dev liblemon-dev

# Build GTest
cd /usr/src/gtest
sudo cmake . && sudo make && sudo cp lib/*.a /usr/lib

# Install Boost 1.78 (if needed)
cd ~/Downloads
wget https://sourceforge.net/projects/boost/files/boost/1.78.0/boost_1_78_0.tar.gz/download -O boost_1_78_0.tar.gz
tar -xzf boost_1_78_0.tar.gz
cd boost_1_78_0
./bootstrap.sh --prefix=/usr/local
sudo ./b2 install

# Install CUDD
cd ~/Downloads
git clone https://github.com/ivmai/cudd.git
cd cudd
autoreconf -i
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
sudo ldconfig

# Clone OpenROAD
cd ~/OpenROAD
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts.git
cd OpenROAD-flow-scripts

# Build Yosys
cd yosys
git submodule update --init --recursive
make clean
make config-clang
make -j$(nproc)
sudo make install PREFIX=~/OpenROAD/OpenROAD-flow-scripts/tools/install/yosys
cd ..

# Build OpenROAD
./build_openroad.sh --local

# Source environment
source ./env.sh

# Verify installation
echo "Verifying installation..."
yosys -V
openroad -version

echo "✅ Installation complete!"
```

---

## Appendix: Flow Execution Script

```bash
#!/bin/bash
# Run floorplan and placement for GCD design

cd ~/OpenROAD/OpenROAD-flow-scripts/flow

# Set design
export DESIGN_CONFIG=./designs/nangate45/gcd/config.mk

# Clean previous runs
echo "Cleaning previous builds..."
make clean_all

# Run floorplan
echo "Running floorplan..."
make DESIGN_CONFIG=$DESIGN_CONFIG floorplan

if [ $? -eq 0 ]; then
    echo "✅ Floorplan completed successfully"
    ls -lh results/nangate45/gcd/base/2_*.odb
else
    echo "❌ Floorplan failed. Check logs:"
    echo "   cat logs/nangate45/gcd/base/2_1_floorplan.log"
    exit 1
fi

# Run placement
echo "Running placement..."
make DESIGN_CONFIG=$DESIGN_CONFIG place

if [ $? -eq 0 ]; then
    echo "✅ Placement completed successfully"
    ls -lh results/nangate45/gcd/base/3_*.odb
    
    # Display summary
    echo ""
    echo "=== Summary ==="
    grep -A 5 "Total Cells" reports/nangate45/gcd/base/3_place.rpt
    grep "Worst Slack" reports/nangate45/gcd/base/3_place_timing.rpt
else
    echo "❌ Placement failed. Check logs:"
    echo "   cat logs/nangate45/gcd/base/3_1_place_gp.log"
    exit 1
fi

# Open GUI
echo ""
echo "Opening placement in GUI..."
make DESIGN_CONFIG=$DESIGN_CONFIG gui_place
```

---

## Appendix: Error Pattern Reference

| Error Message | Root Cause | Quick Fix |
|---------------|------------|-----------|
| `yosys: invalid option --` | Version mismatch | Rebuild Yosys from repo |
| `No such frontend: verilog` | Missing Verilog support | `make config-clang && make` |
| `init_share_dirname error` | Missing share/ directory | `make install PREFIX=...` |
| `Permission denied` | Root-owned directories | `sudo chown -R $USER:$USER ...` |
| `Text file busy` | Process still running | `pkill -9 yosys` |
| `Could NOT find Boost` | Missing/old Boost | Build Boost 1.78 from source |
| `CUDD_LIB not found` | Missing CUDD library | Build CUDD from source |
| `Core area too small` | Utilization > 100% | Reduce `CORE_UTILIZATION` |
| `Cannot legalize` | Too many cells | Lower utilization or increase margin |

---

## Appendix: GUI Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Fit view to window | `F` |
| Zoom in | `Z` or Scroll Up |
| Zoom out | `Shift+Z` or Scroll Down |
| Pan view | Middle-click drag |
| Measure distance | Ruler icon, click 2 points |
| Select object | Left-click |
| Deselect all | `Esc` |
| Toggle layer visibility | Right panel checkboxes |
| Command console | Bottom panel |
| Quit | `Ctrl+Q` |

---

## Appendix: Debugging Flowchart

```
┌─────────────────────────────────┐
│   Installation Failed?          │
└─────────┬───────────────────────┘
          │
          ├─ Dependency missing? ────→ Check logs, install library
          ├─ Build error? ───────────→ Check compiler version (g++ 9+)
          └─ Submodule missing? ─────→ git submodule update --init --recursive

┌─────────────────────────────────┐
│   Synthesis Failed?             │
└─────────┬───────────────────────┘
          │
          ├─ Yosys not found? ───────→ Check: which yosys
          ├─ Verilog frontend? ──────→ Rebuild Yosys with config-clang
          └─ Read error? ────────────→ Check input .v file syntax

┌─────────────────────────────────┐
│   Floorplan Failed?             │
└─────────┬───────────────────────┘
          │
          ├─ Core too small? ────────→ Reduce CORE_UTILIZATION
          ├─ PDN error? ─────────────→ Check power grid configuration
          └─ IO conflict? ───────────→ Check constraint.sdc file

┌─────────────────────────────────┐
│   Placement Failed?             │
└─────────┬───────────────────────┘
          │
          ├─ Can't legalize? ────────→ Lower utilization
          ├─ Timing violation? ──────→ Relax constraints or resize cells
          └─ Congestion? ────────────→ Increase core area

┌─────────────────────────────────┐
│   GUI Won't Open?               │
└─────────┬───────────────────────┘
          │
          ├─ Display error? ─────────→ Check DISPLAY env variable
          ├─ Qt missing? ────────────→ Install Qt5: apt install qt5-default
          └─ File not found? ────────→ Check .odb file exists
```

---

**Tip:** Run `du -sh results/nangate45/gcd/` to check your usage

---

## Appendix: Additional Screenshot Locations

**Optional but valuable screenshots to add:**

### During Installation
- `images/installation/boost-build.png` - Boost compilation
- `images/installation/cudd-build.png` - CUDD compilation
- `images/installation/yosys-build.png` - Yosys compilation

### During Execution
- `images/execution/file-structure-before.png` - Empty results/ folder
- `images/execution/file-structure-after.png` - Generated files
- `images/execution/terminal-username.png` - Terminal with your username

### GUI Details
- `images/visualization/layers-panel.png` - Layer controls
- `images/visualization/measurement-tool.png` - Distance measurement
- `images/visualization/cell-properties.png` - Selected cell info

### Reports
- `images/floorplan/floorplan-report-terminal.png` - `cat 2_floorplan.rpt`
- `images/placement/placement-report-terminal.png` - `cat 3_place.rpt`
- `images/placement/timing-report-terminal.png` - `cat 3_place_timing.rpt`

### Troubleshooting (if you faced issues)
- `images/troubleshooting/error-screenshot.png` - Actual error
- `images/troubleshooting/fix-command.png` - Command that fixed it
- `images/troubleshooting/verification.png` - Successful re-run

---

## Final Checklist

### Installation
- [ ] All dependencies installed
- [ ] Yosys built with Verilog frontend
- [ ] OpenROAD built successfully
- [ ] Tools in PATH (`which yosys`, `which openroad`)

### Execution
- [ ] Synthesis completed (1_synth.v exists)
- [ ] Floorplan completed (2_floorplan.odb exists)
- [ ] Placement completed (3_place.odb exists)
- [ ] No errors in logs

### Visualization
- [ ] GUI opened successfully
- [ ] Floorplan shows rows and boundaries
- [ ] Placement shows all cells

### Reports
- [ ] `2_floorplan.rpt` shows utilization
- [ ] `3_place.rpt` shows cell count and HPWL
- [ ] `3_place_timing.rpt` shows positive slack

---

## Tips

1. **Check logs first:** `cat logs/nangate45/gcd/base/*.log`
2. **Search GitHub issues:** [OpenROAD Issues](https://github.com/The-OpenROAD-Project/OpenROAD/issues)
3. **Ask on Slack:** [OpenROAD Slack](https://openroad.slack.com)
4. **Reference documentation:** [OpenROAD Docs](https://openroad.readthedocs.io/)


- **Review reference repository:**  [ spatha_vsd-hdp](https://github.com/spatha0011/spatha_vsd-hdp/blob/main/Day14/README.md)

---
