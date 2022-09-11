# CMake project for the NXP LPC55S69

This project is based on the [MCU on Eclipse](https://mcuoneclipse.com/) tutorial for [creating CMake projects for use with Eclipse](https://mcuoneclipse.com/2022/09/04/tutorial-creating-bare-bare-embedded-projects-with-cmake-with-eclipse-included/).

The primary chages made were to support CMakePresets, use vcpkg for toolchain acquisition, and enable VS and VS Code usage. The tutorial for using this with MCUXpresso IDE should still work (not validated yet).

A devcontainer has also been configured, but it has only been validated to run builds so far/


## vcpkg
Install vcpkg for your platform.

Run vcpkg activate in the root directory to acquire depenencies and have them configured for use in your shell.

## CMake
To build the project from a vcpkg activated shell run CMake configure for the desired preset, then build.
```
cmake --preset m33-debug
cmake --build --preset m33-debug
```

## Visual Studio
When using as a base for another project see .vs/launch.vs.json and update the debug target to your project name.

Embedded support was added in Visual Studio 2022. To use just open the folder in Visual Studio.

From VS2022 17.4 vcpkg activation will occur automatically, so long as it has been installed.

## Visual Studio Code
When using as a base for another project see .vscode/launch.json and flash.jlink and update the debug target to your project name.

Install the Embedded Tools extension. Run vcpkg activation and launch code from the root of your project, the dependencies will then be available in the VS Code context.

## CLion
See [CLion Embedded Development](https://www.jetbrains.com/help/clion/embedded-overview.html) for how to configure your toolchains and debugger. You should be able to use vcpkg activated toolchains with CLion through the standard configuration.

## Eclipse
See [creating CMake projects for use with Eclipse](https://mcuoneclipse.com/2022/09/04/tutorial-creating-bare-bare-embedded-projects-with-cmake-with-eclipse-included/).

## Containers
Run the below commands from the root of your project. The dockerfile is configured with a base development layer and build layer. The base layer sets up prerequisites using vcpkg and the project manifest. The build layer pulls in all sources and runs the build using CMake. Usage suggestions follow.

### Images
Name the interactive dev image for the target.
The idea here is that the artifacts are common across many, so you can spin up new containers from this image for different apps.
```
docker build --target dev -t arm-dev-image -f .\.devcontainer\Dockerfile .
```

Name the build image for the app being built.
```
docker build --target build -t app-build-image -f .\.devcontainer\Dockerfile .
```

### Containers
Name the container for easy use in subsequent actions, like extracting the build artifacts.
```
docker run --name app-build-container app-build-image
```
Get you binary out of your build container
```
docker cp app-build-container:/src/build/m33-debug/LPC55S69_cmake_template.elf .
```
Don't name the container to easily spin it up multiple times, easily discard them all later with ```docker container prune``` (deletes all containers not running).
```
docker run -it arm-dev-image
```
If you want to interactively see what is happening in your build image
```
docker container run -it app-build-image
```