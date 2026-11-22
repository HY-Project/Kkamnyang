# 깜냥 모딩 안내

깜냥은 MOTP(HY Project #A-01)와 같은 모드 체계를 쓴다. 지금 게임 버전은 **`0.8.0`**이다.

> 모드는 **설치판(HY 런처·zip)에서만** 읽힌다. 웹 버전에서는 모드를 쓸 수 없다.

---

## 1. 어디에 두나

게임 실행 파일 옆의 `mods/` 폴더(없으면 만든다)에 **모드 하나당 폴더 하나**, 또는 같은 내용의 `.zip` 하나를 둔다.

```
Kkamnyang/
  Kkamnyang.exe
  res/
  mods/
    my-mod/
      mod.json
      data/characters.json
      res/sprites/kkamnyang_human.png
```

`modpack.json`이 맨 위에 있고 `mods/` 아래에 모드 여럿이 든 zip은 **모드팩**으로 읽힌다.

## 2. `mod.json`

```json
{
  "id": "your.name.my-mod",
  "name": "내 모드",
  "version": "1.0.0",
  "gameVersion": "0.8.x",
  "author": "이름",
  "description": "무엇을 바꾸는가",
  "entry": "main.js",
  "capabilities": ["events", "mixin"]
}
```

- `id`는 `[A-Za-z0-9._-]`. 같은 `id`가 둘이면 둘 다 꺼진다.
- `gameVersion` — `"0.8.x"` · `{ "min": "0.7.0", "max": "0.8.0" }` · `"*"`. 맞지 않으면 경고하고 그래도 싣는다.
- `entry`·`capabilities`는 코드를 싣는 Advanced 모드만(8절).

## 3. 순서와 충돌

설정 → 일반 → **모드 관리**에서 켜고 끄고 순서를 바꾼다. **위에 있는 모드가 아래를 덮는다.** 기존 것을 덮거나 코드를 싣는 모드에는
⚠ 표시가 붙는다. 모드 때문에 게임이 안 켜지면 세 번째 시도부터 **안전 모드**(모든 모드 끔)로 저절로 뜬다. **엄격 모드**를 켜면
모드 하나라도 실패할 때 게임을 아예 시작하지 않는다(모드팩을 만들 때 권장).

## 4. 덮어쓸 수 있는 것

| 폴더 | 무엇 | 방식 |
|---|---|---|
| `res/` | 그림·소리 — `res/sprites/*.png|json`, `res/audio/sfx/*.ogg`, `res/audio/bgm/*.ogg`, `res/ui/*` | 같은 경로의 파일을 갈아 끼운다 |
| `data/characters.json` | 캐릭터 수치·스킬 | 깊은 병합(아래) |
| `data/enemies.json` | 적 수치 | 깊은 병합 |
| `data/bosses.json` | 보스 — 체력·단계·방패·패턴 조율값 | 깊은 병합 |
| `data/items.json` | 아이템 표 | 깊은 병합 |
| `data/physics.json` | 물리 조율값(속도 단계·중력·점프·그래플 …) | 깊은 병합 |
| `data/progression.json` | 품계·레벨 곡선, 스킬 강화 비용, 보상 | 깊은 병합 |
| `data/keymap.json` | 기본 키 배치 | 깊은 병합 |
| `data/bgm.json` · `data/sfx.json` | 선곡 규칙·음량 | 깊은 병합 |
| `maps/*.json` | 스테이지 맵(같은 `id`면 덮고, 새 `id`면 더한다) | 통째로 |
| `fonts/` | 글꼴 | `mod.json`의 `fonts` |
| `locales/<언어>/<이름공간>.json` | 글(이름공간: common · story · stages · characters · items) | 깊은 병합 |

**깊은 병합** — 객체는 칸마다 섞이고, 배열은 통째로 바뀐다. `$`로 시작하는 맨 위 칸(`$readme` 등)은 건드리지 못한다.

예 — 깜냥의 레이저를 네 발 점사로:

```json
{ "kkamnyang": { "laser": { "burst": 4 } } }
```

스킬 칸(`q`·`e`·`r`)의 `kind`가 **공용 실행기**를 고른다 — 캐릭터마다 코드가 따로 있지 않으니, 어느 동료에게든 어느 실행기든 붙일 수 있다.
비워 둔 칸은 실행기마다의 기본값을 쓴다. 연계기 `tag`(교대해 들어올 때 저절로 나가는 것)는 지금 `laser`뿐이다.

| `kind` | 하는 일 | 주로 쓰는 칸 | 지금 쓰는 동료 |
|---|---|---|---|
| `dash` | 무적 돌진, 지나는 적에게 피해 | `speed` `duration` `damage` | 깜냥 Q · 볕뉘 E |
| `recall` | 처음 누르면 자리를 새기고, 다시 누르면 되돌아오며 회복 | `charges` `heal` | 깜냥 E |
| `volley` | 분신이 가까운 적에게 레이저를 일제히 | `clones` `shots` `interval` `damage` `range` | 깜냥 R |
| `laser` | 큰 레이저 한 줄기 | `damage` `pierce` `range` `width` | 볕뉘 Q·R |
| `grenade` | 조준한 쪽으로 던지는 포탄 — 적·벽에 닿거나 `fuse`초 뒤 터진다 | `speed` `radius` `damage` `fuse` `gravity` | 희나리 Q |
| `rocketJump` | 발밑을 터뜨려 위로 솟구친다(공중에서는 땅에 닿기 전까지 한 번) | `speed` `push` `radius` `damage` | 희나리 E |
| `cannon` | 큰 포탄, 쏜 몸은 뒤로 밀린다(곧게 날리려면 `gravity: 0`) | `speed` `radius` `damage` `push` `gravity` | 희나리 R |
| `stealth` | `duration`초 동안 적의 눈에 안 보인다(몸에 닿으면 여전히 다친다) | `duration` | 해미 Q |
| `waveDash` | 조준한 쪽(여덟 방향)으로 무적 돌진, 물속에서는 더 빠르고 길다 | `speed` `duration` `damage` | 해미 E |
| `tsunami` | 땅·수면을 타고 가는 큰 파도 — 적을 밀고 적 탄을 지운다 | `speed` `duration` `damage` `height` | 해미 R |
| `guard` | `duration`초 동안 앞에 방패 — 보는 쪽에서 오는 것만 막는다 | `duration` | 엄장 Q |
| `slam` | 내려찍기 — 닿은 자리의 충격과 양쪽으로 가는 충격파 둘 | `height` `speed` `damage` `radius` `range` | 엄장 E |
| `bolt` | 하늘에서 번개 기둥을 `shots`번 — 장승의 정면 방패를 넘고 봉수 화로도 켠다 | `shots` `interval` `range` `width` `damage` | 엄장 R |

모든 칸에 `cooldown`·`energyCost`가 있다. 폭발(`grenade`·`rocketJump`·`cannon`·`slam`)은 부서지는 벽을 허물고, 쏜 사람은 다치지 않는다.

예 — 질주 3단을 더 빠르게:

```json
{ "tiers": [ { "after": 0, "max": 150 }, { "after": 0.45, "max": 260 }, { "after": 1.0, "max": 380 }, { "after": 1.9, "max": 520 } ] }
```

예 — 해태 대장의 2단계에 낙하물을 더하고, 돌진 예고를 길게:

```json
{ "haetae-chief": {
  "phases": [
    { "hpBelow": 1, "shield": "red", "shieldHp": 10, "gap": 0.9, "patterns": ["charge", "leap", "summon"] },
    { "hpBelow": 0.5, "shield": "red", "shieldHp": 8, "gap": 0.6, "patterns": ["charge", "leap", "spread", "rain"] }
  ],
  "patterns": { "charge": { "tell": 1.0 }, "rain": { "count": 8, "interval": 0.15 } }
} }
```

보스 패턴은 공용 실행기 아홉이다 — 보스마다 코드를 두지 않고 데이터가 고른다(`patterns`의 키가 이름, `kind`를 비우면 이름이 곧 종류).
모든 패턴 앞에는 예고(`tell`초)가 있고 뒤에는 숨 고르기(`recover`초)가 있다.

| 종류 | 하는 일 | 주로 쓰는 칸 |
|---|---|---|
| `charge` | 예고 때 정한 쪽으로 수평 돌진, 벽에 부딪히면 쿵 | `speed` `duration` |
| `leap` | 예고 때 표시한 자리로 뛰어 내려찍고, 땅을 타는 충격파 둘 | `height` `speed` `damage` |
| `spread` | 플레이어 쪽 부채꼴 탄 | `count` `arc` `speed` `volleys` `interval` |
| `summon` | 보스 양옆에 잡몹(살아 있는 부하는 `max`까지, 보스가 쓰러지면 함께) | `enemy` `count` `max` |
| `sweep` | 눈에서 광선을 `from`°→`to`°로 휩쓴다(0° 앞 수평, 90° 바로 아래). 예고 동안 선이 깜빡인다 | `from` `to` `duration` `range` |
| `rain` | 아레나 천장에서 낙하물 — 떨어질 자리가 예고 때 바닥에 표시된다 | `count` `interval` |
| `wave` | 보스 앞 바닥(물 위의 보스는 수면)에서 플레이어 쪽으로 땅·수면을 타는 물결을 `count`줄 | `count` `interval` `speed` `height` |
| `lob` | 플레이어 둘레(`arc` px 안)에 포물선 포탄 `count`발 — 한 발은 발밑. `volleys`번 되풀이 | `count` `arc` `volleys` `interval` |
| `bolt` | 하늘에서 폭 `width`의 번개 기둥이 차례로 — 칠 자리가 예고 때 표시된다. 위에서 치므로 방패로 못 막는다 | `count` `interval` `width` |

모든 패턴에 `tell`·`recover`·`damage`가 있다. `sweep`은 `arm`(예고선이 깜빡이는 시간)과 `endHeight`(0이 아니면 끝 각도를 플레이어 쪽
그 높이로 맞춘다)도 받는다.

보스 표의 칸:

| 칸 | 뜻 |
|---|---|
| `hp` · `w` · `h` · `speed` · `contactDamage` · `xp` | 체력·몸 크기·패턴 사이에 걷는 속도·몸통 피해·경험 |
| `element` | 보스 자신의 색 — 탄·광선·체력 막대의 색(방패 색은 단계가 정한다) |
| `sheet` | 그림 시트 이름(6절의 보스 애니메이션) |
| `floats` | 물 위에 뜬 보스 — 중력 없이 제 높이에서 출렁이고 걷지 않는다. `charge`·`leap`은 쓰지 않는다 |
| `tellSfx` | 예고음을 바꾼다(효과음 이름, 없으면 공용 예고음) |
| `arena.delay` | 문이 닫히고 첫 패턴까지(초) — 등장 연출의 자리 |
| `phases[]` | `hpBelow` · `shield` · `shieldHp`(0이면 체력의 1/4) · `patterns` · `gap`(패턴 사이 걷는 시간) · `read` |

단계(`phases`)는 체력 비율이 `hpBelow` 아래로 내려가면 들어서고, 들어설 때 포효하며 그 단계의 방패(`shield`·`shieldHp`)를 새로 두른다.
`read: true`인 단계는 방패 색을 **읽는다** — 들어서는 순간(첫 단계는 문이 닫히는 순간) 지금 싸우는 동료가 못 깨고 편성의 다른 동료는
깨는 색을 고른다. 이때 `shield`는 문이 닫히기 전에 보이는 색일 뿐이다(종장 여의).
패턴 고르기의 난수는 씨앗이 고정이라 같은 입력이면 같은 싸움이 된다(주행 기록 회귀가 보스 판에서도 돈다).

## 5. 스테이지 맵

```json
{
  "id": "my-1",
  "chapter": 1,
  "titleKey": "stages:my-1.title",
  "parTimeMs": 60000,
  "startForm": "human",
  "layout": ["..........", "S.......G.", "##########"],
  "legend": {
    "#": { "solid": "full", "decor": "roof" },
    "S": { "marker": "spawn" },
    "G": { "marker": "goal" }
  },
  "entities": [ { "type": "haetae-bot", "x": 5, "y": 1, "params": { "patrol": 3 } } ]
}
```

- **ASCII 격자가 정본이다.** 칸은 16px. 범례 키는 글자 하나.
- 범례 칸 하나는 아래 칸들을 섞어 적는다(예: `{ "solid": "oneway", "hazard": "shock", "decor": "wire" }` = 전깃줄). 모르는 칸은 무시한다.

| 칸 | 값 | 뜻 |
|---|---|---|
| `solid` | `full` | 단단한 칸 |
| | `oneway` | 위에서만 서는 발판(아래에서 뚫고 오른다) |
| | `catflap` | 고양이 틈 — 고양이형만 지나가고 사람형·적에게는 벽 |
| | `crumble` | 밟고 잠시 뒤 무너지고, 얼마 뒤 돌아온다(`physics.json` `crumbleDelay`·`crumbleRestore`) |
| | `mirror` | 거울 — 몸·탄에는 벽, 레이저는 대각선에서 꺾인다(기울기 `reflect`: `/` · `\`) |
| | `breakable` | 폭발로만 영영 부서지는 벽 |
| | `gate` | 봉수 문 — 같은 `gate` 이름의 봉수 화로(`switch`)가 **모두** 켜진 동안 열린다 |
| | `named` | 백패 칸 — 처음엔 빈 칸, 같은 `gate` 이름의 이름 패(`name-tablet`)를 주우면 단단해진다 |
| `hazard` | `spike` | 가시 |
| | `shock` | 감전 — `cycle`이 있으면 그 주기로 켜지고 꺼진다. `gate`가 있으면 같은 이름의 렌즈(`lens`)가 빛을 받는 동안 꺼진다(보안 광선 문) |
| `cycle` | `{ period, on, phase }` | 켜짐 주기(초) — `phase`초에 켜져 `on`초 동안, `period`마다. 감전과 바람에 쓴다 |
| `gate` | 이름 | 위 `gate`·`named`·`shock`이 짝을 찾는 이름 |
| `wind` | `[가로, 세로]` px/s | 바람 — 가로는 몸을 실어 나르고, 세로가 음수면 띄워 올린다(연기 기둥). `cycle`이면 돌풍 |
| `conveyor` | px/s(+는 오른쪽) | 컨베이어 — 단단한 칸에 얹는다 |
| `fluid` | `water` | 물 — 빈 칸이면 헤엄친다. 발판·엽전과 겹쳐도 된다 |
| `anchor` | 그림 이름(`lantern`) | 그래플 고리 |
| `pickup` | `yeopjeon` | 엽전 |
| `marker` | `spawn` · `goal` · `checkpoint` · `arenaL` · `arenaR` | 시작·골·체크포인트·보스 아레나의 두 끝 |
| `decor` | `roof` · `wall` · `beam` · `spike` · `dancheong` · `post` · `catflap` · `crumble` · `wire` · `breakable` · `belt` · `lightdoor` · `gate` · `named` | 타일 그림(충돌과는 따로). 그림 틀 시트의 이름과 같다 |

- 엔티티(`entities`, 칸 좌표 `x`·`y`와 `params`):

| `type` | 무엇 | `params` |
|---|---|---|
| 적(`enemies.json`의 키) | 아래 적 표 | `patrol`(칸) · `facing` · `shield` · `shieldHp` · `hp` |
| `boss` | 보스(아래) | `id`(`bosses.json`의 키) · `facing` |
| `mover` | 움직이는 발판 — 제자리에서 (`dx`,`dy`)칸까지 오간다 | `w`(칸, 기본 2) · `dx` · `dy` · `period`(초, 기본 4) · `phase` |
| `lens` | 렌즈 — 레이저를 맞으면 같은 `gate` 이름의 감전 칸을 끈다 | `gate` · `hold`(빛이 끊긴 뒤 더 버티는 초) |
| `switch` | 봉수 화로 — 레이저·낙뢰로 켠다. 같은 `gate` 이름의 봉수 문을 연다 | `gate` · `hold`(켜져 있는 초, 0이면 계속) |
| `name-tablet` | 이름 패 — 주우면 같은 `gate` 이름의 백패 칸이 단단해진다 | `gate` · `key`(주울 때 뜨는 말풍선 글) |
| `gaedarae` | 개다래 앰플(아래 「시작 꼴과 각성」) | — |
| `heal` | 회복 | — |
| `bigeup-page` | 비급 쪽 | `itemId` |
| `treasure` | 숨은 보물함 — 처음 열면 품계 경험 `rewards.treasurePumgyeXp` | `itemId` |
| `speech` | 닿으면 그 글을 말풍선으로 | `key` |
| `sakura` | 그림뿐인 장식(벚나무). `last: true`인 나무는 몸이 가까이 와야 선다 | `last` |

- 적의 `behavior`(`enemies.json`) — 새 적은 이 중 하나를 고르고 수치만 정하면 된다.

| `behavior` | 하는 일 | 지금 쓰는 적 |
|---|---|---|
| `patrol` | 땅을 걷다 보면 돌진 | `haetae-bot` |
| `turret` | 서서 쏜다(`bulletBounces`만큼 거울에서 꺾인다) | `pojol-bot` · `mirror-turret` |
| `flyer` | 공중에서 출렁이며 순찰하다 아래에 들면 급강하 | `haetae-drone` |
| `artillery` | 플레이어 발밑에 포물선 포탄 — 떨어질 자리가 미리 표시된다 | `hwapo-pojol` |
| `keg` | 화약통 — 맞으면 터진다. 처치 수에 들지 않는다 | `powder-keg` |
| `mine` | 수뢰 — 떠다니다 몸이 `trigger` px 안에 들면 `fuse`초 뒤 터진다 | `sumoe` |
| `heavy` | 정면 방패를 세우고 걷는다 — 등·위에서 맞아야 다친다. 등 뒤를 보면 `turn`초 뒤에 돌아선다 | `jangseung-heavy` |

- `speech`는 제 칸에서 위로 여섯 칸·아래로 두 칸까지 닿는다 — 위를 뛰어 넘어가도 뜬다. 빈 칸이 아닌 칸(천장·위층 바닥·들보)에서 멈추므로
  아래층이나 천장 너머에서는 미리 뜨지 않는다(0.1.0).
- 시작 꼴과 각성: `startForm`(`cat`·`human`) · `awakenDuration`(초, 0이면 영구). 시간제(0보다 크다)인 판의 `gaedarae`는 **앰플 대**다 —
  줍혀도 사라지지 않고, 각성이 풀려 고양이로 돌아온 몸이 다시 닿으면 또 깨어난다(0.1.0). 영구인 판에서는 한 번 줍으면 사라진다.
- 맵 필드 `setsFlags`(문자열 목록) — 클리어하면 세이브의 `flags`에 켠다. 1장 보스의 `permanentHuman`이 켜진 뒤로는 2장부터의 판이
  `startForm:"cat"`이어도 이형태로 시작한다.
- 맵 필드 `party`(캐릭터 id 목록, 0.5.1) — 허브의 **추천 편성**. 앞세울 순서대로 적으면 명부에 든 앞의 셋을 권하고(모자라면 지금 편성으로
  채운다), 까닭은 로케일 `stages:<id>.party`가 든다. 없으면 그 판의 방패 색(보스 단계 순서가 먼저, 그다음 적의 방패가 많은 순)을 깨는 동료로 고른다.
  명부에 동료가 하나뿐이면 권하지 않는다.
- 적 `params`: `patrol`(칸) · `facing`(`left`·`right`) · `shield`(`black`·`white`·`red`·`blue`·`yellow`) · `shieldHp` · `hp`.
- **보스** — 엔티티 `boss`(`params.id` = `bosses.json`의 키, `facing`)와 마커 `arenaL`·`arenaR` 한 쌍. 플레이어가 두 마커 사이에
  온전히 들어서면 마커 칸부터 위로 막힌 칸까지 문이 닫히고 카메라가 아레나에 묶인다. 보스가 쓰러지면 문이 열리고 그때 `goal`이
  나타난다(그 전에는 골이 없다). 파티가 다 쓰러지거나 아레나 밖에서 되살아나면 싸움은 처음으로 돌아간다.
- 그 밖의 맵 칸: `bgm`(곡 이름) · `intro`·`outro`(`story:` 줄 목록, 판 앞뒤 대사) · `unlocks`(클리어하면 합류하는 동료 id) ·
  `rising`(아래에서 차오르는 불 — `speed` · `start` · `delay` · `gap` · `top`) · `fog`(안개 — `sight` 적 시야 배수 · `radius` 맑은 반지름 ·
  `density`) · `autoscroll`(보스 아레나가 문 대신 `speed` px/s로 오른쪽으로 가다 `end`열에서 멈춘다).
- 장별 그림: `tileset`(`alley`·`tower`·`forge`·`harbor`·`mountain`·`ark`)이 타일 시트 `tiles_<theme>`와 시차 배경 `bg_<theme>_far`·`_mid`·`_near`를 고른다.
  없으면 1장 그림(`tiles`·`bg_far`·`bg_mid`·`bg_near`). `parallax`를 적으면 시차 배경만 그것으로 바꾼다. 새 테마는 그 이름의 그림 한 벌을 `sprites/`에 넣으면 된다.
- 스테이지 순서는 `data/chapters.json`이 정한다(지금은 모드가 고칠 수 없다 — 새 스테이지는 기존 자리의 `id`로 넣는다).

## 6. 그림

시트는 Pixi 스프라이트시트 JSON + PNG다. **애니메이션 이름만 지키면** 프레임 크기·개수는 자유다.

- 인간형(동료 다섯): `idle` `run` `jump` `fall` `crouch` `crawl` `slide` `roll` `grapple` `wall` `hurt` `dash` `swim` — 엄장은 `slam`도
- 고양이형: `idle` `run` `jump` `fall` `crouch` `crawl` `slide` `roll` `wall` `hurt` `swim`(고양이도 벽을 찬다 — 0.0.3)
- 적: 해태봇 `walk` `charge` · 포졸봇·화포 포졸 `idle` `shoot` · 해태 드론 `fly` `tell` `dive` · 거울 포탑·화약통 `idle` · 수뢰 `idle` `armed` ·
  장승 중보병 `walk` `charge` `turn`
- 장치: 렌즈·봉수 화로·체크포인트 `off` `on` · 엽전 `spin` · 그 밖의 줍는 것·고리·벚나무 `idle`
- 타일(`tiles`, 테마 시트 `tiles_<theme>`도 같은 이름): `roof_top` `roof_fill` `wall` `wall_top` `beam` `spike` `post` `dancheong` `catflap`
  `crumble` `wire` `shock_on` `shock_off` `mover` `mirror_f` `mirror_b` `breakable` `belt` `lightdoor` `lightdoor_on` `lightdoor_off`
  `gate` `gate_open` `named` `named_on`
- 보스: `idle` `walk` `roar`(등장·단계 전환) `tell`(패턴 예고) `charge` `jump`(도약) `cast`(부채꼴·소환·광선·낙하물·물결·포격·낙뢰)
- 그림은 오른쪽을 본다(게임이 뒤집는다). 발 가운데가 기준점(anchor 0.5, 1)이다.

## 7. 언어 더하기

`mod.json`에 `languages`를 적고 `locales/<id>/*.json`을 싣는다. 빠진 글은 `fallback`(기본 `ko`)으로 떨어진다.

판 안내(`stages`의 `tips`)와 편성 조언(`party`)은 터치 조작일 때 같은 이름 뒤에 `_touch`가 붙은 글을 먼저 찾는다(예: `"jump_touch"`).
키 이름이 들어간 글만 터치 판을 따로 적으면 되고, 없으면 원래 글이 나온다(0.6.5~).

```json
"languages": [ { "id": "en", "name": "English", "fallback": "ko", "fonts": { "ui": "myFont" } } ],
"fonts": { "myFont": { "file": "fonts/MyFont.woff2", "family": "MyFont" } }
```

## 8. Advanced — 코드

예제 모드는 `examples/mods/`에 있다 — 데이터만 바꾸는 맵 모드 하나(`kkam.example.rooftop-dash`).

`entry`의 JS는 인자 `kkam` 하나를 받는 함수 본문으로 실행된다. 처음 켤 때 사람에게 묻는다.

| `kkam.` | capability | 무엇 |
|---|---|---|
| `log(msg)` · `assetUrl(rel)` · `gameVersion` | — | |
| `items.get/ids/set` | — | 아이템 표 |
| `data.characters/enemies/bosses/physics/progression(patch)` | — | 표 깊은 병합 |
| `audio.bgm/sfx(patch)` · `maps.ids/register(map)` · `i18n.addBundle/t` | — | |
| `events.on(name, fn)` | `events` | `boot:modsLoaded` `boot:ready` `stage:start` `stage:clear` `save:write` `save:load` |
| `mixin.inject(target, { at, priority, handler })` | `mixin` | 아래 대상에 `head`·`return`·`replace` |
| `game.state()` · `game.dispatch(action)` · `game.setFlag(name)` | `state` | 진행 상태 |
| `internal.list()` · `internal.module(path)` | `internal` | 내부 모듈 — **아무 약속도 없다** |

**mixin 대상**(판을 올려도 이름은 지킨다): `stageRank` · `stageRewards` · `skillUpgradeCost` · `characterStats` · `stageEntities` ·
`resolveBgmTrackId` · `getItem`. `stageRewards`의 인자에는 0.0.3부터 `treasures`(이번에 처음 연 보물함 수)가 붙는다.

물리 한 스텝 자체(엔진 안)에는 끼어들 수 없다 — 대신 `data/physics.json`과 `characterStats`로 수치를 바꾼다.

## 9. 기록과 로그

`logs/`에 실행마다 기록이 남는다(최근 20개). 모드가 실패하면 어느 모드가 왜 실패했는지 거기 적힌다.
