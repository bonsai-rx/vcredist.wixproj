# vcredist.wixproj

WiX bootstrapper support for the Microsoft Visual C++ Redistributable.

The `vcredist.wixproj.vc143` package adds the official Microsoft Visual C++ 2015-2022 Redistributable installers to a WiX bootstrapper project. The installers are downloaded from Microsoft when the project builds, checked against a pinned SHA-256 hash, and passed to WiX as preprocessor variables, so the package itself contains only build scripts.

## Usage

Reference the package from the WiX project. A WiX v3 project that uses `packages.config` also imports `build\vcredist.wixproj.vc143.targets` from the package, which Visual Studio adds when installing it. The first build needs internet access to download the installers, and later builds reuse them from the intermediate output folder.

The package defines the following preprocessor variables:

- `VCRedistX64Path` and `VCRedistX86Path`, the paths of the downloaded installers.
- `VCRedistVersion`, `VCRedistMinor` and `VCRedistBuild`, the version of the redistributable.

A bundle can install the redistributable, and skip it when the same or a later version is already installed:

```xml
<util:RegistrySearch Root="HKLM" Key="SOFTWARE\Microsoft\VisualStudio\14.0\VC\Runtimes\x64" Value="Minor" Variable="VCRedistX64Minor" Win64="yes" />
<util:RegistrySearch Root="HKLM" Key="SOFTWARE\Microsoft\VisualStudio\14.0\VC\Runtimes\x64" Value="Bld" Variable="VCRedistX64Build" Win64="yes" />

<Chain>
  <ExePackage Id="VCRedistX64"
              SourceFile="$(var.VCRedistX64Path)"
              InstallCommand="/install /quiet /norestart"
              Permanent="yes"
              InstallCondition="VersionNT64"
              DetectCondition="VCRedistX64Minor &gt; $(var.VCRedistMinor) OR (VCRedistX64Minor = $(var.VCRedistMinor) AND VCRedistX64Build &gt;= $(var.VCRedistBuild))">
    <ExitCode Value="1638" Behavior="success" />
  </ExePackage>
</Chain>
```

The installer returns exit code 1638 when a later version is already installed.

## License

The build scripts in this repository are licensed under the MIT License. The downloaded installers are distributed by Microsoft under the [Microsoft Visual C++ Redistributable license terms](https://aka.ms/VCRedistLicense).
