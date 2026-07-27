# Code Review Report 02 (Kimi K3)

**Scope:** Whole-repo review of the fork at `main` (`7773fc8`) — shared plumbing (`cli/`, `callbacks/`, `utils/`, `release/`), shared model code (`models/common/`), packaging/CI/docs, plus targeted spot-checks of model families (diff-based drift checks, selected CLI entrypoints and schedulers).
**Date:** 2026-07-27

## Coverage and method (read this before the findings)

Read in full: all of `release/`, `cli/parser/`, `cli/defaults/`, `callbacks/` (all 12 files), 10 of 15 `utils/` modules, `common/schedulers/` (3 of 4), `common/vae/`, `common/weights/loading/` (loader + applier), `common/resolution/config_resolution.py`, `common/config/model_config.py` (partial), `pyproject.toml`, `README.md`, `CHANGELOG.md` (head), `Makefile`, `.github/workflows/`.

Mechanically verified: all 33 `[project.scripts]` entrypoints resolve to real `main()` functions; all README link targets (12 model READMEs + assets) exist; all `AGENTS.md`-referenced Makefile targets exist; CHANGELOG latest release (`0.18.0`) matches `pyproject.toml` version; zero `TODO`/`FIXME` in `src/`; krea2↔flux and z-image↔z-image-turbo diffs inspected.

**Not deeply reviewed** (coverage limit, stated for candor): per-model transformer/VAE/text-encoder implementations, `common/training/`, `common/tokenizer/`, `lora_loader.py` internals, the shell-completions generator, and test-file contents beyond structure/markers. Findings below are therefore concentrated in shared code — which is also where the blast radius is largest.

## Summary

The codebase is in good shape: consistent architecture across 13 model families, disciplined CLI construction via `CommandLineParser`, clean scheduler math (Euler/flow-match/SeedVR2 derivations all check out), and no lint-level rot (no TODOs, no dead exception swallows outside metadata paths). The serious issues cluster in two places: **the release pipeline** (the PyPI package-name mismatch from REPORT-01 is only half-fixed, and upload failures are unconditionally swallowed) and **silent-failure patterns** (weight loading with discarded `strict=False` mismatches, exception-swallowing save/metadata paths, falsy-`0.0` metadata merges). One significant test gap: the FLUX.1 family — the largest in the repo — has no generation tests.

## Findings

### 1. Release pipeline still targets upstream's PyPI package name (High)

**Files:** `pyproject.toml:16`, `src/mflux/release/release_manager.py:17`, `src/mflux/release/pypi_publisher.py:36-60`

REPORT-01 finding #1 was only partially fixed. `package_name` now defaults to `"mflux2"`, but that value is used **only** for the existence check:

```python
url = f"{repo_url}/{package_name}/json"   # queries pypi.org/pypi/mflux2/json
```

`mflux2` does not exist on PyPI → 404 → "this is normal for new packages" → publish proceeds. But `uv build` names the distribution from `pyproject.toml`'s `[project] name = "mflux"`, and `twine upload` uploads whatever is in `dist/` — so the fork's release workflow will attempt to publish **to the canonical upstream `mflux` PyPI project** using the fork's `PYPI_API_TOKEN` (`.github/workflows/release.yml` provides exactly that secret). Expected outcome is a 403 (fine, except see finding #2: it's swallowed and the release "succeeds" without publishing); the guard that was supposed to prevent this never engages.

**Suggested fix:** rename `[project] name` in `pyproject.toml` (e.g. `mflux2`), or gate PyPI publishing off for this fork; the existence-check name and the built distribution name must be the same string, ideally derived from one source.

### 2. PyPI upload failures are always swallowed; `optional=False` is a dead parameter (Medium)

**File:** `src/mflux/release/pypi_publisher.py:87, 90-165`

`publish_to_pypi()` passes `optional=False`, but `_upload_to_pypi` never reads the `optional` parameter — every failure path (`TwineException`, `OSError`, `ValueError`, `RuntimeError`) ends in `print(... "failures are non-critical, continuing with release process"); return`. So even with finding #1 fixed, a failed/partial upload lets the release continue to git-tagging and GitHub-release creation, producing a "successful" release that was never published — exactly the state `_should_publish_to_pypi`'s re-run guard then refuses to retry (tag + release now exist).

**Suggested fix:** honor the `optional` flag: re-raise when `optional=False` (or delete the parameter and the dead `publish_to_pypi` wrapper intent).

### 3. All weight application discards `strict=False` mismatches (Medium)

**File:** `src/mflux/models/common/weights/loading/weight_applier.py:39-47, 90`

Every `model.update(component_weights, strict=False)` ignores `Module.update`'s return value (the unused/unmatched keys). A typo or drift in any family's `*_weight_mapping.py` therefore fails silently — the affected layers keep random-init weights and the model still runs, producing subtly degraded output. This sits under all 13 model families.

**Suggested fix:** capture the return of `model.update(...)` and `logger.warning` any unmatched keys (keep the non-strict behavior for intentional partial loads such as LoRA).

### 4. `VersionUtil` can stamp a foreign project's version into image metadata (Medium)

**File:** `src/mflux/utils/version_util.py:15-25`

`_scan_pyproject()` walks up from `mflux/utils/` and returns the version from the **first** `pyproject.toml` it finds. For a normally-installed (non-editable) mflux — e.g. `uv tool install` or a venv inside an unrelated project — the first `pyproject.toml` found upward can be the *user's own project's*, so `get_mflux_version()` returns that project's version and writes it into every generated image's `mflux_version` metadata (and, worse, `_validate_changelog_entry` in a release run compares it against CHANGELOG). Editable installs mask this in dev.

**Suggested fix:** prefer `importlib.metadata.version("mflux")` first, or verify the found `pyproject.toml`'s `[project] name == "mflux"` before trusting its version.

### 5. Third-party model configs silently lose sigma/KV-cache/training overrides (Medium)

**Files:** `src/mflux/models/common/resolution/config_resolution.py:113-128`, `src/mflux/models/common/config/model_config.py:442, 513-515, 528-530, 627-647`

`_create_config()` clones a base `ModelConfig` for HuggingFace-repo models (explicit `--base-model` or substring inference) but copies only a subset of fields. Dropped: `sigma_base_shift`, `sigma_max_shift`, `sigma_base_seq_len`, `sigma_max_seq_len`, `sigma_shift_terminal`, `text_encoder_overrides`, `lora_training_steps/guidance`, `supports_kv_cache`. Bases with non-default values — qwen-image (`0.9`/`8192`/`0.02`), ernie-image (`1.3863`), flux2-klein-9b-kv (`supports_kv_cache=True`) — revert to FLUX.1-era defaults for any community fine-tune resolved through them, which shifts the scheduler's sigma schedule (wrong denoising trajectory) and disables KV caching.

**Suggested fix:** copy all constructor fields in `_create_config` (or construct via `dataclasses.replace`-style cloning with only `model_name`/`base_model` overridden).

### 6. `--config-from-metadata` drops explicit `0.0` values (Medium)

**File:** `src/mflux/cli/parser/parsers.py:320, 357, 363-365`

Metadata merge uses truthiness: `if namespace.guidance == guidance_default and guidance_from_metadata:` (same pattern for `image_strength`, `controlnet_strength`, `controlnet_save_canny`). A stored `0.0` — documented as meaningful for `--image-strength` ("A value of 0.0 means no influence") and valid as guidance for distilled models — is falsy and silently ignored. Existing tests cover 4.2/5.0 but not 0.0.

**Suggested fix:** use `is not None` checks instead of truthiness for these four merges.

### 7. `--mlx-cache-limit-gb` silently enables tiled VAE encode/decode (Medium)

**Files:** `src/mflux/callbacks/instances/memory_saver.py:25-27`, `src/mflux/models/common/vae/tiling_config.py:8-10`, `src/mflux/models/common/vae/vae_util.py:16, 45-49`

Most families initialize `model.tiling_config = None` (flux, flux2, qwen, fibo, krea2, z_image, ideogram4 — confirmed via grep). When any cache limit is set (including a bare `--mlx-cache-limit-gb 8` without `--low-ram`), `MemorySaver.__init__` assigns a default `TilingConfig()`, whose defaults turn on **tiled VAE encode** (`vae_encode_tiled=True`) and **8×8 tiled decode**. A flag documented as "limit MLX cache size" thus changes the numerics (tile-boundary blending) and performance of every generation, with no log line saying so.

**Suggested fix:** decouple — don't touch `model.tiling_config` in `MemorySaver`; make tiling an explicit CLI/model option, or at minimum log when it is force-enabled.

### 8. FLUX.1 family has no generation tests (Medium)

**Directory:** `tests/image_generation/`

Golden/generation tests exist for qwen (+edit), krea2, fibo (+edit), flux2-klein (+edit), z_image, ernie, ideogram4 (+golden, +checkpoint-layout), seedvr2 — but nothing references `Flux1`, `flux_generate`, or `FluxInitializer` anywhere under `tests/`. The largest family in the repo (126 files, ~7.4k lines: txt2img, fill, depth, redux, kontext, controlnet×3, concept×2, in-context×3) is covered only incidentally via parser/scheduler unit tests. A regression in any flux variant would not be caught by `make test`.

**Suggested fix:** port the upstream flux golden tests or add minimal slow-marked generation tests per variant.

### 9. `ImageUtil.save_image` swallows the actual save failure (Medium)

**File:** `src/mflux/utils/image_util.py:239-257`

The `try` wraps `image.save(file_path)` itself, and `except Exception` only logs. On disk-full/permission/codec errors the CLI exits 0 having written nothing (or having written a partial file). The `# noqa: BLE001` shows this was a deliberate choice, but it converts I/O failure into silent success at the exact moment the user cares about most.

**Suggested fix:** let exceptions from `image.save` propagate (keep the swallow only around the best-effort metadata-embed calls).

### 10. `z-image-turbo` CLI accepts `--model` but silently ignores predefined names (Medium)

**File:** `src/mflux/models/z_image/cli/z_image_turbo_generate.py:15, 23-24` (contrast `z_image_generate.py`)

The turbo CLI adds `add_model_arguments(require_model_arg=False)` yet constructs `ZImage(model_config=ModelConfig.z_image_turbo(), ...)` unconditionally. `--model z-image` parses fine, resolves `model_path=None` (it's a predefined name), and is silently discarded — the user gets turbo output with no warning. The sibling non-turbo CLI respects the choice via `ModelConfig.from_name(args.model or "z-image")`.

**Suggested fix:** resolve via `ModelConfig.from_name(args.model or "z-image-turbo")`, or drop `--model` from this parser.

### 11. `redux_image_paths` stringified as a Python list literal in metadata (Low)

**File:** `src/mflux/utils/generated_image.py:233` (contrast `:227`)

`"redux_image_paths": str(self.redux_image_paths)` produces `"['/a.png', '/b.png']"` — a stringified list — while `image_paths` on the line above is a proper JSON array. Breaks schema consistency for `--config-from-metadata` round-trips and any downstream JSON consumer.

**Suggested fix:** `[str(p) for p in self.redux_image_paths] if self.redux_image_paths else None`.

### 12. XMP embedding always fails for prompt-less runs (SeedVR2) (Low)

**File:** `src/mflux/utils/metadata_builder.py:74, 68-69`

`metadata.get("prompt", "")` returns `None` when the key exists with value `None` (the SeedVR2 case — `_get_metadata` sets `"prompt": self.prompt` unconditionally), so `.replace(...)` raises `AttributeError`, caught by the blanket `except` → the whole XMP/IPTC embed is skipped with only a log line. Every SeedVR2 upscale silently gets no XMP/IPTC metadata.

**Suggested fix:** `(metadata.get("prompt") or "")` before escaping.

### 13. `get_right_half` drops metadata fields (Low)

**File:** `src/mflux/utils/generated_image.py:77-99`

The copy constructor omits `image_paths`, `redux_image_paths`, `redux_image_strengths`, and `negative_prompt`, so in-context/edit outputs saved via the right-half path lose that metadata.

**Suggested fix:** pass the four missing fields through.

### 14. `torch.load(..., weights_only=False)` on downloaded checkpoint (Low)

**File:** `src/mflux/models/common/weights/loading/weight_loader.py:214`

DepthPro weights load via `torch.load` with `weights_only=False` — full pickle execution on a network-fetched file. The source is Apple's CDN (trusted), but there's no checksum/signature verification in `_download_from_url`, so a compromised or hijacked download is code execution.

**Suggested fix:** if the checkpoint format allows, use `weights_only=True`; otherwise pin and verify a SHA-256 of the known file.

### 15. GitHub API calls have no timeout (Low)

**File:** `src/mflux/release/github_api.py:20, 60`

`requests.get/post` without `timeout` in CI (`release.yml`) can hang the job indefinitely (contrast `pypi_publisher.py:44`, which sets `timeout=30`).

**Suggested fix:** add `timeout=30` to both calls.

### 16. CLI help-text bugs (Low)

**File:** `src/mflux/cli/parser/parsers.py:109, 120, 383`

- `--width` help: `f"Default is {ui_defaults.HEIGHT}"` — should be `WIDTH` (both are 1024 today, so invisible until the defaults diverge).
- `--auto-seeds` help says "random ints between 0 and 1 billion", code uses `max_seed_value = int(1e7)` (10 million).

**Suggested fix:** correct both strings to match the code.

### 17. Dead code and confusing API shapes (Low)

- `add_batch_image_generator_arguments()` (`parsers.py:136-139`) has zero callers in `src/` — dead (and if ever used alone, `parse_args` would hit `namespace.auto_seeds`/`seed` AttributeErrors, since it adds neither flag while setting `supports_image_generation=True`).
- `SeedVR2EulerScheduler.cfg_scale` (`seedvr2_euler_scheduler.py:16`) is assigned, never read.
- `FlowMatchEulerDiscreteScheduler`: `get_timesteps_and_sigmas()` returns `(timesteps, sigmas)` while `_compute_timesteps_and_sigmas()` returns `(sigmas, timesteps)` — opposite orders under near-identical names (`flow_match_euler_discrete_scheduler.py:26, 37, 63-76, 104-120`). Currently unpacked correctly at both call sites; a trap for the next caller.
- `TilingConfig.vae_decode_tiles_per_dim` is used only as a boolean (`> 1`); actual tiling derives from a hardcoded `(512, 512)` tile size in `vae_util.py:59` — the field name promises behavior it doesn't control.

**Suggested fix:** delete or wire up the batch-args method; delete `cfg_scale`; rename the scheduler helpers to make return order explicit (e.g. `..._sigmas_then_timesteps`); align the tiling field with the implementation.

### 18. IPTC stored as a hex string in a PNG tEXt chunk (Low)

**File:** `src/mflux/utils/metadata_builder.py:59`

`pnginfo.add_text("IPTC", iptc_binary.hex())` writes a chunk literally named `IPTC` containing hex text. The de-facto standard for IPTC-in-PNG is Photoshop's `Raw profile type iptc` chunk; as written, common tools (ExifTool, Preview) won't surface this metadata, defeating the "maximum compatibility" intent of `save_image`.

**Suggested fix:** write the standard `Raw profile type iptc` text chunk format, or verify against ExifTool and document what reader is targeted.

### 19. README installs upstream, not this fork (Low — possibly intentional)

**File:** `README.md:33, 68`

`uv tool install --upgrade mflux` and the Python-API example's `dependencies = ["mflux"]` both pull the **upstream** package from PyPI — a user following this fork's README never runs fork code. If the fork isn't meant to be PyPI-published, the instructions should point at the git URL (or note that the README describes the upstream install deliberately).

**Suggested fix:** decide the distribution story alongside finding #1; until then use `uv tool install git+https://github.com/mflux-community/mflux-fork` (or a callout).

### 20. Cosmetic

- `box_values.py:62`: error message is a plain string containing `{value}` (missing `f` prefix), and its example `10px` isn't actually accepted by the parser (only ints and `%`).
- `stepwise_handler.py:100`: filename `step{step}of{steps}` missing separator; `:104` rebuilds the composite from scratch every step (O(steps²) compositing).
- `defaults.py:63`: `QUANTIZE_CHOICES = [3, 5, 4, 6, 8]` — unsorted.
- `defaults.py:43-44, 61`: `MODEL_INFERENCE_STEPS` holds keys that are not CLI choices (`qwen-image`, `qwen-image-edit`, `ideogram-4-fp8`) — harmless, but the mapping is keyed by raw `--model` string, so entries only work when CLI defaults use those exact names.
- `.gitignore:4,11`: duplicate `.venv` entry (carried over unfixed from REPORT-01 finding #3).
- `README.md:124`: "qaility" typo.

## Not flagged (checked and ruled out)

- **All 33 console scripts** in `pyproject.toml` resolve to existing modules with `main()` — verified programmatically.
- **Scheduler math**: `FlowMatchEulerDiscreteScheduler` Euler step, empirical-mu polynomial, and terminal stretch; `LinearScheduler` sigma shift (`mu = m·w·h/256 + b` matches the FLUX seq-len formula); `SeedVR2EulerScheduler` `pred_x0`/`pred_noise` derivation — all checked against the flow-matching definitions.
- **`_compute_linear_mu`** is not dead — used by `fibo_edit.py:72`.
- **krea2 vs flux initializer drift**: the large diff is intentional simplification (krea2 = FluxTransformer + QwenVAE + Qwen3-VL encoder tap), not copy-paste rot.
- **`mflux/__init__.py`** is import-light (only sets `TOKENIZERS_PARALLELISM`), so `release.yml`'s `--no-deps` install is safe.
- **tests.yml**: CLI `-m fast` correctly overrides the ini `addopts` marker expression; `pytest-timer` absence is benign (optional plugin).
- **CHANGELOG ↔ version consistency**: `0.18.0` matches `pyproject.toml`; `Unreleased` holds krea2 work — normal flow, and `ReleaseValidator` enforces the match at release time.
- **`BatterySaver`**: macOS-only guarded, AC-power and non-MacBook cases handled, pmset parse failures degrade gracefully.
- **Alias duplicates** (`krea-dev`/`dev-krea`, `krea-2`/`krea2`) are intentional aliases in `AVAILABLE_MODELS`; both resolve to the same config and the missing `dev-krea` steps key falls back to the same 25 steps.
- **`ConfigResolution` substring inference** sorts by longest alias then priority, so `krea-dev` correctly beats `dev` — the remaining "any repo containing 'dev'" behavior is documented upstream behavior, not a bug.

## Overall assessment

Architecture and shared-abstraction discipline are good — the AGENTS.md rules (thin CLIs, shared plumbing, class-based helpers) are visibly honored, and the review found no numeric errors in any scheduler. The recurring weakness is **silent failure**: wrong-name PyPI targeting that can't trip its own guard (#1), upload errors that can't fail the release (#2), weight mismatches that can't surface (#3), save errors that exit 0 (#9), ignored flags (#10), and metadata merges that drop real values (#6). Most fixes are one-to-five-line changes; the release-pipeline pair (#1, #2) and the config-cloning bug (#5) are the ones worth doing before the next release or model addition.
