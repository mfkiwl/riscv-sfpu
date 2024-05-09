# FPU-SP Single Precision

This floating point unit is conform to IEEE 754-2008 standards. Supported operations are **compare**, **min-max**, **conversions**, **addition**, **subtruction**, **multiplication**, **fused multiply add**, **square root** and **division** in single precisions. Except **square root** and **division** all operations are pipelined.

This unit uses canonical **nan** (not a number) form, if it generates any **nan** as output. E.g. 0x7FC00000. Therefore extra conditions are added in order compare outputs with testfloat data correctly by **nan** generation.

**Square root** and **division** calculations are using same subunit but different path (algorithm). This subunit has a generic variable **_performance_**. For functional iterations which are fast please use **1** and fixed point iterations which are slow use **0**. This unit has also own multiplier.

## DESIGN

This floating point unit transform the single precision to the pseudo extended precision. The main benefit of this implementation is that the design needs few resources because we do not implement extension in pipeline to handle subnormal numbers. It means that all floating numbers are normalized thanks to pseudo extended precision.

|        | sign | exponent | mantissa |
|:------:|:----:|:--------:|:--------:|
| single | 1    | 8        | 23       |
| pseudo | 1    | 9        | 23       |

## LATENCY

| comp | max | conv | add | sub | mul | fma |
|:----:|:---:|:----:|:---:|:---:|:---:|:---:|
| 1    | 1   | 1    | 3   | 3   | 3   | 3   |

|performance| division | square root |
|:---------:|:--------:|:-----------:|
| 0         | 29       | 28          |
| 1         | 12       | 14          |

## SIGNALS

### INPUT

- fp_unit_i.fp_exe_i.data1 => rs1
- fp_unit_i.fp_exe_i.data2 => rs2
- fp_unit_i.fp_exe_i.data3 => rs3
- fp_unit_i.fp_exe_i.op => operation type
- fp_unit_i.fp_exe_i.fmt => precision type
- fp_unit_i.fp_exe_i.rm => rounding mode
- fp_unit_i.fp_exe_i.enable => activate unit

#### TYPE

| op | type |
|:---|:-----|
| fmadd | $(rs1×rs2)+rs3$ |
| fmsub | $(rs1×rs2)-rs3$ |
| fnmsub | $-(rs1×rs2)+rs3$ |
| fnmadd | $-(rs1×rs2)-rs3$ |
| fadd | $rs1+rs2$ |
| fsub | $rs1-rs2$ |
| fmul | $rs1*rs2$ |
| fdiv | $rs1/rs2$ |
| fsqrt | $\sqrt{rs1}$ |

| op | rm | type |
|:---|:---|:-----|
| fsgnj | "000" | J |
| fsgnj | "001" | JN |
| fsgnj | "010" | JX |

| op | rm | type |
|:---|:---|:-----|
| fcmp | "000" | EQ |
| fcmp | "001" | LT |
| fcmp | "010" | LE |

| op | rm | type |
|:---|:---|:-----|
| fmax | "000" | MIN |
| fmax | "001" | MAX |

| op | op.fcvt_op | fmt | type |
|:---|:-----------|:----|:-----|
| fcvt_f2i | "00" | "00" | CONVERSION FROM FLOAT TO INT32  |
| fcvt_f2i | "01" | "00" | CONVERSION FROM FLOAT TO UINT32 |

| op | op.fcvt_op | fmt | type |
|:---|:-----------|:----|:-----|
| fcvt_i2f | "00" | "00" | CONVERSION FROM INT32 TO FLOAT  |
| fcvt_i2f | "01" | "00" | CONVERSION FROM UINT32 TO FLOAT |

| fmt | type |
|:----|:-----|
| "00" | FLOAT |

| rm | type |
|:---|:-----|
| "000" | RNE |
| "001" | RTZ |
| "010" | RDN |
| "011" | RUP |
| "100" | RMM |

### OUTPUT

- fp_unit_o.fp_exe_o.result => calculation result
- fp_unit_o.fp_exe_o.flags => exception flags
- fp_unit_o.fp_exe_o.ready => ready status

#### TYPE

| flags | type |
|:------|:-----|
| "00001" | NX |
| "00010" | UF |
| "00100" | OF |
| "01000" | DZ |
| "10000" | NV |

## TOOLS

The installation scripts of necessary tools are located in directory **tools**. These scripts need **root** permission in order to install packages and tools for simulation and testcase generation. Please run these scripts in directory **tools** locally.

## GENERATE

To generate test cases you could use following command:

```console
make generate
```

## SIMULATION

To simulate the design together with generated test cases you could run following command:

```console
make simulate
```

This command requires two options for hardware description languages **VERILOG** and **VHDL**. The possible settings of these options can be found in the makefile.

Some example executions of this command look like as follows:

```console
make simulate VERILOG=1
make simulate VHDL=1
```

