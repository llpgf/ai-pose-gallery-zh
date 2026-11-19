# 任務指令書：從零撰寫獨立 NSFW 姿勢模板庫（NovelAI V4.5 / V5 真解耦架構）

> **給外部無審查 LLM 的直接執行指令**：
> 1. **不要修改原本的 `data/pose-zh.json`、`data/pose-prompts-zh.json` 或 `data/pose-custom-zh.json`**，也**不要看舊圖、不要寫研究報告、不要反問問題**。
> 2. 請直接從零撰寫一份全新的、乾淨好審核的 **獨立 NSFW 姿勢模板庫**，按下方 **10 大類共 140 條** 分成 **10 個獨立 JSON 檔** 輸出到 `/Volumes/SSD/ai-pose-gallery-zh/data/nsfw_library/`。
> 3. 本任務採用 **「動態函數式真解耦」**：每條姿勢是一個帶**抽象功能槽位（`slots`）**與**配對分派表（`pairing_tags`）**的模板，由下游工廠程式在生圖前代入實際角色卡，動態算出人數、性別與器官結合標籤。

---

## 1. 核心寫法原則（NovelAI V4.5 / V5 母圖連抽黃金標準）

這 140 條模板會直接送進工廠的三明治分鏡系統，當作 **NAI V4.5 / V5 母圖連抽的構圖骨架**：
1. **標準 Danbooru 詞 ＋ 精簡幾何鎖定短語**：
   - 不要只寫 3 個籠統單字（鎖不住構圖），也不要寫冗長散文。
   - **正確示範**（NAI V5 實測連抽 16 張不跑位）：`lower body, from below, front view, lying, on back, knees bent, both feet raised toward camera, soles facing viewer, toes curled slightly, close-up`。
2. **無審查、直白還原體位與接觸點**：
   - 針對口、乳、手、足、前後插入、道具、束縛、觸手與非人互動，直接寫出 NAI 懂的精確體位與解剖接觸詞，禁止使用「親密互動、靠近、擁抱」等含糊敷衍詞。
3. **嚴格禁止在模板主體寫死「人數／性別配對詞」與「非姿勢屬性」**：
   - **`camera_tags` 與 `shared_pose_tags` 嚴禁寫死 `1girl`, `1boy`, `2girls`, `2boys`, `1other`, `solo`, `hetero`, `yuri`, `yaoi`**（這些會由工廠組裝函數依使用者代入的角色卡動態填入，或由該條的 `pairing_tags` 指定）。
   - **全篇嚴禁夾帶**：胸圍大小（`large/medium/small breasts`）、服裝與內衣（`bikini`, `dress`, `skirt`, `shirt`, `pants`, `underwear`, `no underwear`, `stockings`, `thighhighs`）、指甲油（`toenail polish`）、髮長髮色瞳色、畫風（`anime`, `realistic`）、汁液高潮臉（第 10 類高潮骨架除外，一般體位的潮紅/汗水/精液由下游階梯控制）、背景環境（`bedroom`, `outdoors`, `sunlight`）。
4. **善用 NAI V4.5 / V5 多角色互動語法**：
   - 在 `slots.<slot>.tags` 裡用 `source#<動作>`（發起方）、`target#<動作>`（承受方）、`mutual#<動作>`（雙向），並估算角色軀幹質心 `center: {"x": 0.00..1.00, "y": 0.00..1.00}`。
   - `slots.<slot>.tags` **嚴禁帶角色主詞前綴**（禁止 `girl ...`, `boy ...`, `woman ...`, `man ...`, `animal ...`, `monster ...`, `creature ...`, `left boy ...`）。

---

## 2. 物理結構解耦的 3 種模板寫法（解決「能不能互換男女」問題）

不要硬把所有體位都偽裝成男女通用，請依物理結構誠實歸入以下 3 種模式之一：

### 模式 A：`geometry_dispatch`（骨架相同、結合媒介可替換：後背位／騎乘位／正常位／立位／側臥）
- 幾何骨架只寫抽象槽位（如 `receiver` / `giver` 或 `top` / `bottom`），不含性別詞。
- 透過 **`pairing_tags`** 宣告不同角色組合代入時要補上的配對與器官詞（含 `female_male`, `female_futa`, `female_female`(自動改插 `strap-on`), `female_other`, `pov_part`(抽掉 `giver` 角色卡只留 POV/局部)）。

### 模式 B：`organ_bound`（物理強綁定特定器官：乳交／口交／足交／舔陰／磨豆腐）
- **不要硬湊不相容配對！** 每個槽位用 **`requires`** 誠實標明物理前提：
  - `"any"`（不限性別，如 `cunnilingus` 舔陰的施與方，男女皆可擔任！）
  - `"female"`（女性角色）
  - `"breasts"`（需具胸部，如 `paizuri` 乳交施與方）
  - `"pussy"`（需具女性下體，如 `cunnilingus` 承受方、`tribadism` 雙槽位）
  - `"penis_or_dildo"`（需具陽具／扶他／穿戴或獨立假陽具，如 `fellatio`, `paizuri`, `footjob`, `handjob` 承受方）
  - `"penis_or_strapon_or_nonhuman"`（可作為插入方：男角／扶他／穿戴假陽具女角／非人）
  - `"nonhuman"`（動物／魔物／獨立非人角色）
- **`pairing_tags` 只列出物理上合法成立的配對鍵**；沒列出的配對工廠會自動判定不相容。

### 模式 C：`solo_or_pov`（單人特寫／道具束縛／純觸手／第一人稱 POV）
- 只有單一槽位 `subject`（`center: {"x": 0.50, "y": 0.50}`）。道具（震動棒、繩索、純觸手）與 POV 詞全部寫進 `shared_pose_tags` / `camera_tags`，不建立多餘角色槽位。

---

## 3. 朝向、左右手腳與視線規範（支援外部一鍵鏡像與視線熱插拔）

為避免同一姿勢因「朝左／朝右」膨脹成 2 倍條目，同時讓下游工廠能一鍵翻轉左右並熱插拔視線，所有條目必須遵守以下 3 條規則：

### 3.1 身體大朝向：只寫單邊基準 ＋ `default_facing`（絕不寫左右雙胞胎條目）
- 每筆必帶：
  - `"default_facing"`: 限填 `"left"` | `"right"` | `"front"` | `"back"` | `"three_quarter_left"` | `"three_quarter_right"`
  - `"mirrorable"`: `true`（僅在極少數不可鏡像的特殊構圖設 `false`）
- **不要為了朝左／朝右重複寫兩條**。側面與斜側雙人體位統一優先採用「承受方居左（`center.x < 0.50`）、施與方居右（`center.x > 0.50`）」的單邊基準；**`center.x` 與方向詞（`facing left` / `facing right`）必須嚴格一致**，絕不可座標放左邊、文字寫朝右。
- 當使用者在外部點擊 `↔ 鏡像翻轉` 時，工廠程式會自動將每個槽位的 `center.x` 翻轉為 `1.0 - x`，並將 `left ↔ right` 相關詞彙確定性互換。

### 3.2 左右手腳不對稱擺位：寫在姿勢庫裡，用標準可鏡像詞彙
- 不對稱肢體（例如左手撐地、右手扶牆，左腿抬高、右腳著地）**必須直接寫死在姿勢庫對應槽位的 `tags` 與 `natural_en` 中**，以鎖定母圖連抽幾何；對稱動作則直接寫 `both hands ...` / `both legs ...`。
- 務必使用明確、可程式鏡像互換的標準英文方位詞（`facing left`, `facing right`, `left hand ...`, `right hand ...`, `left arm ...`, `right arm ...`, `left leg ...`, `right leg ...`, `left knee ...`, `right foot ...`, `over left shoulder`），**禁止寫成模糊詞**（禁止 `one hand ... the other hand ...` 若該手具明確方向性、或 `this leg`）。

### 3.3 臉部朝向與視線：姿勢庫寫預設，獨立列 tag 供外部熱插拔覆寫
- 每個角色槽位請寫入該體位最自然的預設視線，且**必須當成獨立短 tag 放在該槽位 `tags` 中**（不要和四肢動作黏成一長句），優先使用以下標準詞集：
  - `looking at viewer`（直視鏡頭）
  - `looking at another`、`eye contact`（與對方對視）
  - `looking back`、`looking back over shoulder`（回眸）
  - `looking away`、`head turned away`（羞恥別過頭）
  - `looking up`、`looking up at viewer`（仰視）
  - `looking down`（低頭）
  - `head tilted back`、`closed eyes`（仰頭／閉眼）
- 當使用者在外部點選「視線熱插拔晶片」時，工廠會自動剝離這些標準視線 tag 並替換為指定視線，完全不破壞原本的身體骨架。

---

## 4. 統一 JSON Schema 與 4 個標準範例

每個輸出檔為一個 **純 JSON Array**，每筆元素結構如下：

### 範例 1：模式 A（後背位・真解耦骨架 ＋ 多配對分派 ＋ 基準朝向與可鏡像左右手）
```json
{
  "id": "pos_doggystyle_allfours_side",
  "category": "04_main_positions",
  "subcategory": "後背位",
  "title_zh": "四肢著地後背位・側面扶胯",
  "pattern": "geometry_dispatch",
  "default_canvas": "3:4",
  "default_facing": "left",
  "mirrorable": true,
  "camera_tags": ["full body", "side view", "from side"],
  "shared_pose_tags": ["doggystyle", "sex from behind"],
  "slots": {
    "receiver": {
      "label_zh": "承受方（前方四肢著地）",
      "requires": "any",
      "center": {"x": 0.40, "y": 0.58},
      "tags": [
        "facing left", "all fours", "arched back", "hips raised",
        "left hand braced flat", "right arm bent supporting chest",
        "target#grabbing hips", "looking back over shoulder"
      ],
      "natural_en": "On all fours facing left with deeply arched back and raised hips, left hand braced flat and right arm slightly bent for support while looking back over the left shoulder."
    },
    "giver": {
      "label_zh": "施與方（後方跪姿扶胯）",
      "requires": "penis_or_strapon_or_nonhuman",
      "center": {"x": 0.65, "y": 0.50},
      "tags": [
        "facing left", "kneeling", "behind", "leaning forward",
        "source#grabbing hips", "hands on another's hips",
        "looking down"
      ],
      "natural_en": "Kneeling directly behind on the right facing left, leaning forward over the partner's lower back while firmly gripping both hips and looking down."
    }
  },
  "pairing_tags": {
    "female_male":   {"base": ["hetero", "sex", "vaginal"], "giver": ["penis", "thrusting"]},
    "female_futa":   {"base": ["yuri", "futanari", "sex", "vaginal"], "giver": ["futanari", "penis"]},
    "female_female": {"base": ["yuri", "strap-on", "sex", "vaginal"], "giver": ["strap-on", "dildo"]},
    "female_other":  {"base": ["sex", "vaginal", "interspecies"], "giver": ["source#mounting"]},
    "pov_part":      {"drop_slot": "giver", "base": ["pov", "pov hands", "pov crotch", "sex", "vaginal"]}
  },
  "natural_en": "Full-body side view of a doggystyle pose facing left: the receiver is on all fours on the left with arched back and raised hips looking back over the left shoulder, while the giver kneels behind on the right leaning forward and gripping the receiver's hips.",
  "prompt_zh": "側面全身後背位（基準朝左）：承受方居左四肢著地大幅塌腰挺臀、左手平撐並向左肩回眸；施與方居右跪於身後前傾雙手緊扶其胯部。"
}
```

### 範例 2：模式 B（舔陰 `cunnilingus`・施與方男女通用，承受方限 `pussy`）
```json
{
  "id": "yuri_cunnilingus_kneeling_front",
  "category": "05_yuri_cunnilingus",
  "subcategory": "舔陰侍奉",
  "title_zh": "仰躺張腿舔陰・低角度正面",
  "pattern": "organ_bound",
  "default_canvas": "3:4",
  "default_facing": "front",
  "mirrorable": true,
  "camera_tags": ["cowboy shot", "front view", "from below"],
  "shared_pose_tags": ["cunnilingus", "oral"],
  "slots": {
    "receiver": {
      "label_zh": "承受方（仰躺張腿）",
      "requires": "pussy",
      "center": {"x": 0.50, "y": 0.40},
      "tags": [
        "lying", "on back", "spread legs", "knees bent",
        "arched back", "left hand on another's head", "right hand gripping surface",
        "target#cunnilingus", "head tilted back", "looking down"
      ],
      "natural_en": "Lying back facing front with legs spread wide and knees bent, back arching slightly with the left hand resting on the partner's head and right hand gripping the surface."
    },
    "giver": {
      "label_zh": "施與方（腿間俯身舔舐）",
      "requires": "any",
      "center": {"x": 0.50, "y": 0.68},
      "tags": [
        "kneeling", "between legs", "leaning forward",
        "tongue out", "licking", "hands on another's thighs",
        "source#cunnilingus", "looking up"
      ],
      "natural_en": "Kneeling between the partner's parted thighs, leaning in closely while gripping both thighs and looking up."
    }
  },
  "pairing_tags": {
    "female_male":   {"base": ["hetero"]},
    "female_female": {"base": ["yuri"]}
  },
  "natural_en": "Low-angle front view of cunnilingus: the receiver lies back with spread legs resting the left hand on the giver's head, while the giver kneels between the thighs holding them open.",
  "prompt_zh": "低角度正面舔陰：承受方仰躺張腿微拱腰、左手扶對方頭部右手抓撐；施與方跪於腿間俯身扶住大腿貼近舔舐（男女皆可擔任施與方）。"
}
```

### 範例 3：模式 C（單人／道具／POV・單槽位 `subject`）
```json
{
  "id": "prop_bondage_vibrator_above",
  "category": "08_props_bondage",
  "subcategory": "束縛與道具",
  "title_zh": "反綁仰躺震動棒・俯視全身",
  "pattern": "solo_or_pov",
  "default_canvas": "3:4",
  "default_facing": "front",
  "mirrorable": true,
  "camera_tags": ["full body", "from above", "high angle"],
  "shared_pose_tags": ["bondage", "rope", "tied up", "vibrator", "sex toy"],
  "slots": {
    "subject": {
      "label_zh": "主體（被束縛仰躺）",
      "requires": "female",
      "center": {"x": 0.50, "y": 0.52},
      "tags": [
        "lying", "on back", "bound wrists", "arms behind back",
        "spread legs", "knees bent", "arched back",
        "head tilted back", "looking up"
      ],
      "natural_en": "Lying on her back with wrists bound behind her torso, knees bent and legs spread apart while arching her back upward."
    }
  },
  "pairing_tags": {
    "solo": {"base": ["solo", "masturbation"]}
  },
  "natural_en": "High-angle overhead full-body view of a bound female lying on her back with wrists tied behind her and legs spread apart while using a vibrator.",
  "prompt_zh": "高角度俯視全身：雙腕反綁仰躺，屈膝大幅張腿並拱起腰背，搭配震動棒刺激。"
}
```

### 範例 4：多人組合（`group`・1女2男前後串燒 / 2女1男協力）
- `slots` 依位置命名（如 `center_subject`, `front_partner`, `behind_partner` 或 `giver_left`, `giver_right`, `receiver`），`pairing_tags` 對應 `"mmf"` 或 `"ffm"`，每人標明不重疊的 `center` 座標、明確朝向詞與 `source#`/`target#`。

---

## 5. 輸出目錄與 10 大分類配額（合計 140 條）

請在 `/Volumes/SSD/ai-pose-gallery-zh/data/nsfw_library/` 依序建立以下 **10 個獨立 JSON 檔案**（每檔皆為純 JSON Array，禁止多餘註解與 Markdown 包裹）：

| 檔案名稱 | 大分類 | 目標條數 | 必須涵蓋的細分子分類與構圖變化 |
| :--- | :--- | :---: | :--- |
| `01_solo_closeup.json` | 單人誘惑與局部特寫 | **15 條** | 撩人仰躺/跪坐/翹臀開腿(4)、足部與腳底特寫(3)、胸部特寫(2)、臀部特寫(2)、隱私展示特寫(2)、口舌特寫(2) |
| `02_foreplay.json` | 前戲與身體愛撫 | **15 條** | 背後環抱揉胸(3)、壁咚與壓制親吻(3)、耳頸親吻與撫摸腰腿(3)、掀衣/拉扯互動動作(3)、指交與下體愛撫(3) |
| `03_service_oral_chest_hand_foot.json` | 侍奉類（口/乳/手/足） | **22 條** | 口部侍奉陽具 `fellatio`(6：跪姿仰望/深喉/側舔/雙手扶持/仰躺倒口/俯視)、乳交 `paizuri`(5：正面夾擊/俯視/跨坐/側乳/舌舔)、手部侍奉 `handjob`(4)、足部侍奉 `footjob`(7：雙腳夾擊/單腳踩踏/仰視腳底/桌下隱秘/跪姿反足交/從後足交/臉近距離) — *全數含 `pov_part` 變體* |
| `04_main_positions.json` | 雙人插入主體位 | **26 條** | 後背位(6：四肢側面/四肢斜後/俯趴壓背/站立扶牆/拉臂後仰/床緣趴伏)、正常位(6：傳統俯視/抬腿壓肩/十指交扣/深壓折疊/單腿高舉/床邊懸腿)、騎乘位(6：正向俯視/正向側斜/反向背騎/蹲姿騎乘/前傾貼胸/後仰挺腰)、立位與抱起(4：面對面抱腿/壁貼抬單腿/火車便當/背後立位)、側臥與坐位(4：湯匙側臥/面對面側臥/面對面坐大腿/背向坐懷) — *全數採模式 A 真解耦* |
| `05_yuri_cunnilingus.json` | 女女與下體專屬 | **10 條** | 舔陰 `cunnilingus`(4：仰躺低角/跨坐顏面騎乘/69互舔/站立抬腿，支援 `female_male` 與 `female_female`)、雙女磨豆腐 `tribadism`(3)、雙女互相愛撫揉胸與深吻(3) |
| `06_pov_special_cameras.json` | POV 與特殊漫畫鏡頭 | **14 條** | 第一人稱 POV(6：俯視跪姿/俯視跨坐/POV伸手揉胸/POV後入扶臀/POV正常位抬腿/POV仰視踩踏)、偷窺遮擋鏡(4：門縫窺視/窗框前景/桌下隱秘/監視器俯拍)、特殊透視鏡(4：鏡面雙向倒影/魚眼近迫/跨下極端蟲瞰仰視/荷蘭角動態張力) |
| `07_group_ffm_mmf.json` | 多人組合（FFM/MMF） | **12 條** | 2女1男 FFM(6：雙人口交/雙人乳交/雙人足交/一上一下騎乘+顏騎/雙女左右夾吻/輪流侍奉)、1女2男 MMF(4：四肢前後串燒/站立前後夾擊/雙重插入/左右雙手口侍奉)、多人群聚包圍(2) |
| `08_props_bondage.json` | 人＋道具與束縛調教 | **10 條** | 單人自慰與震動棒/假陽具(3)、繩縛/手腳反綁/M字開腿綁縛(4)、蒙眼/項圈牽引/口球與對方持道具調教(3) |
| `09_tentacles_nonhuman.json` | 觸手與非人／動物 | **8 條** | 純觸手拘束/懸空纏繞/多點入侵(4：採模式 C，不建 role)、獨立非人/魔物/動物互動(4：採模式 A/B，槽位 `requires: "nonhuman"`，含後背壓制/俯趴/侍奉/被包圍) |
| `10_climax_afterglow.json` | 高潮瞬間與事後餘韻 | **8 條** | 高潮拱背抽搐與體內射精瞬間(2)、跪姿仰頭顏射與射胸構圖(2)、事後溢出局部特寫(2)、事後癱軟仰躺與雙人依偎餘韻(2) |
| **總計** | **10 個獨立 JSON 檔** | **140 條** | 每檔 8～26 條，逐檔獨立好審核 |

---

## 6. 完成後自檢腳本（確認 JSON 結構、朝向欄位、總數與零違規）

外部 AI 寫完 10 個 JSON 檔後，執行下方腳本驗證結構、朝向欄位與禁詞零錯誤即可交付審核：

```bash
python3 - <<'PY'
import glob, json, re

FILES = sorted(glob.glob("/Volumes/SSD/ai-pose-gallery-zh/data/nsfw_library/*.json"))
VALID_FACING = {"left", "right", "front", "back", "three_quarter_left", "three_quarter_right"}
BANNED = re.compile(
    r"(?<!\w)(anime|3d|sketch|monochrome|realistic|bikini|swimsuit|dress|uniform|skirt|shirt|"
    r"pants|jeans|shorts|jacket|coat|hoodie|sweater|lingerie|underwear|no underwear|stockings|"
    r"thighhighs|toenail polish|large breasts|medium breasts|small breasts|"
    r"(?:long|short|blonde|black|white|brown|red|blue|pink|silver) hair|"
    r"(?:blue|red|green|brown|yellow|purple|golden) eyes|"
    r"background|sunlight|moonlight|outdoors|indoors|bedroom|classroom|street)(?!\w)", re.I
)
HARDCODED_COUNT = re.compile(r"^(?:\d+\s*(?:girl|girls|boy|boys|other|others|people)|solo|hetero|yuri|yaoi)$", re.I)
ROLE_PREFIX = re.compile(r"^(?:(?:left|right|first|second|third)\s+)?(?:girl|boy|woman|man|animal|monster|creature|viewer|partner)\b", re.I)

total, errors, ids = 0, [], set()
for fp in FILES:
    rows = json.loads(open(fp, encoding="utf-8").read())
    assert isinstance(rows, list), f"{fp}: root must be list"
    print(f"{fp.split('/')[-1]}: {len(rows)} entries")
    for r in rows:
        total += 1
        eid = r.get("id", "")
        if not eid or eid in ids:
            errors.append(f"{fp}: duplicate/empty id {eid!r}")
        ids.add(eid)
        if r.get("default_facing") not in VALID_FACING:
            errors.append(f"[{eid}] invalid/missing default_facing: {r.get('default_facing')!r}")
        if not isinstance(r.get("mirrorable"), bool):
            errors.append(f"[{eid}] missing/non-bool mirrorable")
        for t in r.get("camera_tags", []) + r.get("shared_pose_tags", []):
            if HARDCODED_COUNT.match(t.strip()):
                errors.append(f"[{eid}] hardcoded count/gender in shared/camera tags: {t!r}")
            if BANNED.search(t):
                errors.append(f"[{eid}] banned word in shared/camera tags: {t!r}")
        slots = r.get("slots", {})
        if not isinstance(slots, dict) or not slots:
            errors.append(f"[{eid}] missing slots")
        for sname, s in slots.items():
            c = s.get("center", {})
            if not (isinstance(c, dict) and 0.0 <= float(c.get("x", -1)) <= 1.0 and 0.0 <= float(c.get("y", -1)) <= 1.0):
                errors.append(f"[{eid}] slot {sname} invalid center: {c!r}")
            for t in s.get("tags", []):
                if ROLE_PREFIX.match(t.strip()):
                    errors.append(f"[{eid}] slot {sname} has forbidden subject prefix: {t!r}")
                if BANNED.search(t):
                    errors.append(f"[{eid}] slot {sname} banned word: {t!r}")
            if BANNED.search(s.get("natural_en", "")):
                errors.append(f"[{eid}] slot {sname} natural_en banned word")
        if not r.get("pairing_tags"):
            errors.append(f"[{eid}] missing pairing_tags")

print(f"\nTotal files: {len(FILES)}, Total entries: {total}, Errors: {len(errors)}")
for e in errors[:30]:
    print("  -", e)
PY
```
