# Trimui Smart Pro Build System

A build system for creating applications for the Trimui Smart Pro. It provides a Dockerfile with the cross-toolchain and SDK, and a CMake project that builds your application together with the bundle metadata the launcher expects.

## Setup Instructions

### 1. Clone this repository

```bash
git clone https://github.com/sotlef/trimui-smart-pro-build-system.git
cd trimui-smart-pro-build-system
```

### 2. Build and run the Docker container
The Dockerfile will automatically download and extract all required SDK components from the official Trimui GitHub <a href="https://github.com/trimui/toolchain_sdk_smartpro/releases" target="_blank">repository</a> during the build process.
No manual downloading or setup is needed.

For macOS (Arm):
```bash
docker build --platform linux/amd64 -t trimui-sdk .
docker run -it --rm -v $(pwd):/app trimui-sdk bash
```

For Linux:
```bash
docker build -t trimui-sdk .
docker run -it --rm -v $(pwd):/app trimui-sdk bash
```

For Windows (PowerShell):
```powershell
docker build -t trimui-sdk .
docker run -it --rm -v ${PWD}:/app trimui-sdk bash
```

### 3. Build your application

Inside the Docker container:

```bash
cmake -S . -B cmake-build -DCMAKE_TOOLCHAIN_FILE=toolchains/gcc-trimui-x64.cmake -DCMAKE_BUILD_TYPE=Release
cmake --build cmake-build -j
```

SDL2, freetype, GLES and EGL are resolved through `pkg-config` against the SDK sysroot, so adding new dependencies is just another `pkg_check_modules(...)` call in `CMakeLists.txt`.

Override the bundle name/description from the command line if you want:

```bash
cmake -S . -B cmake-build -DCMAKE_TOOLCHAIN_FILE=toolchains/gcc-trimui-x64.cmake \
      -DAPP_LABEL="My App" -DAPP_DESCRIPTION="Whatever"
```

### 4. Deploying to the Trimui Smart Pro

After building, the complete application will be in `cmake-build/[APP_FOLDER]/`. Copy the entire app folder to the `/SDCARD/Apps` folder on your Trimui Smart Pro.

## App Structure
```
cmake-build/[APP_FOLDER]/
├── Your_Compiled_App   # Your compiled app
├── icon.png            # Your app's icon
├── config.json         # App metadata
├── launch.sh           # Launch script
└── res/                # Resources
    └── arial.ttf       # App font
```

## Customizing Your Application

1. Override `APP_LABEL` and `APP_DESCRIPTION` on the `cmake` command line (or change their defaults in `CMakeLists.txt`)
2. Replace `res/icon.png` with your own app icon

## Controller Tester Application
The included `main.cpp` file is a controller testing app that allows you to visualize all the controller inputs for your device.
![Controller Tester Screenshot](readme/capture.png)
## License
Distributed under the MIT License. See `LICENSE` for more information.
