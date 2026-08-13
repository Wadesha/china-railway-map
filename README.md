# 中国铁路地图 · 演示数据

基于 **ECharts** 的中国铁路干线矢量地图演示，部署在 GitHub Pages，零后端、零地图 API key，纯前端矢量渲染。

> ⚠️ 本仓库所有线路里程、车站数量均为**演示近似值**，不代表铁路部门官方统计。

## 在线预览

- 主站（ECharts 矢量）：https://wadesha.github.io/china-railway-map/
- 子站（腾讯地图真实数据快照）：https://wadesha.github.io/china-railway-map/subsite/
- 仓库：`Wadesha/china-railway-map`

> 子站说明：本地通过**已连接的腾讯地图连接器**采集了真实数据——27 个枢纽站真实坐标（geocoder）、14 大枢纽周边真实 POI（地铁站/餐饮/酒店/旅游景点，placeSearchNearby）、5 组城际真实乘车方案（directionTransit，公交/地铁/高铁混合）——并内联打包进静态页。访问者**无需任何运行时 key** 即可查看真实数据。底图仍用 ECharts 矢量渲染（合规、含完整国界）。连接器代理仅在 WorkBuddy 内有效、无法跟随部署，故采用「本地采集→快照内联」方式；如需实时调用可自行在源码 `TENCENT_KEY` 填 key。

---

## 目录结构

```
china-railway-map/
├── index.html        # 主站最终产物（单文件，GeoJSON 已内联，约 589KB）
├── template.html     # 主站页面模板，含 __CHINA_GEOJSON__ 占位符
├── china.json        # 中国省级边界 GeoJSON（DataV，含台湾/港澳/南海诸岛）
├── build.js          # 主站：把 china.json 内联进 template.html → index.html
├── deploy.js         # 主站部署：GitHub Contents API 上传并启用 Pages
├── subsite/          # 子站（腾讯地图真实数据快照）
│   ├── index.html    # 子站产物（已内联 GeoJSON + 真实数据快照）
│   ├── template.html # 子站模板（含数据占位符）
│   ├── build.js      # 子站构建（内联 GeoJSON + 3 个快照）
│   └── push-subsite.js # 子站上传到仓库 subsite/ 目录
├── data/             # 腾讯地图真实数据快照（本地采集，2026-08-14）
│   ├── stations_real.json  # 27 个枢纽站真实坐标
│   ├── poi.json            # 14 大枢纽周边真实 POI（地铁站 / 餐饮 / 酒店 / 旅游景点）
│   └── routes.json         # 5 组城际真实乘车方案
└── README.md
```

`index.html` 是**唯一需要部署的文件**：GeoJSON 已内联，避免 `file://` 与跨域加载问题，本地双击或挂 Pages 均可直接运行。

---

## 技术说明与合规

- 地图渲染使用 **ECharts 5.5**（`geo` + `registerMap` 矢量绘制），不依赖 Google / OpenStreetMap / Mapbox 等受限瓦片源，也不嵌入任何地图服务 API key。
- 边界数据采用合规的中国省级 GeoJSON，包含台湾省、香港特别行政区、澳门特别行政区及南海诸岛，符合国家测绘标准。
- 干线以 `lines`（polyline）绘制，车站以 `effectScatter` 涟漪点呈现；右侧卡片点击可显隐对应线路（由隐藏图例驱动）。

---

## 本地预览

直接双击 `index.html` 即可（GeoJSON 已内联，无需起服务）。
如需本地静态服务：

```bash
cd china-railway-map
python3 -m http.server 8080
# 浏览器打开 http://localhost:8080/index.html
```

---

## 构建（修改模板或地图数据后）

```bash
node build.js     # 读取 template.html + china.json，生成 index.html
```

> 若想替换底图：把新的中国 GeoJSON 放到 `china.json`（需含台湾/港澳/南海诸岛），重新 `node build.js` 即可。

---

## 部署到 GitHub Pages

`deploy.js` 会：① 创建仓库（已存在则跳过）② 上传 `index.html` 到 `main` 分支 ③ 启用 Pages。

> token 从 `~/.workbuddy/MEMORY.md` 运行时正则读取，**不写进任何文件或回显**。

```bash
node deploy.js
```

首次启用 Pages 后约 1–2 分钟生效，之后推送 `index.html` 自动更新。

---

## 如何更新演示数据

线路与车站都在 `template.html` 顶部的 JS 常量里，改完重新 `build.js` + `deploy.js` 即可。

```js
const LINES = [
  {
    name: "京广线",
    en: "Beijing–Guangzhou",
    color: "#ff5c5c",
    km: 2289,                       // 演示里程（km）
    coords: [                       // 途经站 [经度, 纬度]，近似坐标
      [116.41, 39.90],  // 北京
      [114.51, 38.04],  // 石家庄
      [113.65, 34.76],  // 郑州
      [114.30, 30.59],  // 武汉
      [112.94, 28.23],  // 长沙
      [113.26, 23.13],  // 广州
    ],
  },
  // …… 其余线路
];
```

- 新增线路：照格式加一项（`name`/`en`/`color`/`km`/`coords`）。
- 车站点：由 `coords` 自动聚合去重，经停线路越多圆点越大，无需手动维护。
- 换成真实数据：把 `coords` 替换为真实经纬度表，或接入线路 GeoJSON 后改为分段绘制。

---

## 重新采集真实数据快照（子站）

真实数据由 WorkBuddy 内已连接的**腾讯地图连接器**（MCP 工具）本地采集，落盘为 `data/*.json`，再由 `subsite/build.js` 内联进静态页。流程：

1. 在 WorkBuddy 中确保 `tencent-map` 连接器已连接。
2. 用连接器工具采集（单次并发 ≤3 以避开代理限流）：
   - `geocoder` / `placeSuggestion`：抓车站真实坐标 → `data/stations_real.json`
   - `placeSearchNearby`：抓枢纽周边 POI（中心点取真实站坐标，半径约 2.5km）→ `data/poi.json`
   - `directionTransit`：抓城际乘车方案（from/to 用真实坐标）→ `data/routes.json`
3. `cd subsite && node build.js` 重新内联，`node push-subsite.js` 重新发布。

> 合规提示：仅采集公共设施 / 公开商业 POI，量级克制（每类取前 10 条），快照保留「数据来源：腾讯地图」与采集日期；不在 Pages 上做无 key 的实时调用。

---

## 下一步可扩展

- 接入真实铁路线网 GeoJSON / 车站经纬度库，替换演示数据
- 增加指标面板：线路里程排行、客流量、动车组配属等图表
- 增加搜索定位、按铁路局/区域筛选、线路详情弹窗
