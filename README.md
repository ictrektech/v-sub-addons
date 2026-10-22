# v-sub-addons

V-Sub VOS 镜像内置的 whisper.cpp CUDA addon（预编译，Git LFS 存储）。

`ictrektech/V-Sub` 是 `buxuku/SmartSub` 的公开 fork，GitHub 不允许向公开 fork
上传 LFS 对象，因此预编译产物集中放在本仓库，由 `docker/build_image.sh` 在构建
镜像时按 profile 拉取并校验 sha256。

## 产物

| 文件 | profile | CUDA arch | 大小 | sha256 |
| --- | --- | --- | --- | --- |
| `thor.node` | thor_spark | sm_110（CUDA 13） | 66,407,960 | `d5e86cf00db790beea2f3b600dc2e63346339498ed3b8be71e4df4e1744d68a6` |
| `l4t.node` | l4t | sm_87（CUDA 12，`-DGGML_CUDA_NO_VMM=ON`） | 59,968,200 | `421547ceadb7ad4a7c45ca02c3a725764cbbad045eae1a128664c8e9e9f9a62d` |
| `arm-cuda.node` | ARM_with_cuda | sm_90;sm_100;sm_120a（CUDA 13） | 142,311,952 | `c1fdb61a49967323641a202195b5a82d767d5975b2b824d4e1f15fbde1a60f14` |

AMD_with_cuda 不内置该插件（上游 x64 预编译包可直接下载）。

## 来源与复现

- 上游：`ggml-org/whisper.cpp` tag `v1.9.5`
- 补丁：`buxuku/whisper.cpp` commit `9c38c27391111bdf4e80355e14a3460ef13152d2`
  - `0001-addon.node-cancel-via-AbortSignal.patch`
  - `0002-addon.node-emit-tokens-and-vad-segments.patch`
- 构建脚本：`ictrektech/V-Sub` 的 `scripts/build-whisper-cuda.mjs`，在对应 GPU
  base 镜像内运行，例如：

```bash
node scripts/build-whisper-cuda.mjs --architectures=110 --out=thor.node
node scripts/build-whisper-cuda.mjs --architectures=87 --no-vmm --out=l4t.node
node scripts/build-whisper-cuda.mjs --architectures=90\;100\;120 --out=arm-cuda.node
```

升级上游或补丁后需重新构建并更新本仓库与上面的 sha256，同时在 V-Sub 的
`docker/whisper-cuda/manifest.json` 中同步版本与校验值。
