# Running a Flutter application

## Prerequisites

- Build artifacts (engine, embedder)
- Tizen device (Tizen 5.5 or above)
  - Wearable: Fully supported
  - TV: Not supported
- Flutter SDK for Linux
  - https://flutter.dev/docs/get-started/install/linux
- Tizen SDK and IDE
  - [Visual Studio](https://docs.tizen.org/application/vstools/install) (recommended)
  - [Visual Studio Code](https://docs.tizen.org/application/vscode-ext/dotnet)
  - [Visual Studio for Mac](https://docs.tizen.org/application/vstools-mac/overview)

## How to use

1. Download sample app code from [flutter/samples](https://github.com/flutter/samples.git).

2. Choose one of the samples and build.

   ```bash
   cd jsonexample
   flutter create .
   flutter build bundle # add --debug option for Debug build
   ```

3. Generate AOT compiled code (for Release build).

   ```bash
   mkdir -p build/linux/aot

   # Generate app.dill
   <Flutter SDK root>/bin/cache/dart-sdk/bin/dart \
   <Flutter SDK root>/bin/cache/artifacts/engine/linux-x64/frontend_server.dart.snapshot \
   --sdk-root <Flutter SDK root>/bin/cache/artifacts/engine/common/flutter_patched_sdk_product/ \
   --target=flutter \
   -Ddart.developer.causal_async_stacks=true \
   -Ddart.vm.profile=false \
   -Ddart.vm.product=true \
   --bytecode-options=source-positions \
   --aot \
   --tfa \
   --packages .packages \
   --output-dill build/linux/aot/app.dill \
   --depfile build/linux/aot/kernel_compile.d \
   package:jsonexample/main.dart

   # Generate libapp.so
   <Flutter engine root>/src/out/linux_release_arm/clang_x64/gen_snapshot \
   --causal_async_stacks \
   --deterministic \
   --snapshot_kind=app-aot-elf \
   --elf=build/linux/aot/libapp.so \
   --strip \
   --no-sim-use-hardfp \
   --no-use-integer-division \
   build/linux/aot/app.dill
   ```

4. Copy files to corresponding directories. (Replace the files that I provided.)

   - From `samples/jsonexample`
     - `build/linux/aot/libapp.so` → `dotnet-host/lib/armel`
     - `build/flutter_assets/*` → `dotnet-host/res/flutter_assets`
   - From `embedder`
     - `out/icudtl.dat` → `dotnet-host/res`
     - `out/libflutter_engine.so` → `dotnet-host/lib/armel`
     - `out/libflutter_embedder.so` → `dotnet-host/lib/armel`

5. Open `dotnet-host/FlutterApplication.sln` in Visual Studio.

   - The native (C/C++) host will be provided later.

6. Build and run.
