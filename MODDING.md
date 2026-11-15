# 깜냥 모딩 안내

깜냥은 MOTP(HY Project #A-01)와 같은 모드 체계를 쓴다. 지금 게임 버전은 **`0.6.2`**이다.

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
  "gameVersion": "0.6.x",
  "author": "이름",
  "description": "무엇을 바꾸는가",
  "entry": "main.js",
  "capabilities": ["events", "mixin"]
}
```

- `id`는 `[A-Za-z0-9._-]`. 같은 `id`가 둘이면 둘 다 꺼진다.
- `gameVersion` — `"0.6.x"` · `{ "min": "0.6.0", "max": "0.6.2" }` · `"*"`. 맞지 않으면 경고하고 그래도 싣는다.
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

보스 패턴은 공용 실행기 여섯이다 — 보스마다 코드를 두지 않고 데이터가 고른다(`patterns`의 키가 이름, `kind`를 비우면 이름이 곧 종류).
모든 패턴 앞에는 예고(`tell`초)가 있고 뒤에는 숨 고르기(`recover`초)가 있다.

| 종류 | 하는 일 | 주로 쓰는 칸 |
|---|---|---|
| `charge` | 예고 때 정한 쪽으로 수평 돌진, 벽에 부딪히면 쿵 | `speed` `duration` |
| `leap` | 예고 때 표시한 자리로 뛰어 내려찍고, 땅을 타는 충격파 둘 | `height` `speed` `damage` |
| `spread` | 플레이어 쪽 부채꼴 탄 | `count` `arc` `speed` `volleys` `interval` |
| `summon` | 보스 양옆에 잡몹(살아 있는 부하는 `max`까지, 보스가 쓰러지면 함께) | `enemy` `count` `max` |
| `sweep` | 눈에서 광선을 `from`°→`to`°로 휩쓴다(0° 앞 수평, 90° 바로 아래). 예고 동안 선이 깜빡인다 | `from` `to` `duration` `range` |
| `rain` | 아레나 천장에서 낙하물 — 떨어질 자리가 예고 때 바닥에 표시된다 | `count` `interval` |

단계(`phases`)는 체력 비율이 `hpBelow` 아래로 내려가면 들어서고, 들어설 때 포효하며 그 단계의 방패(`shield`·`shieldHp`)를 새로 두른다.
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
- 범례 칸: `solid`(`full`·`oneway`) · `hazard`(`spike`) · `anchor`(그래플 고리) · `pickup`(`yeopjeon`) · `marker`(`spawn`·`goal`·`checkpoint`·`arenaL`·`arenaR`) · `decor`(타일 그림: `roof`·`wall`·`beam`·`spike`·`dancheong`·`post`).
- 엔티티 종류: 적(`enemies.json`의 키) · `gaedarae`(개다래 앰플) · `heal` · `bigeup-page`(`params.itemId`) · `treasure`(`params.itemId`, 숨은 보물함 — 처음 열면 품계 경험 `rewards.treasurePumgyeXp`) · `speech`(`params.key`, 닿으면 그 글을 말풍선으로).
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
- 장별 그림: `tileset`(`alley`·`tower` …)이 타일 시트 `tiles_<theme>`와 시차 배경 `bg_<theme>_far`·`_mid`·`_near`를 고른다.
  없으면 1장 그림(`tiles`·`bg_far`·`bg_mid`·`bg_near`). `parallax`를 적으면 시차 배경만 그것으로 바꾼다. 새 테마는 그 이름의 그림 한 벌을 `sprites/`에 넣으면 된다.
- 스테이지 순서는 `data/chapters.json`이 정한다(지금은 모드가 고칠 수 없다 — 새 스테이지는 기존 자리의 `id`로 넣는다).

## 6. 그림

시트는 Pixi 스프라이트시트 JSON + PNG다. **애니메이션 이름만 지키면** 프레임 크기·개수는 자유다.

- 인간형: `idle` `run` `jump` `fall` `crouch` `crawl` `slide` `roll` `grapple` `wall` `hurt` `dash`
- 고양이형: `idle` `run` `jump` `fall` `crouch` `crawl` `slide` `roll` `wall` `hurt`(고양이도 벽을 찬다 — 0.0.3)
- 해태봇 `walk` `charge` · 포졸봇 `idle` `shoot` · 해태 드론 `fly` `tell` `dive` · 타일 `roof_top` `roof_fill` `wall` `wall_top` `beam` `spike` `post` `dancheong` (테마 시트 `tiles_<theme>`도 같은 이름)
- 보스: `idle` `walk` `roar`(등장·단계 전환) `tell`(패턴 예고) `charge` `jump`(도약) `cast`(부채꼴·소환·광선·낙하물)
- 그림은 오른쪽을 본다(게임이 뒤집는다). 발 가운데가 기준점(anchor 0.5, 1)이다.

## 7. 언어 더하기

`mod.json`에 `languages`를 적고 `locales/<id>/*.json`을 싣는다. 빠진 글은 `fallback`(기본 `ko`)으로 떨어진다.

```json
"languages": [ { "id": "en", "name": "English", "fallback": "ko", "fonts": { "ui": "myFont" } } ],
"fonts": { "myFont": { "file": "fonts/MyFont.woff2", "family": "MyFont" } }
```

## 8. Advanced — 코드

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
