# 大 Sketch 打开首屏（2026-11-07）

叠在 `cursor/zoom-coverage-continuity-2499` 之上。不改光栅预算，不改目录条目和图片的下标对应。

## 原因

`useSourceOpening` 要等 `items.length===total` 才把预览交到画布。十万级素材目录要先传完条目，第一张已经解码的缩略图也看不见。打开 100–240MB 的拓扑 / test.sketch 时，首屏就卡在这次整表复制上。

## 改动

目录前缀里已经有对应条目的第一张图就可以画。图片下标必须落在已到达的条目前缀里，避免和优先序渲染错位。轮询继续，直到条目和图片都齐。`finished` 仍只在那时为真。

100–240MB 原件的整份导入、第一张原生精确瓦片仍是开放项。这一步只把已经解码的第一张板子提前画出来。

## 复测

拓扑素材或 test.sketch：目录还在传时，第一张已解码缩略图应先出现在打开预览上，不必等条目总数到齐。`source-opening-preview.test.ts` 锁定前缀长度为 2 时就能看到 `data:image/png;base64,first`。

没有 Electron/GPU 回放。33/100ms 与完整 G2/G5 未关闭。
