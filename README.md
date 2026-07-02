# [zsdl](https://github.com/zig-gamedev/zsdl)

Zigified bindings for SDL libs. Work in progress.

## Getting started (SDL2)

Example `build.zig`:

```zig
pub fn build(b: *std.Build) !void {

    const exe = b.addExecutable(.{ ... });
    exe.linkLibC();

    const zsdl = b.dependency("zsdl", .{});

    exe.root_module.addImport("zsdl2", zsdl.module("zsdl2"));
    exe.root_module.addImport("zsdl2_ttf", zsdl.module("zsdl2_ttf"));
    exe.root_module.addImport("zsdl2_image", zsdl.module("zsdl2_image"));

    // Link against SDL libs
    linkSdlLibs(exe);
}
```

Link against SDL:

```zig
pub fn linkSdlLibs(compile_step: *std.Build.Step.Compile) void {
    // Adjust as needed for the libraries you are using.
    switch (compile_step.rootModuleTarget().os.tag) {
        .windows => {
            compile_step.root_module.linkSystemLibrary("SDL2", .{});
            compile_step.root_module.linkSystemLibrary("SDL2main", .{}); // Only needed for SDL2, not ttf or image
            compile_step.root_module.linkSystemLibrary("SDL2_ttf", .{});
            compile_step.root_module.linkSystemLibrary("SDL2_image", .{});
        },
        .linux => {
            compile_step.root_module.linkSystemLibrary("SDL2", .{});
            compile_step.root_module.linkSystemLibrary("SDL2_ttf", .{});
            compile_step.root_module.linkSystemLibrary("SDL2_image", .{});
        },
        .macos => {
            compile_step.root_module.linkFramework("SDL2", .{});
            compile_step.root_module.linkFramework("SDL2_ttf", .{});
            compile_step.root_module.linkFramework("SDL2_image", .{});
        },
        else => {},
    }
}
```

### Using prebuilt libraries

To use the prebuilt libraries, add the following to your `build.zig`:

```zig
pub fn build(b: *std.Build) !void {

    // ... other build steps ...

    // Use prebuilt libs instead of relying on system-installed SDL.
    @import("zsdl").prebuilt_sdl2.addLibraryPathsTo(exe);
    if (@import("zsdl").prebuilt_sdl2.install(b, target.result, .bin, .{
        .ttf = true,
        .image = true,
    })) |install_sdl2_step| {
        b.getInstallStep().dependOn(install_sdl2_step);
    }

    // Prebuilt libraries are installed to the executable directory. Set the RPath so the
    // executable knows where to look at runtime.
    switch (exe.rootModuleTarget().os.tag) {
        .windows => {}, // rpath is not used on Windows
        .linux => exe.root_module.addRPathSpecial("$ORIGIN"),
        .macos => exe.root_module.addRPathSpecial("@executable_path"),
        else => {},
    }
}
```

### Using zsdl2 in your code

```zig
const std = @import("std");
const sdl = @import("zsdl2");

pub fn main() !void {
    ...
    try sdl.init(.{ .audio = true, .video = true });
    defer sdl.quit();

    const window = try sdl.Window.create(
        "zig-gamedev-window",
        sdl.Window.pos_undefined,
        sdl.Window.pos_undefined,
        600,
        600,
        .{ .opengl = true, .allow_highdpi = true },
    );
    defer window.destroy();
    ...
}
```

## Getting started (SDL3)

SDL3 is built from source with the Zig build system (via
[castholm/SDL](https://github.com/castholm/SDL)) and linked statically, so no
prebuilt or system-installed SDL3 is required. Requires Zig 0.16.0 or newer.

Add the dependencies to your project:

```sh
zig fetch --save=zsdl git+https://github.com/zig-gamedev/zsdl.git
zig fetch --save=sdl git+https://github.com/castholm/SDL.git
```

Example `build.zig`:

```zig
pub fn build(b: *std.Build) !void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const exe = b.addExecutable(.{ ... });

    const zsdl = b.dependency("zsdl", .{});
    exe.root_module.addImport("zsdl3", zsdl.module("zsdl3"));

    // Build SDL3 from source and link it statically.
    const sdl = b.dependency("sdl", .{
        .target = target,
        .optimize = optimize,
    });
    exe.root_module.linkLibrary(sdl.artifact("SDL3"));
}
```
