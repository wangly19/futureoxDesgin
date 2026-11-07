# 冷缩出、大文件首屏、落定让路（2026-11-07）

叠在覆盖栈 PR（`cursor/zoom-coverage-continuity-2499`）之上。平移灰条不再改；D 的覆盖栈逻辑保持。

## 1. 冷缩出

素材画板以前在视口齐了就停，不建页级粗覆盖。普通文件要等完全空闲才暖总览。第一次缩出因此要等第一张粗瓦片现场光栅。

现在：视口至少一半已经在缓存、用户没在动时，大源（节点 ≥3000、带素材键、或边长 ≥8000）先排一张最近的页级总览瓦片，最多一张，不占缩放手势。总览瓦片在用户开始移动时取消，把光栅 owner 还给当前视口。已有 `navigationOverview` 尺度，不新开预算。

## 2. 大文件打开首屏

`useSourceOpening` 以前要等 `items.length===total` 才把预览交到画布。十万级素材目录要先传完条目，第一张缩略图也看不见。

现在：目录前缀里已经有对应条目的第一张图就可以画。轮询继续，直到条目和图片都齐，`finished` 仍只在那时为真。

## 3. 落定到精确相位

仍是一个 GPU raster owner，不并行第二套光栅。缩出/平移的覆盖批次改成一张，避免第二张粗瓦片粘在同一次请求里。松手后 `retainInFlightTile(..., settling)`：视口已经被居民涂满的邻近密度立刻让位；还在补洞的那张留下。

## 验证

`coverage-stack.test.ts`、`NativeSourceSurface.test.tsx`、`NativeSourceSurface.pipeline.test.tsx`、`source-opening-preview.test.ts` 共 36/36。没有 Electron/GPU 回放。33/100ms 与完整 G2/G5 未关闭。整页 mip 金字塔、第二 raster owner、100–240MB 原件的秒级导入仍开放。

## 复测

- 117.sketch：打开后停一下再快缩出，第一条不应再空等整页粗瓦片从零开始。平移条仍不应是 `#f5f5f5`（#47）。松手后精确相应比整组粗瓦片先占住 owner 更早开始。
- 拓扑 / test.sketch 量级：目录还在传时，第一张已解码缩略图应先出现在打开预览上。
