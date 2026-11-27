# png-squeeze-01 — lossless smallest-PNG re-encoder

A pure Node.js (standard library only, no dependencies) re-encoder that takes a
1280×720 RGBA screenshot and writes the smallest possible lossless PNG for it.

## How it works

1. **Decode** the input PNG (own decoder: stored/filtered IDAT, zlib inflate).
2. **Analyse** the pixels: opaque vs alpha, greyscale vs colour, colour count,
   and whether a single transparent colour can be keyed out.
3. **Reduce the colour type** losslessly where possible:
   - ≤ 256 colours → palette (colour type 3), palette ordered by frequency,
     with a `tRNS` chunk when any colour is transparent.
   - Greyscale → colour type 0 (or 4 with alpha).
   - Otherwise RGB (type 2) or RGBA (type 6), with a `tRNS` key when the
     image is opaque except for one fully-transparent colour.
4. **Choose filters per scanline**: fixed filters (None/Sub/Up/Paeth),
   minimum-sum-of-absolute-differences, pairwise, and a brute-force
   per-row search with a 32 KiB dictionary — each candidate is scored by
   compressing it with zlib level 6 and keeping the smallest total.
5. **Final deflate** at zlib level 9, trying both the default and the
   `Z_FILTERED` strategy and keeping the smaller stream.

## Determinism

No clock reads, no randomness, fixed tie-breaks everywhere. Two runs on the
same input produce byte-identical output.

## Performance

The whole 8-image practice set re-encodes in ~13 s on one CPU (budget: 60 s),
with output sizes well under the 4 MiB cap per image.

## Running

The grader runs `node /bot/main.ts`; it reads `/input/<id>/image.png` and
writes `/output/<id>/image.png`, creating the output directory itself.
Locally, `MAKEFUN_INPUT_DIR` and `MAKEFUN_OUTPUT_DIR` stand in for those paths.

## License

MIT. See `LICENSE`.
