## MLXlink PCIe port 

```bash
#Access Mellanox Software Tools 
[root@mlnx01~]# mst start 

[root@mlnx01~]# mst staus 

mlxlink -d <device> --port_type PCIE
# view speed and width 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --port_type PCIE 

# performance counter information 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --port_type PCIE -c
RX Errors   : 0 ---> Number of transitions to recovery due to framing erros and crc errors 
TX Errors   : 0 ---> Number of transitions to recovery due to EIOS and TS errors 
CRC Error dllp: 0 ---> Indicates the crc error in data link layer packets 
CRC Error tlp: 0 ---> Indicates the crc error in transaction layer packets 

#Physical grade and eye opening information
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --port_type PCIE  -e
# User manual to pass fail criteria 

# all in once 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --port_type PCIE -ce
```

## MLXlink Fabric ports

```bash
#link status 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1
```
# Types of troubleshooting  
- Hardware Issue : replace cable or module 
- Cable doesn't support speed 
- Module/cable requires more power than system can supply
- Configuration issues 
- High insertion loss 

# Check link quality using MLXLINK

```bash
# SerDes values
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --show_serdes_tx

# set valules 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --serdes_tx <new_value eg. 1,100,10,10,17,0> --serdes_tx_lane 0

## ***Link quality is always measured in receive side, so must check the effect of new values on the link peer receive sides
Receive device---------------------------Mellanox(optics)[PCIe]

# FEC 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --show_fec

# change FEC mode
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --fec NF

# for FEC change to take effect, toogle the link
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 -a tg

# verify using fec value using show_fec again

## speed 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 -s 50G

# toggle the link and verify 

# activate prbs mode 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --test_mode en --rx_prbs prbs31 --tx_prbs prbs31

# RX tuning 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 --test_mode tu

# PRBS needs to be enabled on both sides for locked state 

## Module information 
[root@mlnx01~]#  mlxlink -d /dev/mst/mt4119_pciconf0.1 -m
```

## MLXCables 

- EEPROM : Non-volatile configuration memory on optics/AOC
- Basic information viewing:
        - Part Number 
        - Serial number 
        - Supported speeds 
- Can view Digital Diagnostic Monitoring (DDM) information 
- Need to make sure MFT is installed 

```bash
# for infiniband _ib
[root@mlnx01~]#  mst cable add -with_ib

# view cable information 
[root@mlnx01~]# mlxcables

# cable specification in raw format 
[root@mlnx01~]# mlxcables -d <cabe name eg: CA_MT4115_mtacad05_HCA-1_lid-0x0004,mlx5_0,1_cable> --dump

# registers 
[root@mlnx01~]# mlxcables -d <cabe name eg: CA_MT4115_mtacad05_HCA-1_lid-0x0004,mlx5_0,1_cable> --read_all_regs

# can compare to system expectations SFF-8636 for QSFP , SFF-8472 for SFP

# diagnostic information viewing for optic cable 
[root@mlnx01~]# mlxcables -d <cabe name eg: mt4115_pciconf0_cable_0> --DDM

# write new values 
# read 
[root@mlnx01~]# mlxcables -d <cabe name eg: mt4115_pciconf0_cable_0> -r -p0 -o98

# write 
[root@mlnx01~]# mlxcables -d <cabe name eg: mt4115_pciconf0_cable_0> -w <value eg: 0x01> -p0 -o98

# help
[root@mlnx01~]# mlxcables -h