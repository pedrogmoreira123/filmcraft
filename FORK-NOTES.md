# FORK-NOTES — Fi.V build of FilmCraft

Fork of [storytold/filmcraft](https://github.com/storytold/filmcraft) kept by Fi.V Connect for its
video pipeline. Modified build based on FilmCraft by the ArtCraft team; not an official ArtCraft
product. Branch `fiv/<upstream version>`, cut from the release tag that was tested
(`fiv/0.9.0` from `v0.9.0`). Releases: `fiv-<upstream version>-<n>` (workflow
`.github/workflows/fiv-release.yml`, started by pushing the tag `fiv-<version>-<n>`), with `SHA256SUMS.txt`.

## Build features

| Feature | Why |
|---|---|
| `whisper` (`filmcraft-cli`, `filmcraft`) | The official release ships without speech to text ("speech-to-text is not available in this build"). Our captions are made from the transcript. |
| `unbranded` (`filmcraft`) | `docs/brand/LICENSE-brand.txt`: a modified version must drop the ArtCraft marks. Leaves the wordmark out and says in plain text what the build is based on. |

```sh
cargo build --release -p filmcraft-cli -p filmcraft --features "filmcraft-cli/whisper filmcraft/whisper filmcraft/unbranded"
```

Speech model: `whisper-base` (`exec transcript.downloadModel model=whisper-base`, SHA-256 checked by
FilmCraft). Measured on a 58 s pt-BR sample (i5-10400F): base 9.6 s, small 30.1 s, both 3.8 % WER;
base is the default for narration.

## Changes

### 1. `transcript.removeFillers` takes `standalone` (and `pauseSeconds`)

- **Problem.** Portuguese fillers are also ordinary words. With
  `fillers: ["é","tipo","né","então","ahn","hum"]` the command cut the verb out of "Aqui é a
  Mariana" (4 removals in the test sample, one of them the verb).
- **Change.** `standalone: true` keeps only the fillers set off on both sides by punctuation or a
  pause of at least `pauseSeconds` (default 0.3). Default `false`: upstream behaviour unchanged.
  `crates/edit/src/transcript.rs` (`standalone_fillers`), `crates/engine/src/transcript.rs`.
- **Test.** `cargo test -p filmcraft-edit standalone_fillers_keep_ordinary_words` and
  `cargo test -p filmcraft-engine remove_fillers_standalone_keeps_portuguese_verbs`. By hand:
  `filmcraft-cli --project p.fcproj exec transcript.removeFillers '{"fillers":["é","tipo","né"],"standalone":true}'`.

### 2. `unbranded` feature

- **Problem.** The brand licence forbids the ArtCraft marks in modified builds.
- **Change.** `crates/ui-egui/src/brand.rs` returns no wordmark under the feature; the About box says
  "A modified build, based on FilmCraft by the ArtCraft team. Not an official ArtCraft product."
- **Test.** `cargo clippy -p filmcraft-ui-egui --features unbranded -- -D warnings`; open the app
  and check the Start screen and Help ▸ About show no wordmark.

### 3. CI: `fiv-release.yml`

Builds linux-x86_64 (ubuntu-22.04) and windows-x64 (windows-2022) with the features above, runs the
transcript tests and a smoke test (`transcript.generate` enabled, `removeFillers` lists
`standalone`), packages CLI + app + licences + this file, writes `SHA256SUMS.txt` and publishes
release `fiv-<version>-<n>`. Actions are pinned by commit SHA.

## Known gaps (not changed)

- `filmcraft-cli mcp` (headless) has no `--automation-read-root/--automation-write-root`, unlike the
  other Craft apps. Confine it by running it from a dedicated working folder.
- In bridge mode `render_frame` returns a screenshot of the window, not the Program frame. Use the
  CLI `render --seconds S --out f.png` on a saved project instead.
- Whisper writes numbers as digits ("às 11 da noite" for "onze"); fine for captions.
- Hesitations like "ahn" can come out as non-words ("EA-TN"); `removeFillers` will not match them.
  Edit the transcript with `transcript.set` when the narration is a recording.

## Upstream

Nothing here is pushed to `storytold/filmcraft` (the `upstream` remote has its push URL disabled).
