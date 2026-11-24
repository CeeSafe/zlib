# zlib for CeeSafe

[zlib](https://zlib.net)'s one-shot compression and its CRC-32 and Adler-32 checksums, for
[CeeSafe](https://github.com/CeeSafe/CeeSafe) programs. It links the system's zlib (`links = ["z"]`); nothing of zlib
is copied here.

```toml
[dependencies]
zlib = "0.1"
```

```ceesafe
fn main() {
    let text = "hello hello hello hello".bytes();
    let packed = zlib.Zlib.compress(text)!;
    let back = zlib.Zlib.uncompress(packed.view(), text.length)!;
    Console.writeLine(zlib.Zlib.crc32(back.view()));
}
```

## The API

| Function | Result |
|---|---|
| `Zlib.compress(data: Slice<u8>)` | `Result<Vector<u8>, ZlibError>`: `data` in zlib's format, at the default level |
| `Zlib.compressLevel(data: Slice<u8>, level: i32)` | the same at `level`: 0 stores, 1 is fastest, 9 smallest, -1 the default |
| `Zlib.uncompress(packed: Slice<u8>, size: usize)` | `Result<Vector<u8>, ZlibError>`: `packed` uncompressed into at most `size` bytes, the size it had (zlib's format does not record it) |
| `Zlib.crc32(data)`, `Zlib.crc32Update(crc, data)` | `u32`: the CRC-32 of gzip, PNG and zip, of `data` or of what gave `crc` followed by `data` |
| `Zlib.adler32(data)`, `Zlib.adler32Update(adler, data)` | `u32`: the Adler-32 zlib's format uses |
| `Zlib.version()` | `StringView`: the linked zlib's version, such as `1.3` |

`ZlibError` is `OutOfMemory`, `BufferTooSmall` (more data than `size`), `CorruptData`, `InvalidLevel`, `TooLarge` (a
buffer longer than the target's `uLong` counts: 4 GiB on Windows) or `Failed`.

The raw binding is `src/bind.cee`, written by `ceesafec cbind`. zlib's sizes are `uLong`, C's `unsigned long`, which
CeeSafe spells `CULong`: 32 bits on Windows and 64 elsewhere. Streaming (`z_stream`, `deflate`, `inflate`) is not in
this version.

## Building

The C compiler needs zlib's header and library: on Debian and Ubuntu `apt install zlib1g-dev`, on Fedora
`dnf install zlib-devel`; macOS has them. `ceesafec test` runs the tests, whose expected checksums are Python's
`zlib.crc32` and `zlib.adler32` of the same bytes; `ceesafec test --sweep alloc` fails each allocation in turn.

Tested with ceesafec 0.95.2 and zlib 1.2.12 (macOS, Apple clang) and 0.94.5 and zlib 1.3 (Ubuntu 24.04, gcc and
clang).

`src/bind.cee` was written with:

```sh
ceesafec cbind --header zlib.h -I <zlib's include directory> --cc clang \
  --only zlibVersion,compress,compress2,compressBound,uncompress,crc32,adler32,Z_OK,Z_STREAM_ERROR,Z_DATA_ERROR,Z_MEM_ERROR,Z_BUF_ERROR,Z_VERSION_ERROR,Z_NO_COMPRESSION,Z_BEST_SPEED,Z_BEST_COMPRESSION,Z_DEFAULT_COMPRESSION \
  -o src/bind.cee
```

A `cbind` before CeeSafe's ADR 0221 writes `usize` for `uLong`, which is wrong on Windows.

## Licence

MIT (`LICENSE`). zlib itself is under the [zlib licence](https://zlib.net/zlib_license.html) and is not part of
this package.
