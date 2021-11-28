---
published: true
title: "Removing Microsoft Edge using PowerShell"
excerpt: "Removing Microsoft Edge using PowerShell"
date: 2021-11-28T00:00:00-04:00
show_date: true
tags:
  - Microsoft Edge
  - PowerShell
  - Windows 10
  - Windows 11
---

## Locate setup.exe

First locate the `setup.exe` file, which will be used to uninstall Microsoft Edge.

You should be able to find it in the path below:

```text
 💽 C:
    └── 📂Program Files (x86)
        └── 📂Microsoft
            └── 📂Edge
                └── 📂Application
                    └── 📂<version number>
                        └── 📂Installer
                            ├── ...
                            └── ⚙️setup.exe 👈
```

For example: `C:\Program Files (x86)\Microsoft\Edge\Application\96.0.1054.34\Installer`.

## PowerShell script

The below script will uninstall Microsoft Edge [stable channel](https://docs.microsoft.com/en-us/deployedge/microsoft-edge-channels):

```powershell
$EdgeVersion = (Get-AppxPackage "Microsoft.MicrosoftEdge.Stable" -AllUsers).Version
$EdgeSetupPath = ${env:ProgramFiles(x86)} + '\Microsoft\Edge\Application\' + $EdgeVersion + '\Installer\setup.exe'
& $EdgeSetupPath --uninstall --system-level --verbose-logging --force-uninstall
```

Copy and paste the above script, launch **Windows PowerShell ISE** as an Administrator and execute using the *▶️ Run Script* button, or hit `F5`.

![powershell-ise](/assets/images/powershell-ise.png)

Some more details on each step can be found below.

### Steps explained

Use the [Get-AppxPackage](https://docs.microsoft.com/en-us/powershell/module/appx/get-appxpackage) function to retrieve the Microsoft Edge version number:

```powershell
$EdgeVersion = (Get-AppxPackage "Microsoft.MicrosoftEdge.Stable" -AllUsers).Version
```

Construct the absolute path which contains the `setup.exe` file:

```powershell
$EdgeSetupPath = ${env:ProgramFiles(x86)} + '\Microsoft\Edge\Application\' + $EdgeVersion + '\Installer\setup.exe'
```

Use the [Call operator `&`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_operators), to run the actual uninstall operation and add the following 🚩flags:

- `--uninstall`
- `--system-level`
- `--verbose-logging`
- `--force-uninstall`

```powershell
& $EdgeSetupPath --uninstall --system-level --verbose-logging --force-uninstall
```

![and-its-gone](/assets/images/and-its-gone.gif){: .align-center}
