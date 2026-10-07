# Vendored NVENC headers (pinned)

These are the FFmpeg `nv-codec-headers` at tag **`n11.1.5.4`**, i.e. **NVENC API 11.1**.

They are vendored on purpose: this plugin targets Kepler and first-generation Maxwell
(GM10x, e.g. GTX 750 / 750 Ti) NVENC, whose hardware only answers NVENC API **≤ 11.1**.
The NVENC API version is baked into the plugin and the `nvenc-kepmax-test` helper at
compile time (`NVENCAPI_VERSION`), so building against newer system headers (e.g. the
13.1 headers shipped by recent distros) makes the driver reject the encode session and
the encoders silently never register.

Source: https://github.com/FFmpeg/nv-codec-headers (tag `n11.1.5.4`).
Upstream commit: see `git log` of that repository at the tag.

To use your distribution's headers instead, configure with
`-DFFnvcodec_USE_BUNDLED=OFF`.
