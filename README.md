# NVENC Kepler/Maxwell Plugin

> [!WARNING]
> **Vibe-coded personal fork.** AI-written, unreviewed, tested on exactly one machine — mine. I can't promise support for this plugin in case it croaks or smthn.
>
> Really made specifically for the **GTX 750 Ti**. NVIDIA killed support for Kepler's NVENC block, and GM10x (750 / 750 Ti) shares it. They killed support for a perfectly fine piece of hardware and I'm pissed when it got dropped.
>
> Real work is upstream: [`RanAwaySuccessfully/nvenckepler`](https://github.com/RanAwaySuccessfully/nvenckepler).

## Introduction

This is pretty much a copy-paste of the existing [obs-nvenc plugin](https://github.com/obsproject/obs-studio/tree/master/plugins/obs-nvenc) that comes bundled with OBS Studio adapted to still be compatible with Kepler and first-generation Maxwell (GM10x, e.g. GTX 750 / 750 Ti) cards. This required a downgrade to remove features introduced after NVIDIA Video Codec SDK v11.1 such as AV1 support and split encoding removed (although they have been removed quite hastily and as such, some non-functional leftovers may still be found here and there).

If you want to use NVENC on OBS Studio 31 and above on an older Nvidia graphics card, this plugin is for you.

This plugin has been adapted to use the [official plugin template](https://github.com/obsproject/obs-plugintemplate), but it is very much a non-official plugin.

**NOTE:** NVENC detection works in this fork. The plugin ships a small helper binary (`nvenc-kepmax-test`) that probes the GPU's encoders at load time, so the NVENC encoders only show up when your hardware actually supports them (H.264 on Kepler and GM10x Maxwell, H.264 + HEVC on GM20x Maxwell and newer).


## How to install

Grab the **most recent release from the [Releases section](https://github.com/Octavity/nvenc-kepmax/releases)** and download the one appropriate for your platform.

**You must build against the old NVENC headers (this fork vendors them for you). The NVIDIA
driver version does not matter as much as the NVENC API version the plugin was compiled
against — see the build notes below.**

### How to build manually

#### Linux (x86_64)

Install the build prerequisites for your distribution. You need `libobs` headers, CMake,
Ninja and a C/C++ compiler. **You do not need a system `ffnvcodec`/NVENC headers package** —
the correct (old) NVENC headers are vendored in `deps/nv-codec-headers`:

```sh
# Arch / CachyOS / Manjaro
sudo pacman -S base-devel cmake ninja obs-studio

# Debian / Ubuntu (libobs-dev comes from the OBS PPA)
sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt update
sudo apt install build-essential cmake ninja-build libobs-dev
```

> **Why the headers are vendored:** Kepler and GM10x Maxwell GPUs only answer **NVENC API
> ≤ 11.1**, and the NVENC API version is baked in at compile time. Building against newer
> system headers (e.g. the 12.x/13.x packages shipped by modern distros) makes the driver
> reject the encode session, and the encoders silently never appear. This fork therefore
> always builds against the pinned API 11.1 headers in `deps/nv-codec-headers` by default.

If you *want* to use the system headers instead, configure with `-DFFnvcodec_USE_BUNDLED=OFF`
(or point at a specific copy with `-DFFnvcodec_INCLUDE_DIR=/path/to/include`).

Then configure, build and install with the `linux-x86_64` preset:

```sh
cmake --preset linux-x86_64
cmake --build --preset linux-x86_64
```

When it finishes, the plugin is staged inside the build directory:

```
build_linux/rundir/RelWithDebInfo/nvenc-kepmax.so
build_linux/rundir/RelWithDebInfo/nvenc-kepmax/locale/*.ini
```

**Install for your user only (no `sudo`, recommended):**

```sh
mkdir -p ~/.config/obs-studio/plugins/nvenc-kepmax/bin/64bit
cp build_linux/rundir/RelWithDebInfo/nvenc-kepmax.so ~/.config/obs-studio/plugins/nvenc-kepmax/bin/64bit/
cp -r build_linux/rundir/RelWithDebInfo/nvenc-kepmax ~/.config/obs-studio/plugins/nvenc-kepmax/data
# the NVENC detection helper must live in the plugin's data folder as well:
cp build_linux/src/tester/nvenc-kepmax-test ~/.config/obs-studio/plugins/nvenc-kepmax/data/
```

**Install system-wide (needs `sudo`):**

```sh
sudo cmake --install build_linux --prefix /usr
```

This places `nvenc-kepmax.so` in the OBS plugin directory (for example `/usr/lib/obs-plugins/`
on Arch, or `/usr/lib/x86_64-linux-gnu/obs-plugins/` on Debian) and the locale files in
`/usr/share/obs/obs-plugins/nvenc-kepmax/`.

This fork registers its own encoder IDs (`nvenc_kepmax_*`), so it **coexists with the NVENC
encoders that ship with OBS** — you do *not* need to remove or disable the bundled `obs-nvenc`
plugin. In the encoder list the fork's encoders appear with a `(Kepler/Maxwell Plugin)` suffix,
e.g. `NVIDIA NVENC H.264 (Kepler/Maxwell Plugin)`. (As noted above, they only show up when the
`nvenc-kepmax-test` helper reports that your GPU and driver actually support them.)

**Flatpak / Snap OBS:** the plugin cannot be loaded into the sandboxed build of OBS. Use a
distribution-packaged (native) OBS instead.

#### Windows / macOS

Change the preset to `windows-x64` when making a Windows build. Builds for macOS are not
currently supported by this fork (they can be attempted with the `macos` preset, but expect to
tinker with the CMake settings until it works).

```sh
cmake --preset windows-x64
cmake --build --preset windows-x64
```

Once the commands finish, the resulting `nvenc-kepmax.dll` (or `.so` for Linux) can be found in
the `rundir\RelWithDebInfo` folder inside the build directory.

## Why

Mostly because I had a spare Kepler card I kept as a secondary/tertiary graphics card and while using it is more of a novelty than anything else, I still want NVENC to be available with it in the future. Providing support for these cards is unlikely to get any easier with time, and I don't plan on doing anything other than the extreme basics with this plugin (pull requests are more than welcome though).

Do note that this only works while the NVIDIA driver still ships an NVENC library that speaks
NVENC API 11.1 for your GPU (`libnvidia-encode.so`, which current drivers still do for
Kepler/GM10x). If NVIDIA ever drops that entirely, no plugin can bring it back.

If Nouveau had video encoding support via VA-API I wouldn't even have bothered with this plugin, but they only have decoding support, and only up until Kepler. NVENC is listed on a TODO list with a "Hard" difficulty attached to it, so I imagine this is not going to be ready anytime soon.
