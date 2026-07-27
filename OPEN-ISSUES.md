# MFlux-OG Open Issues

These are the open issues on [Filip's repo](https://github.com/filipstrand/mflux/issues) at Monday 27 July 2026 <br><br>



**[487](https://github.com/filipstrand/mflux/issues/487)** 
**Microsoft Mage-Flow**<br>
**2026-07-26**<br>
Pall13 Request to add support for Microsoft's Mage-Flow text-to-image and Mage-Flow-Edit instruction-based editing models, citing concern that mflux risks becoming obsolete.

**[486](https://github.com/filipstrand/mflux/issues/486)** 
**mflux-generate-krea2 fails to install in v0.18.0**<br>
**2026-07-24**<br>
bobosola The `mflux-generate-krea2` executable is missing from the v0.18.0 install list when using `uv tool install --upgrade mflux`, resulting in a "command not found" error.

**[484](https://github.com/filipstrand/mflux/issues/484)** 
**Qwen-Image 4-bit: quantization noise compounds per denoising step**<br>
**2026-07-24**<br>
SteveGJones With a 4-bit quantized Qwen-Image model on mflux 0.18.0, background grain grows monotonically with step count rather than converging, suggesting per-step quantization error accumulation specific to the Qwen-Image transformer path.

**[480](https://github.com/filipstrand/mflux/issues/480)** 
**Has anyone tried PiD with z-image-turbo or FLUX.2 Klein 9B?**<br>
**2026-07-23**<br>
ianscrivener Community question asking whether NVIDIA-trained PiD (Portrait-in-Detail) ControlNet models are compatible with the Z-Image-Turbo or FLUX.2 Klein 9B pipelines in mflux.

**[479](https://github.com/filipstrand/mflux/issues/479)** 
**What does the next iteration of the MFlux project look like?**<br>
**2026-07-20**<br>
ianscrivener Discussion thread proposing ways to grow the mflux project, including expanding the core developer team, adding GitHub Discussions, and broadening community contributions across HuggingFace, RunPod, and documentation.

**[477](https://github.com/filipstrand/mflux/issues/477)** 
**No News?**<br>
**2026-07-17**<br>
Pall13 Community concern that the project may have been abandoned due to a lack of recent activity, spawning a discussion thread with 11 comments.

**[473](https://github.com/filipstrand/mflux/issues/473)** 
**MFlux HuggingFace Community Space**<br>
**2026-07-12**<br>
ianscrivener Announcement of a new [mflux-community HuggingFace organisation](https://huggingface.co/mflux-community) created to host community models and resources.

**[472](https://github.com/filipstrand/mflux/issues/472)** 
**Add more collaborators?**<br>
**2026-07-11**<br>
plz12345 Request for the project owner to add additional collaborators to help clear the backlog of open PRs given the maintainer's limited availability.

**[470](https://github.com/filipstrand/mflux/issues/470)** 
**Perf: `_hist_match` inverts a permutation via a second `np.argsort` (O(N log N) → O(N))** 
**2026-07-10**<br>
bobmatnyc Performance bug in `SeedVR2Util._hist_match` where a permutation inverse is computed via a redundant O(N log N) sort instead of an O(N) scatter, causing ~125× slower performance and multi-minute stalls on large images.

**[469](https://github.com/filipstrand/mflux/issues/469)** 
**FIBO (base briaai/FIBO) txt2img produces pure noise**<br>
**2026-07-09**<br>
gethyper `mflux-generate-fibo` produces rainbow-confetti noise for all prompts because the transformer's velocity prediction is ~5–8× too small, causing the latent to never leave the initial noise state across all 50 steps.

**[467](https://github.com/filipstrand/mflux/issues/467)** 
**Golden image comparator: atol is hardcoded to 0**<br>
**2026-07-04**<br>
fxd0h Low-priority observation that the image test comparator uses `atol=0`, making near-black pixel comparisons effectively exact-match and causing spurious test failures on different Apple Silicon hardware where shadow details differ slightly.

**[466](https://github.com/filipstrand/mflux/issues/466)** 
**Request for inclusion in Related Projects**<br>
**2026-07-04**<br>
plz12345 Request to list [MLXBits Image Studio](https://github.com/MLXBits/image-studio), a macOS SwiftUI GUI wrapper around mflux, in the project README's Related Projects section.

**[460](https://github.com/filipstrand/mflux/issues/460)** 
**Stepwise images often don't provide a useful preview of intermediate states**<br>
**2026-06-29**<br>
fortinmike Experimental stepwise image previews in mflux rarely show meaningful intermediate states until very late in generation, unlike ComfyUI's Latent2RGB approach, making them unsuitable as real-time progress indicators in a GUI app.

**[458](https://github.com/filipstrand/mflux/issues/458)** 
**LoRA loader splits the shared rank/down dimension for fused qkv layers → matmul shape error**<br>
**2026-06-28**<br>
deadmansahil `LoraTransforms._split_qkv_mlp_down` incorrectly slices the shared down-projection of fused QKV LoRA weights when rank is divisible by 4, causing a matmul shape mismatch on the first denoise step for popular community FLUX LoRAs with rank 64.

**[455](https://github.com/filipstrand/mflux/issues/455)** 
**Multiple LoRAs**<br>
**2026-06-25**<br>
Pall13 Feature question asking whether mflux supports associating specific LoRAs with specific characters or regions to prevent identity bleed when generating scenes with multiple people using separate character LoRAs.

**[437](https://github.com/filipstrand/mflux/issues/437)** 
**Option to suppress or reduce EXIF metadata writing**<br>
**2026-06-09**<br>
plz12345 Request to make the embedded EXIF metadata (which includes full prompt, seed, LoRA paths, etc.) optional rather than always-on, as it allows third parties to fully reproduce an artist's private generation settings.

**[431](https://github.com/filipstrand/mflux/issues/431)** 
**Update**<br>
**2026-06-01**<br>
blogmilancom-source General request asking when the next update will arrive with new model support, including Ernie-Image, Microsoft Lens, Bonsai-Image, and FLUX.2 Klein 9B-KV.

**[425](https://github.com/filipstrand/mflux/issues/425)** 
**WHERE are these model files stored?**<br>
**2026-05-28**<br>
buggz2k User question asking where mflux downloads and caches HuggingFace model files on disk, having noticed large downloads occurring without a clear storage location.

**[424](https://github.com/filipstrand/mflux/issues/424)** 
**Feature Request: model support for Microsoft Lens**<br>
**2026-05-27**<br>
ianscrivener Request to add support for Microsoft's Lens 3.8B image generation model (using GPT-OSS encoder and Flux2 VAE), which reportedly punches above its weight for Macs with modest memory.

**[423](https://github.com/filipstrand/mflux/issues/423)** 
**LoRA `_split_qkv_mlp_down` splits shared down-projection and corrupts LoRA weights**<br>
**2026-05-26**<br>
danielkhalilov Duplicate bug report of #458: `_split_qkv_mlp_down` incorrectly splits the shared LoRA down-projection for common ranks (4, 8, 16, 32), silently corrupting LoRA decomposition and producing incorrect results.

**[415](https://github.com/filipstrand/mflux/issues/415)** 
**FLUX.2-klein-9b-kv model not properly recognized in image metadata**<br>
**2026-04-30**<br>
Pall13 The `flux2-klein-9b-kv` variant is not listed in the `--base-model` options for `mflux-generate-flux2`, and metadata stores a full file path instead of a human-readable model name.

**[414](https://github.com/filipstrand/mflux/issues/414)** 
**Can mflux support distributed inference with MLX?**<br>
**2026-04-22**<br>
southkorea2013 Feature request to support distributed inference across multiple Macs via MLX's `mlx.launch`, leveraging macOS RDMA over Thunderbolt 5 and JACCL to reduce generation time.

**[411](https://github.com/filipstrand/mflux/issues/411)** 
**tools/inpaint_mask_tool.py DNE?**<br>
**2026-04-13**<br>
carcinocron Documentation bug: the README references `tools/inpaint_mask_tool.py` for inpainting workflows, but the file does not exist in the repository.

**[410](https://github.com/filipstrand/mflux/issues/410)** 
**Expose latents with some extra functionality**<br>
**2026-04-11**<br>
pericalypsis Feature request to expose input/output latents via the API, enabling multi-model refinement workflows with `denoising_start`/`denoising_end` parameters for seamless handoff between a base model and a refiner.

**[407](https://github.com/filipstrand/mflux/issues/407)** 
**Enable VAE tiling for FLUX.2 Klein to reduce peak memory ~50%**<br>
**2026-04-09**<br>
TimPietruskyRunPod The VAE decode stage accounts for ~70% of peak memory in FLUX.2 Klein generation; enabling the existing `VAETiler` (already used for video) on image models could reduce peak memory from ~14 GB to ~6–7 GB with no quality loss.

**[401](https://github.com/filipstrand/mflux/issues/401)** 
**Shared Directory for Models with ollama?**<br>
**2026-04-04**<br>
yosun User question asking whether mflux can share a model directory with Ollama to avoid duplicate downloads.

**[399](https://github.com/filipstrand/mflux/issues/399)** 
**Feature Request: Stable Diffusion 3.5 Support**<br>
**2026-04-03**<br>
CodeAndCanvas728 Detailed proposal to add SD3.5 (Large, Medium, Turbo) to mflux, noting significant architectural overlap with FLUX and the lack of any actively maintained MLX implementation since DiffusionKit was archived in March 2026.

**[398](https://github.com/filipstrand/mflux/issues/398)** 
**Incompatible protobuf constraint (<6.0) conflicts with mlx-audio (>=6.33.5)** 
**2026-04-03**<br>
LangeRobert Dependency conflict: mflux pins `protobuf<6.0` while `mlx-audio` requires `>=6.33.5`, making it impossible to install both in the same environment with Poetry or uv.

**[394](https://github.com/filipstrand/mflux/issues/394)** 
**Your project is featured in Awesome MLX**<br>
**2026-03-26**<br>
raullenchai Notification that mflux has been listed in the [Awesome MLX](https://github.com/raullenchai/awesome-mlx) curated directory of 120+ MLX-based projects.

**[386](https://github.com/filipstrand/mflux/issues/386)** 
**Benchmark table / does mflux work with Macbook Air M5 and 32GB RAM?**<br>
**2026-03-18**<br>
timaschew Community question about mflux performance on a MacBook Air M5 with 32 GB RAM, noting the benchmark table was removed from the README.

**[385](https://github.com/filipstrand/mflux/issues/385)** 
**Image Size Stretch using Flux Klein Image Edit with different aspect ratio**<br>
**2026-03-17**<br>
royaldust Bug where using a 1:1 reference image with a different output aspect ratio (e.g., 3:4) in Flux Klein Image Edit produces a stretched output image instead of a properly recomposed one.

**[373](https://github.com/filipstrand/mflux/issues/373)** 
**Implement Gradient Accumulation**<br>
**2026-03-06**<br>
waldheinz Feature request and proposal to add gradient accumulation to the `TrainingTrainer.train` method via a new `grad_accum_steps` parameter, enabling training on hardware with limited memory.

**[357](https://github.com/filipstrand/mflux/issues/357)** 
**Design review and discussion: MFLUX v1.0 API and CLI**<br>
**2026-02-16**<br>
filipstrand Open design discussion from the project owner inviting community input on a redesigned, unified CLI for mflux v1.0 covering command names, model/variant specification, resolution handling, and low-RAM flags.

**[355](https://github.com/filipstrand/mflux/issues/355)** 
**[Feature(s)] Sigma schedules, aspect ratios, shift override, MCF sampler for Z-Image**<br>
**2026-02-15**<br>
terribilissimo PR-linked feature request proposing opt-in enhancements for Z-Image pipelines: alternative sigma schedules (cosine, Karras, exponential), MCF sampler, aspect ratio presets, sigma shift override, and descriptive filenames.

**[324](https://github.com/filipstrand/mflux/issues/324)** 
**FR: SVG output**<br>
**2026-01-16**<br>
twalderman Feature request to add a `--vector`/`--svg` post-processing flag that converts generated raster images to SVG using tools like vtracer, with style presets for logos, icons, illustrations, and silhouettes.

**[320](https://github.com/filipstrand/mflux/issues/320)** 
**Is SeedVR2's image upscaling speed normal?**<br>
**2026-01-13**<br>
weigeloveu User question asking whether a 2 min 18 sec upscale time with 18.39 GB peak memory for a 4× SeedVR2 upscale on an M4 Mac mini with 16 GB is expected behaviour.

**[312](https://github.com/filipstrand/mflux/issues/312)** 
**mflux-generate-fibo - Prompt is too long (2414 bytes), truncating to 2000 bytes for IPTC**<br>
**2026-01-03**<br>
zaskara Every generation with `mflux-generate-fibo` emits a warning that the prompt is being truncated to 2000 bytes for IPTC metadata, with no clear way to suppress or work around it.

**[311](https://github.com/filipstrand/mflux/issues/311)** 
**mflux-generate-z-image-turbo takes twice VRAM at end of generation**<br>
**2026-01-03**<br>
zaskara Memory usage spikes to 1.5–2× normal at the end of Z-Image-Turbo generation, and the model unloads from VRAM after every prompt with no option to keep it resident between generations.

**[309](https://github.com/filipstrand/mflux/issues/309)** 
**Supporting Z-Image-Turbo-Fun-Controlnet-Union-2.1?**<br>
**2026-01-01**<br>
shaoju Feature request to add ControlNet support for Z-Image via the `alibaba-pai/Z-Image-Turbo-Fun-Controlnet-Union-2.1` model to enable structural/pose control during generation.

**[308](https://github.com/filipstrand/mflux/issues/308)** 
**[New Model] Qwen-Image-2512 is coming**<br>
**2025-12-31**<br>
jiangdi0924 Heads-up that Qwen-Image-2512 has been released on HuggingFace, implicitly requesting mflux support.

**[299](https://github.com/filipstrand/mflux/issues/299)** 
**[New Model] Qwen-Image-Layered**<br>
**2025-12-20**<br>
jiangdi0924 Heads-up that Qwen-Image-Layered has been released on HuggingFace, implicitly requesting mflux support.

**[298](https://github.com/filipstrand/mflux/issues/298)** 
**Support Qwen Image Edit 2511**<br>
**2025-12-16**<br>
filipstrand Maintainer-filed tracking issue to add support for Qwen Image Edit 2511 (recently merged into diffusers), while keeping support for 2509 given existing community LoRA investment.

**[296](https://github.com/filipstrand/mflux/issues/296)** 
**Can't use quantized models of schnell, dev, qwen or fibo**<br>
**2025-12-15**<br>
timaschew Multiple models fail to load when using symlinked HuggingFace cache paths, with errors ranging from missing tokenizer dependencies (protobuf, tiktoken) to shape mismatches in the Qwen text encoder.

**[294](https://github.com/filipstrand/mflux/issues/294)** 
**[New Model] NewBie image Exp0.1 3.5B**<br>
**2025-12-07**<br>
amadeuzou Heads-up that NewBie-image-Exp0.1 3.5B has been released on HuggingFace, implicitly requesting mflux support.

**[292](https://github.com/filipstrand/mflux/issues/292)** 
**New model - LongCat-Image 6B**<br>
**2025-12-05**<br>
amadeuzou Heads-up that LongCat-Image 6B (and Edit/Dev variants) from Meituan have been released on HuggingFace, implicitly requesting mflux support.

**[280](https://github.com/filipstrand/mflux/issues/280)** 
**Add support for FLUX 2**<br>
**2025-11-25**<br>
rolux Request to add support for the newly announced FLUX 2 model family from Black Forest Labs.

**[266](https://github.com/filipstrand/mflux/issues/266)** 
**Proposal: install CodeRabbit code review**<br>
**2025-09-29**<br>
anthonywu Proposal for the project owner to install the free CodeRabbit AI code review bot on the mflux GitHub repo to help manage the PR backlog.

**[262](https://github.com/filipstrand/mflux/issues/262)** 
**Question about using 4-bit dev-kontext**<br>
**2025-09-06**<br>
Satwato User reports that `mflux-generate-kontext` with a locally-cloned 4-bit Kontext model either outputs the unchanged input image, or rejects `dev-kontext` as an invalid `--model` / `--base-model` value.

**[260](https://github.com/filipstrand/mflux/issues/260)** 
**ValueError: invalid literal for int() with base 10: 'None' when mflux-generate with a non-quantized model**<br>
**2025-09-02**<br>
southkorea2013 Using `mflux-generate` with a locally saved non-quantized model raises a `ValueError` in `QuantizationUtil.quantize_model` because `q_level` is `None` and is passed directly to `int()`.

**[251](https://github.com/filipstrand/mflux/issues/251)** 
**mflux-save doesn't support local directories in -m**<br>
**2025-08-07**<br>
greggh `mflux-save` rejects local filesystem paths for the `--model` argument, requiring an `org/model` HuggingFace format, which forces unnecessary re-downloads of large models already present on disk.

**[248](https://github.com/filipstrand/mflux/issues/248)** 
**Open discussion: mflux-extras, affiliated libraries**<br>
**2025-08-05**<br>
anthonywu Proposal to create a separate `mflux-extras` package (or namespace packages) to house out-of-scope features like Gradio GUI, REST API, ComfyUI integration, and MCP server without bloating the core library.

**[242](https://github.com/filipstrand/mflux/issues/242)** 
**Upcoming Side Project: SwiftUI over (MCP or FastAPI) over mflux core**<br>
**2025-08-02**<br>
anthonywu Announcement of a concept project building a native macOS SwiftUI frontend connected to mflux via a FastAPI or MCP server intermediary, to work around Swift app sandbox restrictions on Python process access.

**[231](https://github.com/filipstrand/mflux/issues/231)** 
**Missing option to generate images from custom embeds and latents**<br>
**2025-08-02**<br>
_(truncated in API response)_ Feature request to expose prompt embeddings and latent tensors as inputs/outputs, enabling animation interpolation workflows and other advanced use cases requiring direct latent manipulation.
