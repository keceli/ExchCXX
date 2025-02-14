# About

ExchCXX Copyright (c) 2020-2022, The Regents of the University of California,
through Lawrence Berkeley National Laboratory (subject to receipt of
any required approvals from the U.S. Dept. of Energy). All rights reserved.

If you have questions about your rights to use or distribute this software,
please contact Berkeley Lab's Intellectual Property Office at
IPO@lbl.gov.

NOTICE.  This Software was developed under funding from the U.S. Department
of Energy and the U.S. Government consequently retains certain rights.  As
such, the U.S. Government has been granted for itself and others acting on
its behalf a paid-up, nonexclusive, irrevocable, worldwide license in the
Software to reproduce, distribute copies to the public, prepare derivative 
works, and perform publicly and display publicly, and to permit others to do so.

# Synopsis

ExchCXX is a modern C++ library for the evaluation of exchange-correlation (XC)
functionals required for density functional theory (DFT) calculations. In
addition to providing high-level wrappers for the ubiquitous
[Libxc](https://www.tddft.org/programs/libxc) library for the CPU evaluation of
XC functionals, ExchCXX also provides locally developed implementations of a
small subset of XC functionals which may be evaluated either on the host (CPU)
or device (GPU, FPGA, etc). Currently GPU support is provided through the
[CUDA](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html) for NVIDIA
GPUs, [HIP](https://rocmdocs.amd.com/en/latest/Programming_Guides/HIP-GUIDE.html) for
AMD GPUs and [SYCL](https://registry.khronos.org/SYCL/specs/sycl-2020/html/sycl-2020.html) (experimental)
for generic accelerator backends (including Intel GPUs).


ExchCXX is a work in progress. Its development has been funded by the U.S.
Department of Energy Exascale Computing Project 
([NWChemEx](https://github.com/NWChemEx-Project)).

# Design Goals

* Provide a modern C++ wrapper around various XC functional libraries (Libxc, XCFun, etc)
* Provide stable, portable and high-performance implementations for the evaluation of XC functionals on various accelerator architectures (GPUs, FPGAs, etc)


# Example Usage

```
// Setup the B3LYP Functional
using namespace ExchCXX;

auto backend = Backend::builtin; // Set to Backend::libxc for Libxc wrapper
auto func    = Functional::B3LYP;
auto polar   = Spin::Polarized; // Set to Unpolarized for unpolaried calculation

XCFunctional b3lyp( backend, func, polar );


// Evaluate on Host (CPU)
size_t npts = ...;
std::vector<double> 
  rho(b3lyp.rho_buffer_len(npts)), 
  sigma(b3lyp.sigma_buffer_len(npts)), 
  exc(b3lyp.exc_buffer_len(npts)), 
  vrho(b3lyp.vrho_buffer_len(npts)), 
  vsigma(b3lyp.vsigma_buffer_len(npts));

// Set up rho / gamma

b3lyp.eval_exc_vxc( npts, rho.data(), gamma.data(), exc.data(), vrho.data(),
  vgamma.data() );


// Evaluate on the Device (GPU)
cudaStream_t stream = ...;

double *rho_device, *gamma_device, *exc_device, *vrho_device, *vgamma_device;

// Allocate device memory
// Send rho / gamma to device

b3lyp.eval_exc_vxc_device( npts, rho_device, gamma_device, exc_device, vrho_device,
  vgamma_device, stream );

```


# Implemented XC Functional Interfaces

## `XCKernel`

| Name                                 | Libxc Identifier      | ExchCXX Identifier       | Device? |
|--------------------------------------|-----------------------|--------------------------|---------|
| Slater Exchange                      | `XC_LDA_X`            | `Kernel::SlaterExchange` |    Y    |
| Vosko-Wilk-Nusair III                | `XC_LDA_C_VWN_3`      | `Kernel::VWN3`           |    Y    |
| Vosko-Wilk-Nusair V                  | `XC_LDA_C_VWN_RPA`    | `Kernel::VWN5`           |    Y    |
| Perdew-Burke-Ernzerhof (Exchange)    | `XC_GGA_X_PBE`        | `Kernel::PBE_X`          |    Y    |
| Perdew-Burke-Ernzerhof (Correlation) | `XC_GGA_C_PBE`        | `Kernel::PBE_C`          |    Y    |
| Revised PBE from Zhang & Yang        | `XC_GGA_X_PBE_R`      | `Kernel::revPBE_X        |    Y    |
| Perdew-Wang 91 (LDA)                 | `XC_LDA_C_PW`         | `Kernel::PW91_LDA`       |    Y    |
| Perdew-Wang 91 (LDA) Modified        | `XC_LDA_C_PW_MOD`     | `Kernel::PW91_LDA_MOD`   |    Y    |
| Perdew-Wang 91 (LDA) RPA             | `XC_LDA_C_PW_RPA`     | `Kernel::PW91_LDA_RPA`   |    Y    |
| Perdew-Zunger 86                     | `XC_LDA_C_PZ`         | `Kernel::PZ86_LDA`       |    Y    |
| Perdew-Zunger 86 Modified            | `XC_LDA_C_PZ_MOD`     | `Kernel::PZ86_LDA_MOD`   |    Y    |
| Becke Exchange 88                    | `XC_GGA_X_B88`        | `Kernel::B88`            |    Y    |
| Lee-Yang-Parr                        | `XC_GGA_C_LYP`        | `Kernel::LYP`            |    Y    |
| PBE0                                 | `XC_HYB_GGA_XC_PBEH`  | `Kernel::PBE0`           |    Y    |
| B3LYP                                | `XC_HYB_GGA_XC_B3LYP` | `Kernel::B3LYP`          |    Y    |



# Feature Requests / Contributing

Although ExchCXX may in principle wrap any XC functional implemented by third-party software,
generating these interfaces must currently be done by hand. As such, if there is a functional
that you would like to see wrapped in either host or device code, please create a [GitHub issue](https://github.com/wavefunction91/ExchCXX/issues)
which details your request. Alternatively, we are more than happy to accept [pull-requests](https://github.com/wavefunction91/ExchCXX/pulls) if
you have made the changes locally and would like to see them integrated in upstream.


# License

ExchCXX is made freely available under the terms of a modified 3-Clause BSD license. See
LICENSE.txt for details.

# Acknowledgments

The development of ExchCXX is supported by the Exascale Computing Project
(17-SC-20-SC), a collaborative effort of the U.S. Department of Energy Office
of Science and the National Nuclear Security Administration.
