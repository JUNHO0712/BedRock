# 코드 리뷰 리포트

## my_code.py (246줄)

### 스타일 검사

코드가 100번 라인에서 잘려 있지만, 제공된 코드 범위 내에서 리뷰를 진행하겠습니다.

---

다음은 PEP8 및 일반적인 스타일 규칙 위반 사항입니다:

---

**네이밍 규칙 위반**

- [26라인] 스타일: 함수명 `now_text()`는 기능을 명확히 표현하지 못합니다. PEP8 위반은 아니나, 의미 전달이 부족합니다.
  - 수정 제안: `get_current_timestamp()` 또는 `get_now_isoformat()`
- [30라인] 스타일: 함수명 `save_line()`은 CSV 행 저장이라는 구체적 동작을 충분히 표현하지 못합니다.
  - 수정 제안: `append_csv_row(filename, row)`
- [36라인] 스타일: `make_base_payload()`의 `make_`는 Python 관용 표현보다 `create_` 또는 `build_`가 더 명확합니다.
  - 수정 제안: `build_base_payload(vehicle)` 또는 `create_base_payload(vehicle)`
- [21~22라인] 스타일: `cleansed_list`, `anomaly_list`는 전역 변수로 사용되며, 이름만으로 전역 상태임을 알기 어렵습니다.
  - 수정 제안: 전역 변수 사용 자체를 지양하거나, 명확한 모듈 수준 주석 추가

---

**매직 넘버 사용**

- [59라인] 스타일: `37.40`, `37.75`, `0.005` 등 위도 범위 및 이동 step 값이 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    LAT_MIN = 37.40
    LAT_MAX = 37.75
    LON_MIN = 126.80
    LON_MAX = 127.15
    LOCATION_STEP = 0.005
    ```
- [61라인] 스타일: 속도 범위 `0`, `110`, step `8`이 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    SPEED_MIN = 0
    SPEED_MAX = 110
    SPEED_STEP = 8
    ```
- [62라인] 스타일: 연료 범위 `0`, `100`, step `2`가 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    FUEL_MIN = 0
    FUEL_MAX = 100
    FUEL_STEP = 2
    ```
- [75라인] 스타일: 속도 스파이크 범위 `80`, `120`이 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    SPEED_SPIKE_MIN = 80
    SPEED_SPIKE_MAX = 120
    ```
- [81~82라인] 스타일: 위치 이상값 오프셋 `3`, `5`가 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    INVALID_LOCATION_OFFSET_MIN = 3
    INVALID_LOCATION_OFFSET_MAX = 5
    ```
- [88라인] 스타일: 저연료 범위 `0`, `3`이 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    LOW_FUEL_MIN = 0
    LOW_FUEL_MAX = 3
    ```
- [94~100라인] 스타일: 이상 데이터 발생 확률 `0.05`, `0.10`, `0.15`, `0.20`이 하드코딩되어 있습니다.
  - 수정 제안:
    ```python
    PROB_MISSING = 0.05
    PROB_SPEED_SPIKE = 0.10
    PROB_INVALID_LOCATION = 0.15
    PROB_LOW_FUEL = 0.20
    ```

---

**함수 설계 및 복잡도**

- [58~63라인] 스타일: `update_vehicle_location()`은 위치, 속도, 연료를 모두 수정하며 단일 책임 원칙(SRP)에 위배됩니다. 함수명도 `location`만 암시하지만 실제로는 speed, fuel도 변경합니다.
  - 수정 제안: 함수명을 `update_vehicle_state()`로 변경하거나, 각 항목별 업데이트 함수로 분리
- [66~89라인] 스타일: `inject_missing_data`, `inject_speed_spike`, `inject_invalid_location`, `inject_low_fuel` 4개 함수가 모두 `broken = dict(payload)` 복사 후 특정 필드만 수정하는 동일한 패턴을 반복합니다.
  - 수정 제안: 공통 로직을 추출한 헬퍼 함수 또는 딕셔너리 기반 디스패치 구조로 리팩터링
    ```python
    def _copy_and_update(payload: dict, updates: dict) -> dict:
        broken = dict(payload)
        broken.update(updates)
        return broken
    ```
- [92~100라인] 스타일: `choose_payload_mode()`의 확률 분기가 연속 `if`로 나열되어 있어 확률값 변경 시 유지보수가 어렵습니다.
  - 수정 제안: 확률-모드 매핑 테이블 방식으로 리팩터링
    ```python
    PAYLOAD_MODES = [
        (PROB_MISSING, "missing"),
        (PROB_SPEED_SPIKE, "speed"),
        (PROB_INVALID_LOCATION, "location"),
        (PROB_LOW_FUEL, "fuel"),
    ]
    ```

---

**타입 힌트 누락**

- [26라인] 스타일: `now_text()` 함수에 반환 타입 힌트가 없습니다.
  - 수정 제안: `def now_text() -> str:`
- [30라인] 스타일: `save_line(filename, row)` 함수에 파라미터 및 반환 타입 힌트가 없습니다.
  - 수정 제안: `def save_line(filename: str, row: list) -> None:`
- [36라인] 스타일: `make_base_payload(vehicle)` 함수에 타입 힌트가 없습니다.
  - 수정 제안: `def make_base_payload(vehicle: dict) -> dict:`
- [49라인] 스타일: `random_walk(value, minimum, maximum, step)` 함수에 타입 힌트가 없습니다.
  - 수정 제안: `def random_walk(value: float, minimum: float, maximum: float, step: float) -> float:`
- [58라인] 스타일: `update_vehicle_location(vehicle)` 함수에 타입 힌트가 없습니다.
  - 수정 제안: `def update_vehicle_location(vehicle: dict) -> dict:`
- [66~89라인] 스타일: `inject_*` 계열 함수 모두 타입 힌트가 없습니다.
  - 수정 제안: 각 함수에 `(payload: dict) -> dict` 타입 힌트 추가
- [92라인] 스타일: `choose_payload_mode()` 함수에 반환 타입 힌트가 없습니다.
  - 수정 제안: `def choose_payload_mode() -> str:`

---

**docstring 누락**

- [26라인] 스타일: `now_text()` 함수에 docstring이 없습니다.
  - 수정 제안:
    ```python
    def now_text() -> str:
        """현재 시각을 ISO 8601 형식의 문자열로 반환합니다."""
    ```
- [30라인] 스타일: `save_line()` 함수에 docstring이 없습니다.
  - 수정 제안:
    ```python
    def save_line(filename: str, row: list) -> None:
        """지정된 CSV 파일에 한 행을 추가(append) 방식으로 저장합니다."""
    ```
- [49라인] 스타일: `random_walk()` 함수에 docstring이 없습니다. 파라미터 의미 설명이 필요합니다.
  - 수정 제안:
    ```python
    def random_walk(value: float, minimum: float, maximum: float, step: float) -> float:
        """
        현재 값에서 [-step, step] 범위의 랜덤 변화를 적용하고,
        [minimum, maximum] 범위로 클리핑하여 반환합니다.
        """
    ```
- [58라인] 스타일: `update_vehicle_location()` 함수에 docstring이 없습니다.
  - 수정 제안: 함수가 위치뿐 아니라 속도·연료도 갱신함을 명시하는 docstring 추가
- [66~89라인] 스타일: `inject_*` 계열 함수 모두 docstring이 없습니다.
  - 수정 제안: 각 함수에 어떤 이상 데이터를 주입하는지 설명하는 한 줄 docstring 추가
- [92라인] 스타일: `choose_payload_mode()` 함수에 docstring이 없습니다.
  - 수정 제안: 확률 기반으로 페이로드 모드를 선택함을 설명하는 docstring 추가

---

**파일 핸들링**

- [30~33라인] 스타일: `save_line()`은 매 호출마다 파일을 열고 닫습니다. 멀티스레딩 환경(4라인에서 `threading` import 확인)에서 `list_lock`이 파일 쓰기에는 적용되지 않아 동시 쓰기 시 파일 손상 위험이 있습니다.
  - 수정 제안: `save_line()` 내부에서도 `list_lock`을 사용하거나, 파일 쓰기 전용 락을 별도로 정의
    ```python
    file_lock = threading.Lock()
    
    def save_line(filename: str, row: list) -> None:
        with file_lock:
            with open(filename, "a", encoding="utf-8", newline="") as f:
                writer = csv.writer(f)
                writer.writerow(row)
    ```

---

> **참고:** 코드가 100라인에서 잘려 있어 `choose_payload_mode()` 이후 로직, 메인 실행 흐름, 전역 변수 실제 사용처에 대한 리뷰는 포함되지 않았습니다. 전체 코드 제공 시 추가 리뷰가 가능합니다.

### 보안 검사

코드를 분석한 결과, 아래와 같은 보안 및 코드 품질 위험 요소가 존재합니다.

> **참고**: 제공된 코드는 라인 100번에서 잘려 있어, 이후 로직(파일 저장, DB 연동, 권한 처리 등)은 확인할 수 없습니다. 확인 가능한 범위 내에서 분석합니다.

---

## 심각도: 높음

### 1. 전역 공유 리스트에 대한 동시성 문제

- **위치**: 라인 21~23, `cleansed_list`, `anomaly_list`
- **유형**: 동시성 문제
- **설명**: `cleansed_list`와 `anomaly_list`는 전역 리스트로 선언되어 있고, `list_lock`이 존재하지만 실제로 이 리스트에 접근하는 코드가 잘려 있어 락이 올바르게 사용되는지 확인할 수 없습니다. 만약 락 없이 여러 스레드가 동시에 `.append()` 등을 호출하면 데이터 손상(race condition)이 발생할 수 있습니다. `threading` 모듈이 임포트되어 있어 멀티스레드 환경임이 명확합니다.
- **수정 제안**: 리스트에 접근하는 모든 코드에서 반드시 `with list_lock:` 블록 내에서만 읽기/쓰기를 수행하도록 강제하세요. 또는 `queue.Queue`를 사용하면 스레드 안전성이 내장되어 있어 더 안전합니다.

```python
# 권장 방식
import queue
cleansed_queue = queue.Queue()
anomaly_queue = queue.Queue()
```

---

### 2. 파일 쓰기 시 동시성 및 원자성 미보장

- **위치**: 라인 30~33, `save_line` 함수
- **유형**: 파일 처리 안정성 / 동시성 문제
- **설명**: `save_line` 함수는 파일을 `"a"` (append) 모드로 열어 CSV 행을 씁니다. 멀티스레드 환경에서 여러 스레드가 동시에 같은 파일에 쓰기를 시도할 경우, 파일 핸들이 충돌하여 데이터가 뒤섞이거나 손상될 수 있습니다. 특히 `CLEANSED_FILE`과 `ANOMALY_FILE` 두 파일 모두 이 함수를 공유할 가능성이 높습니다.
- **수정 제안**: 파일 쓰기 전용 락을 별도로 두거나, 파일 쓰기 작업을 단일 전용 스레드(writer thread)에서만 처리하도록 분리하세요.

```python
file_lock = threading.Lock()

def save_line(filename, row):
    with file_lock:
        with open(filename, "a", encoding="utf-8", newline="") as f:
            writer = csv.writer(f)
            writer.writerow(row)
```

---

## 심각도: 중간

### 3. 파일명 상수 하드코딩 및 경로 검증 부재

- **위치**: 라인 10~11, `CLEANSED_FILE`, `ANOMALY_FILE`
- **유형**: 하드코딩된 설정값 / 파일 처리 안정성
- **설명**: 출력 파일 경로가 소스코드에 직접 하드코딩되어 있습니다. 현재는 단순 파일명이라 경로 탈출(path traversal) 위험은 낮지만, 향후 이 값이 외부 입력으로 변경될 경우 위험해질 수 있습니다. 또한 파일이 이미 존재할 때 헤더 중복 기록 여부도 확인이 필요합니다.
- **수정 제안**: 파일 경로는 설정 파일(`.env`, `config.yaml` 등)에서 읽어오도록 분리하고, 경로 사용 전 `os.path.abspath()`로 정규화하여 의도한 디렉터리 내에 있는지 검증하세요.

```python
import os
BASE_DIR = os.path.dirname(os.path.abspath(__file__))

def safe_path(filename):
    full_path = os.path.abspath(os.path.join(BASE_DIR, filename))
    if not full_path.startswith(BASE_DIR):
        raise ValueError("허용되지 않은 파일 경로입니다.")
    return full_path
```

---

### 4. 차량 및 운전자 정보 평문 하드코딩

- **위치**: 라인 13~19, `vehicles` 리스트
- **유형**: 하드코딩된 비밀값 / 민감 정보
- **설명**: 차량 ID(`BUS-101` 등)와 운전자 ID(`D-1001` 등)가 소스코드에 직접 하드코딩되어 있습니다. 운전자 ID는 개인 식별 가능 정보(PII)로 분류될 수 있으며, 소스코드가 버전 관리 시스템(Git 등)에 올라갈 경우 불필요하게 노출됩니다.
- **수정 제안**: 차량 및 운전자 정보는 외부 설정 파일, 데이터베이스, 또는 환경 변수에서 로드하도록 분리하세요. Git에 커밋되지 않도록 `.gitignore`도 함께 관리하세요.

---

### 5. `random` 모듈 사용 (보안 목적 부적합)

- **위치**: 라인 3, `random` 모듈 전반
- **유형**: 안전하지 않은 난수 생성
- **설명**: `random` 모듈은 암호학적으로 안전하지 않은 의사 난수 생성기(PRNG)입니다. 현재 코드에서는 시뮬레이션 데이터 생성 목적으로 사용되므로 직접적인 보안 위협은 낮습니다. 그러나 만약 이 난수가 토큰, 세션 ID, 또는 보안 관련 값 생성에 사용된다면 예측 가능성 문제가 발생합니다.
- **수정 제안**: 현재 시뮬레이션 용도라면 허용 가능하나, 보안 관련 값 생성 시에는 반드시 `secrets` 모듈을 사용하세요.

```python
import secrets  # 보안 목적 난수 생성 시
```

---

## 심각도: 낮음

### 6. 이상 데이터 주입 함수의 명시적 문서화 부재

- **위치**: 라인 66~89, `inject_*` 함수들
- **유형**: 코드 품질 / 가독성
- **설명**: `inject_missing_data`, `inject_speed_spike`, `inject_invalid_location`, `inject_low_fuel` 함수들은 테스트/시뮬레이션 목적의 이상 데이터 생성 함수입니다. docstring이나 주석이 없어 유지보수 시 의도를 오해할 수 있으며, 실수로 프로덕션 코드에 포함될 위험이 있습니다.
- **수정 제안**: 각 함수에 docstring을 추가하고, 테스트 전용 코드임을 명시하세요. 가능하다면 별도 모듈(`simulation_injectors.py`)로 분리하세요.

```python
def inject_speed_spike(payload):
    """
    [시뮬레이션 전용] 속도 이상값을 주입합니다.
    프로덕션 환경에서 사용 금지.
    """
    ...
```

---

### 7. PEP 8 네이밍 규칙 준수 (경미한 위반)

- **위치**: 라인 9~11, 상수 선언부
- **유형**: 코드 스타일
- **설명**: `TARGET_COUNT`, `CLEANSED_FILE`, `ANOMALY_FILE`은 PEP 8의 `UPPER_SNAKE_CASE` 규칙을 올바르게 따르고 있습니다. 전반적인 네이밍 규칙은 양호합니다. 다만 라인 100 이후 코드가 잘려 있어 전체 준수 여부는 확인 불가합니다.
- **수정 제안**: 현재 수준 유지, 이후 코드에서도 동일한 규칙을 적용하세요.

---

## 요청 항목 결론

| 점검 항목 | 존재 여부 | 비고 |
|---|---|---|
| 입력값 검증 부족 | 없음 (확인 범위 내) | 외부 사용자 입력을 직접 받는 코드가 확인된 범위 내에 없음. 잘린 코드 이후 존재 가능 |
| 데이터베이스 쿼리 생성 위험 | 없음 (확인 범위 내) | DB 연동 코드가 확인된 범위 내에 없음. 잘린 코드 이후 존재 가능 |
| OS 명령 실행 위험 | 없음 | `os`, `subprocess` 등 OS 명령 실행 코드 없음 |
| 하드코딩된 비밀값 | 있음 | 라인 13~19: 차량 ID, 운전자 ID 하드코딩. 라인 10~11: 파일 경로 하드코딩 |
| 민감 정보 로그 출력 | 없음 (확인 범위 내) | 확인된 범위 내 로그 출력 코드 없음. 잘린 코드 이후 존재 가능 |
| 안전하지 않은 데이터 로딩 | 없음 (확인 범위 내) | `pickle`, `yaml.load` 등 위험한 역직렬화 코드 없음 |
| 파일 처리 안정성 | 있음 | 라인 30~33: 멀티스레드 환경에서 파일 동시 쓰기 시 락 미적용 |
| 권한 검사 부족 | 정보 부족 | 코드가 라인 100에서 잘려 있어 권한 검사 로직 확인 불가 |
| 동시성 문제 | 있음 | 라인 21~23: 전역 리스트 접근 시 락 적용 여부 불명확. 라인 30~33: 파일 쓰기 락 미적용 |

