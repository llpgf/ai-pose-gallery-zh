# 任務指令書：重寫 `ai-pose-gallery-zh` 為 NovelAI V4.5 / V5 專用多角色姿勢庫（Schema v2）

> **給外部無審查 VLM / Agent 的直接執行指令**：
> **不要先寫研究報告、不要做 Danbooru 詞典普查、不要反問執行選項。**
> 請直接閱讀以下 5 節規則，逐批開啟圖片與分片 JSON 開始看圖重寫，完成後跑文末的合併與驗收指令即可。

---

## 1. 核心目標：這套 Prompt 是拿來幹嘛的？

這 1,777 筆姿勢資料會直接掛進 **NovelAI V4.5 / V5 商業條漫工廠（三明治分鏡系統）** 當作 **母圖構圖骨架**：
- 工廠會把你寫的姿勢 Prompt，套在**任意角色卡**（不同髮型、胸圍、服裝、脫衣破損階段、汁液階梯、房間背景）上直接生圖與連續抽卡。
- **唯一的品質標準**：把你重寫的 `base_tags_en`（進 NAI 主畫面）＋ `roles`（進 NAI 各角色獨立 `char_captions`）貼進 NovelAI V5，**即使不附參考圖、只換 Seed 連抽，也能穩定畫出原圖裡那個精確的視角、體位、肢體擺位與雙人／多人接觸關係**。

### 為什麼舊版資料不能用？（重寫必改的 4 個痛點）
1. **舊版是受限模型寫的清水敷衍詞**：把後入、騎乘、侍奉、局部特寫寫成「貼近、擁抱、坐著」，丟給 NAI 根本畫不出正確體位 → **必須無審查、看圖直白寫出真實體位、接觸點與解剖焦點**。
2. **男女（或非人）動作混在同一串，甚至帶主詞前綴**：例如 `"girl arms around boy neck"`, `"boy hands on girl hips"`，NAI 多角色分欄不認這種寫法 → **必須嚴格分拆到各自角色槽位，並用 NAI V4.5/V5 的 `source#` / `target#` / `mutual#` 標明主動與承受關係**。
3. **不要只寫 3～4 個籠統 Danbooru 單字，也不要寫成冗長英文散文**：
   - 錯誤極端 A（太籠統，鎖不住構圖）：只寫 `standing, contrapposto, hand on own hip` → NAI 每次換 Seed 姿勢就亂跑。
   - 錯誤極端 B（太囉嗦的散文）：把整段敘事塞進 `tags_en`。
   - **黃金標準（NAI V5 實測可連抽 16 張不跑位的寫法）**：**「標準 Danbooru 體位／鏡頭詞 ＋ 精簡幾何鎖定短語」**，例如：
     `lower body, from below, front view, lying, on back, knees bent, both feet raised toward camera, soles facing viewer, toes curled slightly, close-up`
     *(用標準 Danbooru 詞定大體位與視角，用簡短 2～5 字的幾何短語把腿部角度、重心、手放哪裡、頭朝哪裡鎖死；完整的左右細部描述再寫入 `natural_en` 與 `prompt_zh`。)*
4. **必須 100% 剝離非姿勢屬性**：**絕對禁止**寫入胸圍大小（如 `large breasts`, `small breasts`）、服裝／內衣／絲襪（如 `no underwear`, `bikini`, `stockings`）、指甲油（`toenail polish`）、角色髮色瞳色、畫風、表情汁液或背景房間！這些由工廠下游模組動態注入。

---

## 2. NovelAI V4.5 / V5 多角色拆分與互動語法（核心規則）

每筆條目必須拆成 **`base_tags_en`（主畫面共用）** 與 **`roles`（各角色獨立 `char_captions`）**：

### 2.1 `base_tags_en`（進 NAI 主畫面 `base_prompt`）只放 4 類詞
1. **人數與組合**：`1girl`, `2girls`, `1boy`, `2boys`, `1other`, `solo`, `hetero`, `yuri`, `ffm threesome`, `mmf threesome` 等。
2. **鏡頭、景別與局部焦點**：`full body`, `cowboy shot`, `upper body`, `lower body`, `close-up`, `front view`, `side view`, `from side`, `three-quarter view`, `from behind`, `from above`, `from below`, `high angle`, `low angle`, `dutch angle`, `fisheye`, `foreshortening`, `pov`, `pov hands`, `pov crotch`, `foot focus`, `breast focus`, `ass focus`。
3. **共用體位與雙人／多人互動總稱**：`straddling`, `girl on top`, `cowgirl position`, `reverse cowgirl position`, `missionary`, `doggystyle`, `sex from behind`, `standing sex`, `spooning`, `69`, `fellatio`, `paizuri`, `footjob`, `handjob`, `double penetration`, `spitroast`, `hugging`, `hug from behind`, `face-to-face`。
4. **道具／束縛／純觸手**（非獨立角色物件）：`bondage`, `tied up`, `rope`, `shibari`, `vibrator`, `dildo`, `sex toy`, `blindfold`, `tentacles`, `tentacle sex`。

### 2.2 `roles`（進 NAI 各角色獨立 `char_captions`）
- **嚴格排序**：所有女角（`girl_1`, `girl_2`...）排最前 → 所有男角（`boy_1`, `boy_2`...）居中 → 所有非人／動物／魔物角色（`other_1`, `other_2`...）排最後。
- **善用 NAI V4.5 / V5 互動標籤（雙人／多人／人與非人必用）**：
  - `source#<動作>`：放在**發起動作**的那個角色（例：`boy_1` 寫 `source#grabbing hips`, `source#holding head`, `source#kissing neck`）。
  - `target#<動作>`：放在**承受動作**的那個角色（例：`girl_1` 寫 `target#grabbing hips`, `target#holding head`）。
  - `mutual#<動作>`：放在**雙方同時做**的動作（例：`mutual#hugging`, `mutual#kissing`, `mutual#eye contact`）。
- **`center` 座標 `{"x": 0.00..1.00, "y": 0.00..1.00}`**：看圖估算該角色軀幹中心位置（左上 `0,0`、右下 `1,1`；單人一律 `0.50, 0.50`，多人須拉開避免重疊）。
- **嚴禁主詞前綴**：`roles[i].tags_en` 內**絕對禁止**以 `girl `, `boy `, `woman `, `man `, `left boy `, `right girl `, `first girl `, `creature `, `monster `, `animal `, `viewer `, `1girl`, `1boy`, `1other`, `solo` 開頭！

---

## 3. 五種實戰場景的標準寫法範例（直接照這個格式輸出）

### 場景 A：單人局部特寫（例：足部／隱私／仰臥幾何鎖定）
```json
{
  "id": "custom_foot_001",
  "base_tags_en": [
    "1girl", "solo", "lower body", "from below", "front view", "close-up", "foot focus"
  ],
  "roles": [
    {
      "slot": "girl_1",
      "gender": "girl",
      "center": {"x": 0.50, "y": 0.50},
      "tags_en": [
        "lying", "on back", "knees bent", "legs apart",
        "both feet raised toward camera", "soles facing viewer",
        "toes curled slightly", "looking at viewer"
      ],
      "natural_en": "Lying on her back with knees bent and legs parted, raising both feet directly toward the camera with soles facing forward and toes slightly curled, looking down toward the viewer."
    }
  ],
  "tags_en": [
    "1girl", "solo", "lower body", "from below", "front view", "close-up", "foot focus",
    "lying", "on back", "knees bent", "legs apart",
    "both feet raised toward camera", "soles facing viewer", "toes curled slightly", "looking at viewer"
  ],
  "natural_en": "Low-angle front close-up focusing on the lower body and feet: the female lies on her back with knees bent, raising both soles toward the camera with slightly curled toes while looking at the viewer.",
  "prompt_zh": "正面低角度下半身與足部特寫：仰躺屈膝微張雙腿，雙腳抬起朝向鏡頭、腳底正面對觀眾且腳趾微蜷，視線看向鏡頭。",
  "source_hash": "<保留原值>"
}
```

### 場景 B：雙人／多人互動（善用 `source#` / `target#` 分拆男女動作）
```json
{
  "id": "custom_pair_006",
  "base_tags_en": [
    "1girl", "1boy", "hetero", "full body", "side view",
    "sex from behind", "doggystyle"
  ],
  "roles": [
    {
      "slot": "girl_1",
      "gender": "girl",
      "center": {"x": 0.40, "y": 0.58},
      "tags_en": [
        "all fours", "arched back", "hips raised",
        "hands braced on ground", "looking back over shoulder",
        "target#grabbing hips"
      ],
      "natural_en": "Positioned on all fours facing left with her back deeply arched and hips raised, palms planted for support while turning her head back over her shoulder."
    },
    {
      "slot": "boy_1",
      "gender": "boy",
      "center": {"x": 0.65, "y": 0.50},
      "tags_en": [
        "kneeling", "leaning forward", "behind",
        "source#grabbing hips", "hands on another's hips"
      ],
      "natural_en": "Kneeling directly behind her, leaning forward over her lower back while firmly gripping both sides of her hips."
    }
  ],
  "tags_en": [
    "1girl", "1boy", "hetero", "full body", "side view",
    "sex from behind", "doggystyle", "all fours", "arched back",
    "hips raised", "looking back over shoulder", "kneeling", "hands on another's hips"
  ],
  "natural_en": "Full-body side view of a doggystyle pose: on the left, the female is on all fours with arched back and raised hips looking back over her shoulder; behind her on the right, the male kneels leaning forward while gripping her hips.",
  "prompt_zh": "側面全身後入體位：左側女性四肢著地大幅塌腰挺臀、回眸看向後方；右側男性跪於身後身體前傾，雙手緊扶其胯部。",
  "source_hash": "<保留原值>"
}
```

### 場景 C：第一人稱 POV 或「只畫局部、不畫男方全身」
- **規則**：`roles` **只放被拍攝的女角**（不建 `boy_1`），主觀視角與前景局部詞全部留在 `base_tags_en`。
```json
{
  "id": "custom_pov_004",
  "base_tags_en": [
    "1girl", "pov", "pov hands", "pov crotch", "from behind", "from above",
    "lower body", "sex from behind"
  ],
  "roles": [
    {
      "slot": "girl_1",
      "gender": "girl",
      "center": {"x": 0.50, "y": 0.56},
      "tags_en": [
        "all fours", "arched back", "hips raised", "ass focus",
        "looking back over shoulder", "looking at viewer"
      ],
      "natural_en": "Seen from above and behind on all fours with her back arched and hips lifted toward the foreground, turning back to look up at the viewer."
    }
  ],
  "tags_en": [ "..." ],
  "natural_en": "...",
  "prompt_zh": "...",
  "source_hash": "<保留原值>"
}
```

### 場景 D：人＋道具／束縛／純觸手（道具不建 role）
- **規則**：道具（震動棒、繩索、項圈、純觸手）放進 `base_tags_en` 與受方動作詞，`roles` 只放 `girl_1`（若有操控道具的男角則加 `boy_1`）。
```json
{
  "id": "custom_toy_example",
  "base_tags_en": [
    "1girl", "solo", "full body", "from above",
    "bondage", "rope", "vibrator", "sex toy"
  ],
  "roles": [
    {
      "slot": "girl_1",
      "gender": "girl",
      "center": {"x": 0.50, "y": 0.54},
      "tags_en": [
        "lying", "on back", "bound wrists", "arms behind back",
        "spread legs", "knees bent", "arched back", "head tilted back"
      ],
      "natural_en": "Lying on her back with wrists bound behind her, knees bent and legs spread wide with back arched upward."
    }
  ],
  "tags_en": [ "..." ],
  "natural_en": "...",
  "prompt_zh": "...",
  "source_hash": "<保留原值>"
}
```

### 場景 E：人＋動物／魔物／獨立非人角色（使用 `gender: "other"`）
- **規則**：非人對象在畫面中有獨立軀幹與站位時，於 `base_tags_en` 加 `1other`，在 `roles` 最後建立 `slot: "other_1", gender: "other"`，同樣用 `source#` / `target#` 標明互動，且 `other_1.tags_en` 不可帶 `animal ` / `monster ` / `creature ` 前綴。
```json
{
  "id": "custom_other_example",
  "base_tags_en": [
    "1girl", "1other", "full body", "side view", "from behind"
  ],
  "roles": [
    {
      "slot": "girl_1",
      "gender": "girl",
      "center": {"x": 0.38, "y": 0.58},
      "tags_en": [
        "all fours", "arched back", "looking back", "target#mounting"
      ],
      "natural_en": "On all fours facing left with arched back and hips raised, glancing back."
    },
    {
      "slot": "other_1",
      "gender": "other",
      "center": {"x": 0.64, "y": 0.52},
      "tags_en": [
        "behind", "leaning forward", "source#mounting", "grabbing waist"
      ],
      "natural_en": "Positioned closely behind her on the right, leaning forward over her lower back."
    }
  ],
  "tags_en": [ "..." ],
  "natural_en": "...",
  "prompt_zh": "...",
  "source_hash": "<保留原值>"
}
```

---

## 4. 硬性驗證紅線（違反會被 `--verify` 自動攔截）

1. **`id` 與 `source_hash` 嚴禁改動**：直接原字元複製輸入分片裡的值。
2. **角色排序與人數一致**：`girl_*` → `boy_*` → `other_*`；除 POV 視角外，`base_tags_en` 的 `Ngirls` / `Nboys` / `Nothers` 必須與 `roles` 內對應性別數量一致。
3. **`roles[i].tags_en` 禁用主詞／計數前綴**：禁止以 `girl`, `boy`, `woman`, `man`, `female`, `male`, `partner`, `creature`, `monster`, `animal`, `viewer`, `left/right/first/second + 上述詞`, `1girl`, `1boy`, `1other`, `solo`, `hetero`, `ffm threesome`, `mmf threesome` 開頭。
4. **嚴禁夾帶非姿勢禁詞（全英文欄位檢查）**：
   - 禁畫風：`anime`, `3d`, `sketch`, `monochrome`, `greyscale`, `grayscale`, `lineart`, `watercolor`, `oil painting`, `photorealistic`, `realistic`, `cel shading`, `illustration`, `manga`, `comic`, `chibi`
   - 禁服裝：`bikini`, `swimsuit`, `dress`, `uniform`, `skirt`, `shirt`, `t-shirt`, `pants`, `jeans`, `shorts`, `jacket`, `coat`, `hoodie`, `sweater`, `lingerie`, `underwear`, `stockings`, `thighhighs`
   - 禁髮色瞳色：`long hair`, `short hair`, `blonde/black/white/brown/red/blue/pink/silver hair`, `blue/red/green/brown/yellow/purple/golden eyes`
   - 禁背景環境：`background`, `sunlight`, `moonlight`, `outdoors`, `indoors`, `scenery`, `landscape`, `sky`, `clouds`, `forest`, `beach`, `classroom`, `bedroom`, `street`, `cityscape`
   - 額外切勿寫入：胸圍大小（`large/medium/small breasts`）、指甲油（`toenail polish`）、表情汁液（交給工廠公式階梯控制）。

---

## 5. 本機分片位置與收尾合併指令（直接開跑）

- **工作目錄**：`/Volumes/SSD/ai-pose-gallery-zh`
- **每筆輸入的圖片路徑**：`/Volumes/SSD/ai-pose-gallery-zh/{orig_path}`（高清原圖）或 `{thumb_path}`（縮圖）
- **優先處理（任務 1：`custom` 庫，共 338 筆，4 個分片）**：
  - `tool/.tmp/v2_shards_custom/shard_001.json` ～ `shard_004.json`
- **接續處理（任務 2：`base` 庫，共 1,439 筆，15 個分片）**：
  - `tool/.tmp/v2_shards_base/shard_001.json` ～ `shard_015.json`

### 處理方式
1. 每次讀取 1 個分片（建議每批 5～10 張圖看圖標註），將改寫後的 JSON Array 直接覆寫回該分片檔（或輸出至同結構檔案）。
2. 每完成一批立刻跑下方指令合併並驗證（任何 1 筆違規會直接報出行號與錯誤，不覆寫原檔）：

```bash
cd /Volumes/SSD/ai-pose-gallery-zh

# 合併 Custom 分片（338 筆）
python3 scripts/pose_prompts_tool.py --target custom --merge "tool/.tmp/v2_shards_custom/shard_*.json"

# 合併 Base 分片（1,439 筆）
python3 scripts/pose_prompts_tool.py --target base --merge "tool/.tmp/v2_shards_base/shard_*.json"

# 全庫驗收
python3 scripts/pose_prompts_tool.py --verify --target all
python3 -m unittest discover -s scripts -p "test_*.py" -v
```
