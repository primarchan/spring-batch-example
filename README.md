# 대규모 서비스를 위한 Spring Batch

## 성능 개선과 성능 비교

### 성능 개선 계획 이해
- `SaveUserTasklet` 에서 `User` 데이터 40,000 건 저장
- `Chunk Size` 는 1,000
- 성능 개선 대상 `Step` 은 `userLevelUpStep`
- 아래 표 순서대로 실행
  - 예제를 만들고 성능 측정 후 비교
  - 3번씩 실행
  - 환경에 따라 성능이 다를 수 있음

| 구분                        | 1회   | 2회   | 3회        |
|---------------------------|------|------|-----------|
| Simple Step               | 4930mills | 5101mills | 5026mills |
| Async Step                | 4944mills | 5006mills | 4950mills |
| Multi-Thread Step         | 3308mills | 3201mills | 3233mills |
| Partition Step            | 3666mills | 3617mills | 3650mills |
| Async + Partition Step    | 3686mills | 3626mills | 3624mills |
| Parallel Step             | 4820mills | 4842mills | 4823mills |
| Partition + Parallel Step | 3589mills | 3554mills | 3521mills |

### Async Step 적용하기
- `ItemProcessor` 와 `ItemWriter` 를 `Async` 로 실행
- `java.util.concurrent` 에서 제공되는 `Future` 기반 `asynchronous`
- `Async` 를 사용하기 위해 `spring-batch-integration` 이 필요

### Multi-Thread Step 적용하기
- `Async Step` 은 `ItemProcessor` 와 `ItemWriter` 기준으로 비동기 처리
- `Multi-Thread Step` 은 `Chunk` 단위로 `Multi-threading` 처리
- `Thread-Safe` 한 `ItemReader` 필수

### Partition Step 적용하기
- 하나의 `Master` 기준으로 여러 `Slave Step` 을 생성해 `Step` 기준으로 `Multi-Thread` 처리
- 예를 들어
  - `item` 이 40,000 개, `Slave Step` 8개이면,
  - 40,000 / 8 = 5,000 이므로 하나의 `Slave Step` 당 50,000건 씩 나눠서 처리
- `Slave Step` 은 각각 하나의 `Step` 으로 동작


### Parallel Step 적용하기
- `n개`의 `Thread` 가 `Step` 단위로 동시 실행
- `Multi-Thread Step` 은 `Chunk` 단위로 동시 실행했다면, `Parallel Step` 은 `Step` 단위로 동시 실행
- 예시
  - `Job` 은 `FlowStep1` 과 `FlowStep2` 를 순차 실행
  - `FlowStep2` 는 `Step2` 와 `Step3` 을 동시에 실행
  - 설정에 따라 `FlowStep1` 과 `FlowStep2` 를 동시 실행도 가능