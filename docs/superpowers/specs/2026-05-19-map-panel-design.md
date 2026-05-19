# 設計文件：航海地圖收集面板

**日期：** 2026-05-19  
**目標：** 將右側四色合成收集面板（`drawCollectionPanel`）改為像素 RPG 航海地圖風格

---

## 現狀

- `drawCollectionPanel()` 在右側畫一個 70px 寬的小面板，4 個色塊（紅黃藍綠）
- `synthesizedColors: Set` 記錄本局已合成的顏色（0-3）
- 四色全亮 → 觸發 JP（`bossCardPick`）

## 新設計

### 視覺風格
- 像素 RPG 地圖：深藍/深綠漸層背景，細格線暗紋（座標格感）
- 維持現有側欄空間（約 200px 寬，與盤面等高）

### 4 個地形點（不規則散布）
| colorId | 顏色 | 地形 | 地圖相對位置 |
|---------|------|------|-------------|
| 0 | 🔴 紅 | 火山島港口 🌋 | 左上偏中 |
| 1 | 🟡 黃 | 沙漠城市 🏜️ | 右上 |
| 2 | 🔵 藍 | 冰山要塞 🧊 | 左下 |
| 3 | 🟢 綠 | 叢林神殿 🌿 | 右下偏中 |

### 目標點狀態
- **未完成**：灰白輪廓 icon，透明度 35%，旗桿空
- **完成**：地形 icon 亮起對應顏色，彩色旗子飄動

### 船（`shipMarker`）
- 初始位置：地圖中央偏下「出發港」
- 每次 `synthesizedColors.add(colorId)` 觸發：船飛向對應地形點
- 若目標已插旗：船跳過去停一下再回原位，不重複插旗
- 新局：船回出發港，所有旗子清空

### 動畫
- **拋物線飛行**：20 幀，弧高 = 水平距離 × 0.4，easeInOut
- **落點濺起**：5 個小圓向外擴散 15px，10 幀消失
- **旗子展開**：8 幀從旗桿底部滑出
- **旗子飄動**：`sin(Date.now() / 400)` × 3°，持續搖擺
- **四旗全插**：面板外框閃金光（與原 allLit 邏輯同步）

---

## 實作範圍

### 新增狀態
```javascript
let mapShip = { x: 0, y: 0, tx: 0, ty: 0, frame: 0, flying: false };
let mapFlags = [false, false, false, false];  // 各色是否已插旗
```

### 修改位置
- `drawCollectionPanel()` → 整個替換為 `drawMapPanel()`
- `applySingleSynthesis()` 中 `synthesizedColors.add(colorId)` 後觸發船動畫
- `resetRoundState()` 中清空 `mapFlags`、重置船位置

### 不改動
- `synthesizedColors` 邏輯不變
- JP 觸發條件（`synthesizedColors.size >= 4`）不變
- 面板的 `cx` 位置不變
