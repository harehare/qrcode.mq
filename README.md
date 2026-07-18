<h1 align="center">qrcode.mq</h1>

A [QR Code](https://www.qrcode.com/en/about/standards.html) (ISO/IEC 18004) encoder implemented as an [mq](https://github.com/harehare/mq) module. Generates real, scannable QR codes from text or URLs and renders them as ASCII art or SVG.

## Features

- Byte-mode encoding (any UTF-8 text, including URLs)
- Error correction levels `L`, `M`, `Q`, `H`
- Automatic version selection (1-6) based on input length and error correction level
- Automatic mask-pattern selection using the standard penalty-scoring rules
- Reed-Solomon error correction with correct block splitting/interleaving for multi-block versions
- ASCII-art and SVG rendering, both with a customizable quiet zone

### Limits

Only versions 1-6 are supported (up to 134 bytes at level `L`, 60 bytes at level `H`). Larger inputs raise an error rather than silently truncating.

## Installation

Copy `qrcode.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp qrcode.mq ~/.local/mq/config/
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "qrcode" | qrcode::qr_generate(.)' <<< "https://github.com/harehare/mq"
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "qrcode" | qrcode::qr_generate(.)' <<< "https://github.com/harehare/mq"
```

Or via [HTTP Import](https://github.com/harehare/mq/blob/main/docs/books/src/reference/modules_and_imports.md#http-imports), no local installation required:

```sh
mq -I raw 'import "github.com/harehare/qrcode.mq" | qrcode::qr_generate(.)' <<< "https://github.com/harehare/mq"
```

## API

### `qr_generate(text, ecc="M", dark="##", light="  ")`

Generates an ASCII-art QR code for `text` and returns it as a multi-line string, including a 4-module quiet-zone border.

| Argument | Type | Description |
|---|---|---|
| `text` | String | The text or URL to encode |
| `ecc` | String | Error correction level: `"L"`, `"M"` (default), `"Q"`, or `"H"` |
| `dark` | String | Characters used for a dark module (default `"##"`) |
| `light` | String | Characters used for a light module (default `"  "`) |

`dark`/`light` default to two characters wide so modules render roughly square in a terminal.

### `qr_matrix(text, ecc="M")`

Encodes `text` and returns a dict describing the raw symbol, for callers that want custom rendering:

```
{"version": Number, "ecc": String, "mask": Number, "size": Number, "modules": Array}
```

`modules` is a `size` x `size` array of arrays of `0`/`1` (`1` = dark module).

### `qr_svg(text, ecc="M", module_size=4, dark="#000000", light="#ffffff")`

Generates an SVG QR code for `text` and returns it as an SVG document string, including a 4-module quiet-zone border.

| Argument | Type | Description |
|---|---|---|
| `text` | String | The text or URL to encode |
| `ecc` | String | Error correction level: `"L"`, `"M"` (default), `"Q"`, or `"H"` |
| `module_size` | Number | Pixel size of one QR module (default `4`) |
| `dark` | String | Fill color for dark modules (default `"#000000"`) |
| `light` | String | Fill color for light modules (default `"#ffffff"`) |

```sh
mq -L . -I raw 'import "qrcode" | qrcode::qr_svg(.)' <<< "https://github.com/harehare/mq" > qrcode.svg
```

All three functions raise an error if `ecc` is not one of `L`/`M`/`Q`/`H`, or if `text` is too long to fit in version 6 at the requested error correction level.

## Example

```sh
mq -L . -I raw 'import "qrcode" | qrcode::qr_generate(.)' <<< "https://github.com/harehare/mq"
```

```
                                                                          
                                                                          
                                                                          
                                                                          
        ##############        ##    ##      ####    ##############        
        ##          ##    ##  ##        ######      ##          ##        
        ##  ######  ##      ######  ####  ##  ##    ##  ######  ##        
        ##  ######  ##            ##    ##  ######  ##  ######  ##        
        ##  ######  ##        ####        ####      ##  ######  ##        
        ##          ##  ##    ##      ####    ##    ##          ##        
        ##############  ##  ##  ##  ##  ##  ##  ##  ##############        
                          ##########  ######                              
        ##    ##  ####  ##  ####    ##  ##            ##  ##    ##        
        ########                  ##  ######    ##  ##    ##    ##        
          ##  ####  ######    ##########    ########    ########          
              ##  ##  ####  ########    ##  ######      ##  ####          
          ##  ##  ######    ####    ####  ##  ########    ##  ####        
                  ##    ####  ##  ##################                      
            ######  ####      ##  ####      ##  ##  ##############        
          ##  ####            ##      ##  ##    ######    ##  ##          
        ####  ####  ##  ##    ######      ##    ##    ##      ##          
          ########    ##  ##    ##########      ########  ##    ##        
        ##  ####    ##  ##    ####  ######        ##  ####    ####        
                ####        ####    ##    ####  ####  ####    ####        
        ##  ##      ##  ##    ##      ####  ##############  ##            
                        ####  ##    ####      ####      ##  ######        
        ##############    ##  ####        ##    ##  ##  ##    ##          
        ##          ##  ####  ##    ##      ##  ##      ######  ##        
        ##  ######  ##    ##  ####    ######    ##########    ##          
        ##  ######  ##    ####  ####  ####    ####  ##########  ##        
        ##  ######  ##            ####    ##  ####      ######  ##        
        ##          ##        ##      ##  ##########    ##    ##          
        ##############    ##    ##      ####    ############  ##          
                                                                          
                                                                          
                                                                          
                                                                          
```

Lower the error correction level or shorten the text if you need a smaller/simpler code:

```sh
mq -L . -I raw 'import "qrcode" | qrcode::qr_generate(., "L")' <<< "hi"
```

## Verification

There is no QR *decoder* in this module, so correctness was verified externally: outputs were cross-checked against the Python [`qrcode`](https://pypi.org/project/qrcode/) and [`reedsolo`](https://pypi.org/project/reedsolo/) libraries, and decoded end-to-end with an independent QR decoder across versions 1-6, all four error correction levels, multi-block Reed-Solomon layouts, and multi-byte UTF-8 input.

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.6 or later.

## License

MIT
