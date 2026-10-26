# R0 lab note: machine and toolchain (2026-10-26)

Recorded on this PC on 2026-10-26, from commands run in this session. Values are copied from tool output, not assumed.

## Hardware and OS

- GPU: AMD Radeon RX 5600 XT (PCI `VEN_1002&DEV_731F`, subsystem `51201849`, revision `CA`)
- Windows driver (WMI `Win32_VideoController.DriverVersion`): `32.0.21045.16010`, driver date 2026-10-16
- Vulkan (`vulkaninfo --summary`): instance 1.4.309; device apiVersion 1.4.315; driverVersion field `2.0.353`; driverID `DRIVER_ID_AMD_PROPRIETARY`; driverInfo `26.10.1 (AMD proprietary shader compiler)`; conformanceVersion 1.4.0.0
- Device memory reported by torchvulkan: 6157238272 bytes
- OS: Windows 11 Home 10.0.26300

## Toolchain

- Python on PATH: 3.9.13. Too old for torchvulkan, which needs Python 3.10 or later.
- Other interpreters: Python 3.11 and 3.14 (found with `py -0p`)
- CMake 3.29.2; Git 2.49.0.windows.1
- `glslc`, `glslangValidator` and `slangc` were not found. `VULKAN_SDK` is unset.

## Vulkan layer noise

`vulkaninfo` printed load errors from a Medal capture overlay layer (`VK_LAYER_MEDAL_HOOK`), and a 32-bit hook DLL failed to open (error 126). Device enumeration was not affected. Recorded so that later runs can be compared.

## Python environments (in the session scratchpad, not in the repo)

- `venv311`: Python 3.11, `torch 2.14.1+cpu`, `torchvulkan 0.2.1` from PyPI, `torchvision 0.29.1`, `matplotlib 3.11.2`
- `venv_dml`: Python 3.11, `torch 2.4.1`, `torch-directml 0.2.5.dev240914`, `torchvision 0.19.1`

## Notes

- Installing PyPI `torchvulkan 0.2.1` pulled in `torch 2.14.1`. Its pyproject requires `torch>=2.10.0`.
- When queried, the PyTorch CPU wheel index on this PC listed `torch` only up to `2.8.0+cpu`. The `torch 2.14.1` install came from the default index. Check this before relying on either.
- Upstream source audited: `zainsharief/torchvulkan` at commit `078dd8ab0ed41aed41c6cbcd4def775b695fbd30` (2026-10-24), cloned into the session scratchpad.
