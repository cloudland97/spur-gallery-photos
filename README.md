# Spur Gallery

개인 VRChat 스크린샷 갤러리 **사이트**입니다. 저장소에 있는 모든 사진은 저장소 소유자 본인이 VRChat 내에서 직접 촬영한 것입니다.

**사이트: https://cloudland97.github.io/spur-gallery-photos/**

이 저장소는 두 가지 역할을 겸합니다.

1. `index.html` 을 통해 브라우저에서 볼 수 있는 사진 갤러리 사이트
2. 같은 VRChat 월드(Bongall/SpurGallery)가 런타임에 `photos/` 아래 파일을 URL로 직접 불러다 쓰는 이미지 소스

## 디렉터리 구조

| 경로 | 설명 |
|---|---|
| `index.html` | 갤러리 웹페이지. 외부 의존성 없이 `manifest.json` 을 읽어 썸네일 그리드와 라이트박스를 렌더링한다. |
| `photos/0001.jpg` ~ `photos/{total}.jpg` | 원본 스크린샷. 파일명은 촬영 순서(=manifest 배열의 인덱스+1)를 4자리로 0-padding한 번호. **VRChat 월드가 이 경로를 직접 참조하므로 파일명·경로를 바꾸면 안 된다.** |
| `photos/atlas/A001.jpg` ~ `photos/atlas/A0{atlasN}.jpg` | 여러 장의 사진을 한 장으로 묶어 압축한 아틀라스 텍스처. VRChat 월드가 개별 원본 대신 이 아틀라스 조각을 불러와 다운로드량을 줄이는 용도로 쓴다(아래 `atlasPos`/`atlasRect` 참고). |
| `manifest.json` | 사진 메타데이터. 아래 스키마 참고. VRChat 월드 런타임과 `index.html` 이 모두 파싱하므로 기존 키 구조를 바꾸면 안 된다. |
| `map.csv` | `번호;원본 VRChat 캡처 파일명` 매핑. `번호`는 1부터 시작하며 `manifest.json` 배열 인덱스+1과 같다(=`photos/{번호 4자리}.jpg`). 원본 촬영 파일명(`VRChat_YYYY-MM-DD_HH-mm-ss.SSS_...`)을 보존해 두는 용도. |
| `inbox/` | 신규 사진 원본을 임시로 두는 작업 폴더. `.gitignore` 대상이라 저장소에는 올라가지 않는다. |
| `featured/` | 큐레이션(추천) 픽업용 폴더. 현재는 비어 있다. |

## `manifest.json` 스키마

모든 배열은 **0-based 인덱스**로 정렬되어 있고, 인덱스 `i` 는 파일 `photos/{i+1 을 4자리로 0-padding}.jpg` 에 대응한다(예: 인덱스 `0` → `photos/0001.jpg`).

| 키 | 타입 | 설명 |
|---|---|---|
| `total` | number | 전체 사진 수 |
| `months` | `{ id, start, count }[]` | 월별 구간. `id`는 `"YYYY-MM"`, `start`는 해당 월 첫 사진의 인덱스, `count`는 장수. 인덱스 순서 자체가 이미 시간순이다. |
| `dates` | string[] | 사진별 촬영일, `"YY.MM.DD"` 형식(예: `"24.05.04"` → 2024-05-04). 날짜를 모르면 빈 문자열 `""`. |
| `aspect` | number[] | 사진별 가로/세로 비율(width/height). 이미지를 내려받기 전에 레이아웃 크기를 잡기 위한 값. |
| `atlasN` | number | 아틀라스 텍스처 파일 개수(`photos/atlas/A001.jpg` ~ `A0{atlasN}.jpg`). |
| `atlasPos` | number[] | 사진별로 `atlasRect` 배열의 인덱스를 가리킨다. 아틀라스에 없으면 `-1`. |
| `atlasRect` | `[atlas파일 인덱스(0-based), x, y, w, h][]` | 아틀라스 이미지 안에서 해당 조각이 차지하는 정규화 좌표(0~1). `atlas파일 인덱스 0` = `photos/atlas/A001.jpg`. |
| `insideX` | number[] | 내벽 캔버스(가로 액자) 8칸용 **미고정 풀**. 앞에서부터 빈 액자에 순서대로 채워진다. 현재 8개. |
| `insideY` | number[] | 내벽 위쪽 세로 액자 8칸용 미고정 풀. 현재 8개. |
| `outsideX` | number[] | 외벽 캔버스(가로 액자) 27칸용 미고정 풀. **현재 빈 배열** — 27칸이 전부 아래 `outsideX_N`으로 고정돼 있어서 풀에 남는 사진이 없다. |
| `outsideY` | number[] | 외벽 위쪽 세로 액자 27칸용 미고정 풀. 9:16 기준면이라 세로 사진용. **현재 빈 배열**(위와 같은 이유). |
| `main` | number[] | 갤러리 정면 대형 액자. 여러 장이면 슬라이드로 순환한다. |
| `meme` | number[] | 비밀방 전용. 허브 뒷벽의 콜라이더 없는 가짜 패널을 통과해야 나오는 방이라 일반 동선에는 노출되지 않는다. |
| `higana` | number[] | 촬영자 구분. 갤러리는 촬영자별로 레인이 나뉘며(나가는 길 / 돌아오는 길) 각 레인이 독립적으로 시간순 정렬된다. |
| `portrait` | number[] | 세로 방향 사진 인덱스. 세로 액자(`insideY`/`outsideY`) 배치 판단에 쓰인다. *(추정 — 배열 내용과 `aspect` 값의 상관으로 판단)* |
| `outsideX_1` ~ `outsideX_27`, `outsideY_1` ~ `outsideY_27` | number[] | **액자 번호 고정 매핑.** 예를 들어 `outsideX_5`는 외벽 5번째 액자에 고정할 사진이다. 번호 순서는 입구에서 시계방향. 이 키가 없는 액자에는 나머지 사진이 순서대로 채워지므로, 사진을 추가·삭제하면 고정되지 않은 액자의 배치는 밀린다(정상 동작). |

슬롯 총합은 71칸이다 — `insideX` 8 + `insideY` 8 + `outsideX` 27 + `outsideY` 27 + `main` 1. 각 칸에 사진 한 장이 올라가고, 한 칸에 여러 장이 배정되면 일정 간격으로 순환한다.

## 사진 추가 워크플로

이 저장소는 VRChat 월드 프로젝트(상위 폴더)의 PowerShell 스크립트로 관리된다. 스크립트는 이 저장소가 아니라 상위 프로젝트의 `tools/` 폴더에 있다(예: 상위 프로젝트 루트가 `Bongall/` 이면 `Bongall/tools/Add-Month.ps1`).

- `tools/Add-Month.ps1` — `inbox/`에 넣어둔 사진을 리사이즈(최대 2048px) + EXIF/GPS 메타데이터 제거 후 전역 순번으로 `photos/`에 배치하고 `manifest.json`을 갱신, git push까지 수행한다.
- `tools/Import-Archive.ps1` — 최초 1회 대량 임포트용. `inbox/` 하위 폴더를 재귀 스캔해 VRChat 캡처 파일명에서 날짜를 추출하고 시간순으로 정렬해 `manifest.json`을 처음부터 재구성한다.
- `tools/Auto-Group.ps1` / `tools/Set-Groups.ps1` — 같은 장면 연속 촬영 사진을 자동 클러스터링해 그룹 태그를 붙인다.
- `tools/Set-Featured.ps1` — `featured/` 큐레이션 목록을 갱신한다.

사진은 추가 전에 긴 변 2048px로 리사이즈되고 EXIF/GPS 메타데이터가 제거된다(VRChat의 URL 이미지 최대 해상도가 2048×2048이기도 하다).

## 라이선스

- 사진(`photos/`, `photos/atlas/`, `featured/`): **CC BY-NC 4.0** (저작자 본인 촬영, 비영리 저작자표시). 자세한 내용은 [`LICENSE`](./LICENSE) 참고.
- 코드(`index.html` 등): **MIT**. 자세한 내용은 [`LICENSE-CODE`](./LICENSE-CODE) 참고.
