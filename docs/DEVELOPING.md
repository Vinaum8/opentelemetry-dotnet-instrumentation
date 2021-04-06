# Development

## Components

**[Datadog Agent](https://github.com/DataDog/datadog-agent)**: A service that runs on your application servers, accepting trace data from the Datadog Tracer and sending it to Datadog. The Agent is not part of this repo; it's the same Agent to which all Datadog tracers (e.g. Go, Python, Java, Ruby) send data.

**[Datadog .NET Tracer](https://github.com/DataDog/dd-trace-dotnet)**: This repository. A set of .NET libraries that let you trace any piece of your .NET code. Supports manual instrumentation and can automatically instrument supported libraries out-of-the-box.

## Windows

### Minimum requirements

- [Visual Studio 2019 (16.8)](https://visualstudio.microsoft.com/downloads/) or newer
  - Workloads
    - Desktop development with C++
    - .NET desktop development
    - .NET Core cross-platform development
    - Optional: ASP.NET and web development (to build samples)
  - Individual components
    - .NET Framework 4.7 targeting pack
- [.NET 5.0 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)
- [.NET Core 3.1 Runtime](https://dotnet.microsoft.com/download/dotnet-core/3.1)
- Optional: [.NET Core 3.0 Runtime](https://dotnet.microsoft.com/download/dotnet-core/3.0) to test in .NET Core 3.0 locally.
- Optional: [.NET Core 2.1 Runtime](https://dotnet.microsoft.com/download/dotnet-core/2.1) to test in .NET Core 2.1 locally.
- Optional: [nuget.exe CLI](https://www.nuget.org/downloads) v5.3 or newer
- Optional: [WiX Toolset 3.11.1](http://wixtoolset.org/releases/) or newer to build Windows installer (msi)
  - Requires .NET Framework 3.5 SP2 (install from Windows Features control panel: `OptionalFeatures.exe`)
  - [WiX Toolset Visual Studio Extension](https://wixtoolset.org/releases/) to build installer from Visual Studio
- Optional: [Docker for Windows](https://docs.docker.com/docker-for-windows/) to build Linux binaries and run integration tests on Linux containers. See [section on Docker Compose](#building-and-running-tests-with-docker-compose).
  - Requires Windows 10 (1607 Anniversary Update, Build 14393 or newer)

Microsoft provides [evaluation developer VMs](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines) with Windows 10 and Visual Studio pre-installed.

### Building from a command line

From a _Developer Command Prompt for VS 2019_:

```cmd
rem Restore NuGet packages
rem nuget.exe is required for command line restore because msbuild doesn't support packages.config
rem (see https://github.com/NuGet/Home/issues/7386)
nuget restore Datadog.Trace.sln

rem Build C# projects (Platform: always AnyCPU)
msbuild Datadog.Trace.proj /t:BuildCsharp /p:Configuration=Release

rem Build NuGet packages
dotnet pack src\Datadog.Trace\Datadog.Trace.csproj
dotnet pack src\Datadog.Trace.OpenTracing\Datadog.Trace.OpenTracing.csproj

rem Build C++ projects
rem The native profiler depends on the Datadog.Trace.ClrProfiler.Managed.Loader C# project so be sure that is built first
msbuild Datadog.Trace.proj /t:BuildCpp /p:Configuration=Release;Platform=x64
msbuild Datadog.Trace.proj /t:BuildCpp /p:Configuration=Release;Platform=x86

rem Build MSI installer for Windows x64 (supports both x64 and x86 apps)
msbuild Datadog.Trace.proj /t:msi /p:Configuration=Release;Platform=x64

rem Build MSI installer for Windows x86 (supports x86 apps only)
msbuild Datadog.Trace.proj /t:msi /p:Configuration=Release;Platform=x86

rem Build tracer home directory for Windows.
rem Valid values for property `Platform` are `x64`, `x86`, and `All`.
msbuild Datadog.Trace.proj /t:CreateHomeDirectory /p:Configuration=Release;Platform=All
```

## Linux

### Minimum requirements

To build C# projects and NuGet packages only
- [.NET 5.0 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)
- [.NET Core 3.1 Runtime](https://dotnet.microsoft.com/download/dotnet-core/3.1)
- Optional: [.NET Core 3.0 Runtime](https://dotnet.microsoft.com/download/dotnet-core/3.0) to test in .NET Core 3.0 locally.
- Optional: [.NET Core 2.1 Runtime](https://dotnet.microsoft.com/download/dotnet-core/2.1) to test in .NET Core 2.1 locally.

To build everything and run integration tests
- [Docker Compose](https://docs.docker.com/compose/install/)

### Building and running tests with Docker Compose

You can use [Docker Compose](https://docs.docker.com/compose/) with Linux containers to build Linux binaries and run the test suites. This works on both Linux and Windows hosts.

```bash
# build C# projects
docker-compose run build

# build C++ project
docker-compose run Profiler

# run integration tests
docker-compose run IntegrationTests
```

## Visual Studio Code

This repository contains example configuration for VS Code located under `.vscode.example`. You can copy it to `.vscode`.

```sh
cp -r .vscode.example .vscode
```

### OmniSharp issues

Because of [Mono missing features](https://github.com/OmniSharp/omnisharp-vscode#note-about-using-net-5-sdks), `omnisharp.useGlobalMono` has to be set to `never`. Go to `File` -> `Preferences` -> `Settings` -> `Extensions` -> `C# Configuration` -> Change `Omnisharp: Use Global Mono` (you can search for it if the menu is too long) to `never`. Afterwards, you have restart OmniSharp: `F1` -> `OmniSharp: Restart OmniSharp`.

There may be a lot of errors, because some projects target .NET Framework. Switch to `Datadog.Trace.Minimal.sln` using `F1` -> `OmniSharp: Select Project` in Visual Studio Code to load a subset of projects which work without any issues. You can also try building the projects which have errors as it sometimes helps.

If for whatever reason you need to use `Datadog.Trace.sln` you can run `for i in **/*.csproj; do dotnet build $i; done` to decrease the number of errors.

### Development Container

The repository also contains configuration for developing inside a Container ([installation steps](https://code.visualstudio.com/docs/remote/containers#_installation)) using [Visual Studio Code Remote - Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) located under `.devcontainer.example`. You can copy it to `.devcontainer`.

```sh
cp -r .devcontainer.example .devcontainer
```

The Development Container configuration mixes [Docker in Docker](https://github.com/microsoft/vscode-dev-containers/tree/master/containers/docker-in-docker) and [C# (.NET)](https://github.com/microsoft/vscode-dev-containers/tree/master/containers/dotnet) definitions. Thanks to it you can use `docker` and `docker-compose` inside the container.


## Further Reading

Datadog APM
- [Datadog APM](https://docs.datadoghq.com/tracing/)
- [Datadog APM - Tracing .NET Core and .NET 5 Applications](https://docs.datadoghq.com/tracing/setup_overview/setup/dotnet-core)
- [Datadog APM - Tracing .NET Framework Applications](https://docs.datadoghq.com/tracing/setup_overview/setup/dotnet-framework)

Microsoft .NET Profiling APIs
- [Profiling API](https://docs.microsoft.com/en-us/dotnet/framework/unmanaged-api/profiling/)
- [Metadata API](https://docs.microsoft.com/en-us/dotnet/framework/unmanaged-api/metadata/)
- [The Book of the Runtime - Profiling](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/profiling.md)

OpenTracing
- [OpenTracing documentation](https://github.com/opentracing/opentracing-csharp)
- [OpenTracing terminology](https://github.com/opentracing/specification/blob/master/specification.md)

## Get in touch

If you have questions, feedback, or feature requests, reach our [support](https://docs.datadoghq.com/help).