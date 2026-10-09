Opening a Paint.NET PDN3 document now produces editable bitmap layers instead of an unsupported-format error. Layer order, Unicode names, visibility, opacity and pixel alpha survive import; painting, masks, transforms, duplication, reordering and undo use PhotoCraft's existing layer commands. Saving an imported PDN suggests `.pcraft`, which preserves the layers and blend modes.

### Changes

- Add an independent, data-only NRBF reader and raw/gzip BGRA32 chunk decoder using the existing pure-Rust `flate2` dependency. Validate input lengths, references, dimensions, strides, chunk ranges and checksums; bound nesting, objects, strings, layers and decoded pixels; support cancellation.
- Support all 14 PDN3 blend modes, including older blend-operation objects and newer numeric records. Add CPU/GPU implementations and existing menu/control support for Reflect, Glow, Negation, XOR and the distinct Paint.NET Color Burn/Dodge variants. Existing Photoshop mode ordering and GPU IDs remain stable.
- Route desktop/browser opening, CLI batch conversion and platform file associations through the shared importer. Reject `.pdn` export explicitly, and warn when PSD export replaces a Paint.NET-only blend mode with Normal, including layer effects.
- Add synthetic, malformed-input and mutation tests, plus opt-in real-file import/edit/history/native-save checks. Document the measured compatibility and limits. Opt in to syncing @SilentYeti's public GitHub profile name for contributor credit.

### Validation

- Formatting, diff checks, dependency layering and touched-crate Clippy checks pass. Native desktop development/release builds pass.
- Nine PDN unit tests cover all modes, Unicode/hidden/opacity/alpha layers, padded rows, raw/gzip/reordered chunks, both mode layouts, cancellation, truncation, checksums and parser limits. Native round trips and PSD loss warnings are covered.
- Fourteen local Paint.NET 4.0.10 texture documents preserve layers exactly through `.pcraft`; composites differ from embedded full-size previews by at most 2.142/255. A supplied 1280×720, 11-layer 5.x document passes editing, painting, duplicate isolation, masks, transforms/reordering, undo/redo and native persistence. Its downsampled composite matches the embedded thumbnail with mean error 0.418/255 (maximum 8/255). No third-party documents or user assets are included in this PR.
- Broad native regression run: 2,799 passing tests across 91 completed suites. Two existing desktop socket tests hit sandbox `EPERM`; seven existing parallel GPU UI suites crashed in the software driver, then all passed serially (37 tests). Final I/O tests, CLI tests and the adversarial command panic test pass.
- WGSL validation and the actual half-float GPU XOR rounding regression pass on llvmpipe. The quick performance scenarios complete; hardware differs from the baseline and quick mode does not enforce budgets.
- Offscreen UI screenshots were inspected using an original synthetic four-layer PDN (Normal, Reflect, Glow, XOR). The screenshots show the workflow before opening and after native import, rather than a comparison with an older build. 

### Limits / checks for CI

Import only: PDN3 bitmap BGRA32 layers, up to 1,024 layers and 1 GiB of decoded pixel memory. Document metadata, print resolution and profiles are omitted; nonempty user metadata produces a warning. Unsupported layouts/versions return errors. Compatibility with every Paint.NET release is not established, and the supplied 5.x document has no matching full-size export oracle.

Wasm checking could not complete because the target standard library is absent. The official external corpus could not run because its missing fixtures require network access. Full GPU blend parity skips on the local adapter because it cannot render 32-bit float targets. CI or suitable hardware should run these checks before merge.

### Screenshots

Original synthetic fixture, captured with the new build. These show before opening the document and after native import; they are not an older-build comparison.

| Before opening | After import — editable layers |
| --- | --- |
| ![Before opening the synthetic PDN](https://raw.githubusercontent.com/SilentYeti/photocraft/c40a22fe5fb57ca606ca64219328c503a5ce6607/before-open.png) | ![Synthetic PDN imported as editable layers](https://raw.githubusercontent.com/SilentYeti/photocraft/c40a22fe5fb57ca606ca64219328c503a5ce6607/after-import.png) |

[Review screenshot attribution](https://github.com/SilentYeti/photocraft/blob/c40a22fe5fb57ca606ca64219328c503a5ce6607/ATTRIBUTION.md). Screenshot artifacts are on a separate fork branch and are not part of the application diff.
