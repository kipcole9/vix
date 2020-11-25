# Vix

Vix is Elixir extension for [vips](https://libvips.github.io/libvips/).

Vix is a **NIF bindings** for libvips. It uses dirty IO scheduler to avoid blocking VM. Operation binding are  auto generated using GObject introspection, so documentation and bindings are up-to-date with the vips version installed.

### What is Vips

*(from vips documentation)*

> libvips is a [demand-driven, horizontally threaded](https://github.com/libvips/libvips/wiki/Why-is-libvips-quick) image processing library. Compared to similar libraries, [libvips runs quickly and uses little memory](https://github.com/libvips/libvips/wiki/Speed-and-memory-use).

See [libvips documentation](https://libvips.github.io/libvips/API/current/How-it-works.md.html) for more details.

### Example

```elixir
alias Vix.Vips.Image
alias Vix.Vips.Operation

def example(path) do
  {:ok, im} = Image.new_from_file(path)

  # put im at position (100, 100) in a 3000 x 3000 pixel image,
  # make the other pixels in the image by mirroring im up / down /
  # left / right, see
  # https://libvips.github.io/libvips/API/current/libvips-conversion.html#vips-embed
  im = Operation.embed(im, 100, 100, 3000, 3000, extend: :VIPS_EXTEND_MIRROR)

  # multiply the green (middle) band by 2, leave the other two alone
  im = Operation.linear(im, [1, 2, 1], [0])

  # make an image from an array constant, convolve with it
  {:ok, mask} =
    Image.new_matrix_from_array(3, 3,
      [
        [-1, -1, -1],
        [-1, 16, -1],
        [-1, -1, -1]
      ],
      scale: 8
    )

  im = Operation.conv(im, mask, precision: :VIPS_PRECISION_INTEGER)

  # finally, write the result back to a file on disk
  :ok = Vix.Vips.Image.write_to_file(im, "out.jpg")
end
```

The [libvips reference manual](https://libvips.github.io/libvips/API/current/) has a complete explanation of every method

### Simple *unscientific* comparison

Generating thumbnail for a sample image

|   | Vix       | Mogrify   |
|---|-----------|-----------|
| 1 | 298.731ms | 618.854ms |
| 2 | 29.873ms  | 605.824ms |
| 3 | 34.479ms  | 609.820ms |
| 4 | 31.339ms  | 604.712ms |
| 5 | 32.526ms  | 605.553ms |

Notice that the gain is significant for the subsequent repetition of the operation for Vix since vips  [caches the operations](https://libvips.github.io/libvips/API/current/VipsOperation.html).

### Warning

This library is experimental, untested, and unstable. Interface might change significantly in the future versions. The code is not well tested or optimized, so you might experience crashes.

### Requirements

* libvips
* pkg-config

### Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `vix` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:vix, "~> x.x.x"}
  ]
end
```

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at [https://hexdocs.pm/vix](https://hexdocs.pm/vix).

### TODO
- [ ] support `VipsConnection`
- [ ] move GObject-introspection to a separate library
