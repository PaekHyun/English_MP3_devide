# English_MP3_devide — 페이지 구분 소리 기반 MP3 자동 분할 도구

## 배경

아이들 책을 읽어주는 CD 오디오는 **특정 소리(책장 넘기는 소리)** 로 페이지를 구분하고 있습니다.

그래서 세이펜 작업을 할 때 이 오디오를 페이지 단위 MP3로 모두 분리하기가 쉽지 않은데,
이 도구는 페이지 구분 소리를 템플릿으로 자동 매칭해서 **MP3 파일을 페이지별로 모두 나눠 줍니다.**

## 동작 원리

1. 마커 소리 템플릿과 대상 MP3를 22050Hz 모노로 로드
2. 표준화 정규화 후 시간 도메인 **교차상관(cross-correlation)** 계산
3. 동적 임계값 산출: `mean + (max − mean) × sensitivity`
4. `find_peaks`로 임계값 이상 피크 = 마커(책장 소리) 위치 검출 (최소 간격 `min_gap_sec`)
5. pydub로 마커 구간(피크 앞 0.1초 ~ 뒤 0.2초+템플릿 길이)을 잘라내고
   남은 구간을 `segment_001.mp3`, `segment_002.mp3`, ... 로 저장

## 파일 구성 (`book_page_turn_splitter.ipynb`)

| 셀 | 내용 |
|---|---|
| 0 | 노트북 제목: 페이지 구분 소리와 전체 녹음 소리의 상관관계 분석 |
| 1 | 템플릿↔메인 상관계수 실험 (튜닝용) |
| 4 | **`auto_split_page_turns()` (실효 버전) + `batch_process_mp3_folder()` 폴더 일괄 분할** |
| 7 | `cut_mp3_folder_ms()` — 폴더 내 모든 MP3 앞/뒤 ms 단위 절단 |
| 11 | `cut_matching_mp3_in_tree()` — 하위 폴더 트리에서 특정 파일명만 절단 |
| 13~16 | 후처리 실행 예시 (segment_012/013/016/018 뒤 17초 절단 — 수동 조정 기록) |

## 사용법

1. 준비물
   - 마커 소리 템플릿 파일 (예: `book_page_turn_34s.mp3`) — **마커 소리 1회 분량 권장**
   - 분할 대상 MP3가 담긴 폴더
2. 노트북에서 일괄 분할 실행:
   ```python
   batch_process_mp3_folder(
       input_folder="./mp3_folder",
       template_path="./book_page_turn_34s.mp3",
       output_root_dir="./split_results",
   )
   ```
3. 결과: `split_results/<원본 파일명>/segment_NNN.mp3` (파일별 하위 폴더 생성)

### 주요 파라미터

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| `sensitivity` | 0.6 | 감도. 오탐이 많으면 ↑, 마커를 놓치면 ↓ |
| `min_gap_sec` | 6.0 | 마커 사이 최소 간격(초) |
| `remove_before_sec` / `remove_after_sec` | 0.1 / 0.2 | 마커 앞뒤로 잘라내는 여유 |
| `bitrate` | 192k | 저장 MP3 비트레이트 |

## 요구 사항

- Python 3 + `librosa`, `scipy`, `pydub` (pydub는 ffmpeg 필요)

## 주의 사항

- 템플릿에 마커 소리 외 여백이 길면 세그먼트 뒤쪽이 함께 잘리므로, 마커 1회 분량으로 자른 파일을 권장
- 셀 12~16의 절단 값(뒤 17초 등)은 작업 시점에 수동 맞춘 값이므로 용도에 맞게 수정해서 사용
