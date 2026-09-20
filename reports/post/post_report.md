# 실험 후 레포트: LAB2-01 업/다운 카운터

작성자: 상혁 (2025440084) / 작성일: `<기입>` / 소스 커밋: `<기입>` / 제출 태그: `<기입>` / GitHub 저장소: `<https://github.com/YOUR_NAME/YOUR_PROJECT>`

> 실험 후에 채운다. 수행하지 않은 항목은 "미수행"으로 표시하고, 구현 성공을 실물 동작 확인으로 대신하지 않는다. 사전 레포트: [pre](../pre/pre_report.md)

## 진행 경로

- [ ] Vivado GUI  - [ ] 오픈소스 CLI (Icarus, Yosys, nextpnr, Project X-Ray, openFPGALoader 버전 기록)

## Vivado 프로젝트와 시뮬레이션

- 프로젝트 이름 `lab2_counter`, 부품 `xc7s75fgga484-1`, Design/Simulation/Constraints 소스 등록, Simulation top `tb_counter4`, Top module `lab2_counter` (실제 화면 기준으로 확인)
- Vivado XSim 결과: `LAB2_PASS counter4 checks=36`, 종료 356 ns가 나오는지 확인하고 로그·파형 캡처를 넣는다.

| 항목 | VS Code(Icarus) | Vivado(XSim) | 차이·해석 |
|---|---|---|---|
| PASS 로그 | checks=36 | `<기입>` |  |
| 종료 시각 | 356 ns | `<기입>` |  |
| 주요 파형 | 사전 레포트 표 | `<기입>` |  |

## 합성·구현 결과

| 항목 | 값 | 해석 |
|---|---|---|
| WNS / WHS | `<기입>` | Setup·Hold failing endpoint 0인지 확인. 내부 타이밍만 뜻한다. |
| DRC | `<기입>` | 오류·경고 목록. CFGBVS-1은 보드 회로도 확인 후 값을 정하고 임의로 넣지 않는다. |
| TIMING-18 등 남은 경고 | `<기입>` | 외부 입출력 지연 미지정 경고 등 해석. |
| 자원 사용량 | `<기입>` |  |

## bit 파일

- 경로: `vivado/lab2_counter.runs/impl_1/lab2_counter.bit`
- SHA-256: `<기입>`

## 실제 장치 기록과 관찰

- Program Device 화면, 보드 전체 사진(배선·입력·출력이 함께 보이게), 조작 영상을 `evidence/post/`에 넣고 링크한다.

| 조작 | 예상 (사전 레포트) | 실제 관찰 | 비고 |
|---|---|---|---|
| K4 초기화 | LED[3:0]=0 | `<기입>` |  |
| sw[0]=0, N8 한 번 | 1 증가 (LED=0001) | `<기입>` |  |
| 15에서 N8 한 번 | 0으로 순환 | `<기입>` |  |
| sw[0]=1, N8 한 번 | 1 감소, 0에서는 15로 순환 | `<기입>` |  |
| N8를 길게 누름 | 연속 증가하지 않고 한 번만 반영 | `<기입>` |  |

## 예상과 실제의 차이·문제 해결

차이가 있었던 조건, 원인 추정, 수정 내용, 재확인 결과를 기록한다. (없으면 "차이 없음")

## 링크

- 소스 커밋 / 제출 태그:
- 사전 레포트: [pre_report.md](../pre/pre_report.md)
- 실행 로그·파형·사진·영상: `evidence/`
- GitHub 검증 기록(날짜):
