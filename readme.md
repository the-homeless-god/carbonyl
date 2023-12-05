`wb` is a Chromium based browser built to run in a terminal. [Read the blog post](https://fathy.fr/carbonyl).

It supports pretty much all Web APIs including WebGL, WebGPU, audio and video playback, animations, etc..

It's snappy, starts in less than a second, runs at 60 FPS, and idles at 0% CPU usage. It does not require a window server (i.e. works in a safe-mode console), and even runs through SSH.

Based on [`html2svg`](https://github.com/fathyb/html2svg) and is now the runtime behind it.

## Usage

> `wb` on Linux without Docker requires the same dependencies as Chromium.

### Docker

```shell
$ docker run --rm -ti the-homeless-god/wb https://youtube.com
```

### npm

```console
$ npm install --global wb
$ wb https://github.com
```

### Binaries

- [macOS amd64](https://github.com/the-homeless-god/wb/releases/download/v0.0.3/wb.macos-amd64.zip)
- [macOS arm64](https://github.com/the-homeless-god/wb/releases/download/v0.0.3/wb.macos-arm64.zip)
- [Linux amd64](https://github.com/the-homeless-god/wb/releases/download/v0.0.3/wb.linux-amd64.zip)
- [Linux arm64](https://github.com/the-homeless-god/wb/releases/download/v0.0.3/wb.linux-arm64.zip)

## Demo

<table>
  <tbody>
    <tr>
      <td>
        <video src="https://user-images.githubusercontent.com/5746414/213682926-f1cc2de7-a38c-4125-9257-92faecfc7e24.mp4">
      </td>
      <td>
        <video src="https://user-images.githubusercontent.com/5746414/213682913-398d3d11-1af8-4ae6-a0cd-a7f878efd88b.mp4">
      </td>
    </tr>
    <tr>
      <td colspan="2">
        <video src="https://user-images.githubusercontent.com/5746414/213682918-d6396a4f-ee23-431d-828e-4ad6a00e690e.mp4">
      </td>
    </tr>
  </tbody>
</table>

## Known issues

- Fullscreen mode not supported yet

## Comparisons

| Feature                 | Lynx                                          | Browsh                                       |
|-------------------------|-----------------------------------------------|----------------------------------------------|
| Type                    | Terminal web browser                          | Browser into a terminal                       |
| Age                     | Oldest and still maintained                   | Original "normal browser into a terminal"   |
| Layout Optimization     | Best layout when it understands a page         | Custom stylesheets to fix layout             |
| Web Standards Support   | Limited support for modern web standards      | Utilizes Firefox in headless mode            |
| JavaScript/WebAssembly  | Not supported                                 | Not supported                                |
| Media Support           | Cannot view or play media (audio, video, DOOM)| Depends on underlying browser (Firefox)     |
| Extension Support       | N/A                                           | Supports extensions (as of today)           |
| Resource Usage          | N/A                                           | Slower and requires more resources than `wb`|
| Development Ease        | N/A                                           | Easier to update underlying browser         |

## Operating System Support

As far as tested, the operating systems under are supported:

- Linux (Debian, Ubuntu and Arch tested)
- MacOS
- Windows 11 and WSL

## Contributing

Carbonyl is split in two parts: the "core" which is built into a shared library (`libcarbonyl`), and the "runtime" which dynamically loads the core (`carbonyl` executable).

The core is written in Rust and takes a few seconds to build from scratch. The runtime is a modified version of the Chromium headless shell and takes more than an hour to build from scratch.

If you're just making changes to the Rust code, build `libcarbonyl` and replace it in a release version of Carbonyl.

### Core

```console
$ cargo build
```

### Runtime

Few notes:

- Building the runtime is almost the same as building Chromium with extra steps to patch and bundle the Rust library. Scripts in the `scripts/` directory are simple wrappers around `gn`, `ninja`, etc..
- Building Chromium for arm64 on Linux requires an amd64 processor
- Carbonyl is only tested on Linux and macOS, other platforms likely require code changes to Chromium
- Chromium is huge and takes a long time to build, making your computer mostly unresponsive. An 8-core CPU such as an M1 Max or an i9 9900k with 10 Gbps fiber takes around ~1 hour to fetch and build. It requires around 100 GB of disk space.

#### Fetch

> Fetch Chromium's code.

```console
$ ./scripts/gclient.sh sync
```

#### Apply patches

> Any changes made to Chromium will be reverted, make sure to save any changes you made.

```console
$ ./scripts/patches.sh apply
```

#### Configure

```console
$ ./scripts/gn.sh args out/Default
```

> `Default` is the target name, you can use multiple ones and pick any name you'd like, i.e.:
>
> ```console
> $ ./scripts/gn.sh args out/release
> $ ./scripts/gn.sh args out/debug
> # or if you'd like to build a multi-platform image
> $ ./scripts/gn.sh args out/arm64
> $ ./scripts/gn.sh args out/amd64
> ```

When prompted, enter the following arguments:

```gn
import("//carbonyl/src/browser/args.gn")

# uncomment this to build for arm64
# target_cpu = "arm64"

# comment this to disable ccache
cc_wrapper = "env CCACHE_SLOPPINESS=time_macros ccache"

# comment this for a debug build
is_debug = false
symbol_level = 0
is_official_build = true
```

#### Build binaries

```console
$ ./scripts/build.sh Default
```

This should produce the following outputs:

- `out/Default/headless_shell`: browser binary
- `out/Default/icudtl.dat`
- `out/Default/libEGL.so`
- `out/Default/libGLESv2.so`
- `out/Default/v8_context_snapshot.bin`

#### Build Docker image

```console
# Build arm64 Docker image using binaries from the Default target
$ ./scripts/docker-build.sh Default arm64
# Build amd64 Docker image using binaries from the Default target
$ ./scripts/docker-build.sh Default amd64
```

#### Run

```
$ ./scripts/run.sh Default https://wikipedia.org
```
