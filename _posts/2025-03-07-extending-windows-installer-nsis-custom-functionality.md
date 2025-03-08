---
layout: post
title: "Extending the Windows Installer (NSIS) with Custom Functionality"
description: "A comprehensive guide to adding custom functionality to your Windows installer using NSIS, with real-world examples including CUDA detection and installation"
date: 2025-03-07
last_modified_at: 2025-03-07
image: /assets/images/posts/nsis-installer.jpg
image_alt: "NSIS Windows Installer Configuration Interface"
author: Alex Ash
keywords: nsis, windows installer, electron-builder, cuda, custom installer, windows setup
tags: [windows, development, tutorial, electron]
---

This guide explains how to add custom functionality to your application's Windows installer using NSIS (Nullsoft Scriptable Install System). We'll use a real-world example of adding CUDA detection and installation during setup.

## Overview

NSIS is a professional open-source system to create Windows installers. When building with electron-builder, NSIS is used as the default Windows installer. You can extend its functionality by:
- Adding custom initialization code
- Including additional NSIS plugins
- Adding custom pages to the installer
- Running checks before/during installation
- Downloading and installing additional components

## Basic Setup

First, configure your package.json to use custom NSIS scripts:

```json
{
    "nsis": {
        "oneClick": false,
        "allowToChangeInstallationDirectory": true,
        "include": "build/installer.nsh",
        "perMachine": true,
        "allowElevation": true
    }
}
```

## Adding Required NSIS Plugins

Add NSIS plugin packages as devDependencies in package.json:

```json
{
    "devDependencies": {
        "nsisextra": "^1.0.0",
        "nsis-wmi": "^1.0.0",
        "nsis-inet-plugin": "^1.0.1"
    }
}
```

## Creating Custom NSIS Scripts

Create a file named installer.nsh in your build directory. This file will contain your custom NSIS code.

Basic structure of installer.nsh:

```nsis
!macro customInit
    ; Your initialization code here
    Function MyCustomFunction
        ; Function code
    FunctionEnd

    Function .onInit
        ; Code to run at installer startup
        Call MyCustomFunction
    FunctionEnd
!macroend
```

## Real-World Example: CUDA Detection and Installation

Here's a complete example that shows how to:
- Detect NVIDIA GPUs
- Check CUDA installation
- Download and install CUDA if needed

### Step 1: Create Variables

```nsis
!macro customInit
    Var cudaVersion
    Var driverVersion
    Var cudaInstaller
    Var cudaInstallerUrl
```

### Step 2: Create Detection Functions

```nsis
Function DetectNvidiaGPU
    nsExec::ExecToStack 'wmic path win32_videocontroller get name'
    Pop $0
    Pop $1
    ${If} $1 contains "NVIDIA"
        StrCpy $0 "1"
    ${Else}
        StrCpy $0 "0"
    ${EndIf}
    Push $0
FunctionEnd
```

### Step 3: Check Driver Version

```nsis
Function GetNvidiaDriverVersion
    ReadRegStr $0 HKLM "SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{B2FE1952-0186-46C3-BAEC-A80AA35AC5B8}_Display.Driver" "DisplayVersion"
    Push $0
FunctionEnd
```

### Step 4: Download and Install

```nsis
Function InstallCuda
    DetailPrint "Downloading CUDA installer..."
    inetc::get "$cudaInstallerUrl" "$TEMP\cuda_installer.exe" /END
    ExecWait '"$TEMP\cuda_installer.exe" /s /noreboot'
    Delete "$TEMP\cuda_installer.exe"
FunctionEnd
```

## Best Practices

1. Always provide feedback to users about what's happening
   - Use DetailPrint for installer progress
   - Use MessageBox for important decisions/information

2. Handle errors gracefully
   - Check return values from commands
   - Provide fallback options
   - Don't abort installation unless absolutely necessary

3. Clean up after yourself
   - Remove temporary files
   - Close handles and connections
   - Don't leave partial installations

4. Respect user choice
   - Ask before downloading large files
   - Provide skip options for optional components
   - Remember user preferences

## Common NSIS Commands

```nsis
MessageBox MB_YESNO|MB_ICONQUESTION "Question?" IDNO label
    ; Shows a message box with Yes/No buttons

DetailPrint "Message"
    ; Shows progress in the installer

ExecWait "command" $0
    ; Runs a command and waits for completion

ReadRegStr $0 HKLM "Registry\Path" "ValueName"
    ; Reads from Windows Registry

${If} condition
${ElseIf} condition
${EndIf}
    ; Conditional statements
```

## Debugging Tips

1. Use DetailPrint to log progress
2. Test installer with logging enabled:
   ```
   hello-world-setup.exe /LOG=installer.log
   ```
3. Check Windows Event Viewer for errors
4. Test on clean virtual machines
5. Verify elevated privileges when needed

## Resources

- [NSIS Documentation](https://nsis.sourceforge.io/Docs/)
- [electron-builder NSIS](https://www.electron.build/nsis.html)
- [NSIS Plugin List](https://nsis.sourceforge.io/Category:Plugins)

## Need Help?

Reach out to me on X [@alexheloai](https://twitter.com/alexheloai)

---

*About the Author:*
Alex Ash is an AI engineer focusing on automation and cross-platform development. Contact at alex@helosolutions.ai or follow [@alexheloai](https://twitter.com/alexheloai) for more tutorials.