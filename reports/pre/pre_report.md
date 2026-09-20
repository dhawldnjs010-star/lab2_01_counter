# 실험 전 레포트: LAB2-01 업/다운 카운터

작성자: 상혁 (2025440084) / 작성일: 2026-09-20 / 소스 커밋: `d68bd08` / workspace: `LAB1.code-workspace` (템플릿 v2.0.1) / OS: `Windows 11 Home 10.0.26200` / Python: `Python 3.14.7` / 시뮬레이터: Icarus Verilog `Icarus Verilog version 12.0 (devel) (s20150603-1539-g2693dd32b)`

> 이 레포트는 VS Code(Icarus) 시뮬레이션까지의 사전 검증이다. Vivado GUI와 실물 보드 결과는 실험 후 레포트([post](../post/post_report.md))에서 다룬다. 시각은 clk 상승 에지(5, 15, 25, … ns)의 1 ns 뒤, 즉 TB가 비교하는 시각으로 적었다.

## 목적과 예상 동작

4비트 값을 상승 에지마다 증가·감소시키는 동기식 카운터를 설계한다. 리셋·enable·방향 입력의 우선순위와 15→0, 0→15 순환을 예상하고 시뮬레이션으로 확인한다.

### 포트 (`counter4.v`의 `counter4`)

| 포트 | 방향 | 비트 폭 | 설명 |
|---|---|---|---|
| clk | in | 1 | 상승 에지 기준 클록 |
| rst | in | 1 | 동기 active-high 리셋. enable보다 우선 |
| enable | in | 1 | 1이면 갱신, 0이면 값 유지 |
| down | in | 1 | 0이면 증가, 1이면 감소 |
| value | out (reg) | 4 | 카운트 값, value[3]이 MSB |

최상위(`lab2_counter.v`, `lab2_counter`): `clk`(B6), `rst`(K4), `button`(N8), `sw[7:0]`, `led[7:0]`. 프런트엔드가 만든 `press`가 `enable`, `switches[0]`이 `down`으로 연결되고 `led = {4'b0000, value}`이다.

### 동작 규칙과 경계 입력

- 규칙: value(k+1) = rst ? 0 : (enable ? (down ? value−1 : value+1) mod 16 : value).
- 정상·경계: 증가 15→0, 감소 0→15(4비트 오버플로/언더플로), enable=0 유지, rst=1과 enable=1이 동시에 오면 리셋 우선.
- 시간 기준: TB 클록은 10 ns 주기(상승 에지 5, 15, 25, … ns)이고 입력은 에지 1 ns 뒤에 바꾼다. 레지스터는 다음 상승 에지에서 갱신된다. 실제 보드 클록(1 kHz)은 시뮬레이션의 10 ns와 별개다.

## 소스와 테스트벤치

- 설계 top: `lab2_counter` / 시뮬레이션 top: `tb_counter4`
- 소스: [`src/counter4.v`](../../src/counter4.v), [`src/input_frontend.v`](../../src/input_frontend.v), [`src/lab2_counter.v`](../../src/lab2_counter.v)
- 테스트벤치: [`sim/tb_counter4.sv`](../../sim/tb_counter4.sv)
- 제약: [`constraints/lab2_counter.xdc`](../../constraints/lab2_counter.xdc)
- 설정: [`simulation.json`](../../simulation.json) (sources 3개, testbench `sim/tb_counter4.sv`, simulation_top `tb_counter4`)

| 파일 | 역할 |
|---|---|
| `src/counter4.v` | 핵심 동작을 담은 코어 `counter4`. TB가 이 모듈을 직접 검사한다. |
| `src/input_frontend.v` | 버튼·스위치를 클록에 맞추는 입력 회로. 리셋 2단 해제 동기화, 버튼·스위치 2단 동기화 플립플롭, STABLE_CYCLES=20(1 kHz에서 20 ms) 안정 확인 뒤 한 클록짜리 `press` 펄스를 만든다. |
| `src/lab2_counter.v` | 보드 top. 프런트엔드와 코어를 연결하고 LED로 출력한다. |
| `sim/tb_counter4.sv` | 입력 자극, 기대값 계산, 자동 비교(`check`), PASS/FAIL 출력, VCD 생성, watchdog. |
| `constraints/lab2_counter.xdc` | 핀 번호·전압과 1 kHz 클록 정의. Icarus는 XDC를 읽지 않으므로 이 사전 시뮬레이션에는 사용되지 않는다. |
| `simulation.json` | VS Code 시뮬레이션 작업이 읽는 소스 목록·테스트벤치·시뮬레이션 top. |

### 테스트벤치 동작

- 자극 순서: 리셋 검사 → rst=0, enable=1로 증가 16회(1, 2, …, F, 0) → down=1로 감소 16회(F, E, …, 0) → enable=0 유지 → enable=1, down=1에서 0→F 순환 → rst=1(enable=1 유지)로 리셋 우선 검사. 각 입력은 상승 에지 1 ns 뒤에 바꾸고 다음 에지에 반영된다.
- 검사 횟수: 1(reset)+16(증가)+16(감소)+1(hold)+1(down wrap)+1(reset beats enable) = 36
- 종료·watchdog: 마지막 검사 뒤 `finish` task가 `LAB2_PASS counter4 checks=N`을 출력하고 `$finish`한다. 별도로 100000 ns(100 µs) 뒤에 `watchdog timeout`으로 `$fatal` 처리한다. 예상 종료 시각은 356 ns이다.
- 이 TB는 코어 `counter4`만 시험한다. 입력 동기화·디바운스와 실제 핀·타이밍이 통과했다는 뜻은 아니다.

### XDC 설명

`lab2_counter.xdc`은 포트 이름을 `lab2_counter.v`과 맞춰 핀을 지정한다. 모든 I/O는 `LVCMOS33`이고 `create_clock -name trainer_1khz -period 1000000.000 [get_ports clk]`로 주 클록을 1 kHz(주기 1,000,000 ns)로 정의하며 `set_false_path -from [get_ports {rst button sw[*]}]`로 비동기 입력을 타이밍 경로에서 제외한다.

| 포트 | 핀 | 보드 대응 |
|---|---|---|
| clk | B6 | 1 kHz 주 클록 |
| rst | K4 | 리셋 (active-high) |
| button | N8 | 스텝 버튼 |
| sw[7:0] | U4(sw[0]), V4, W1, W4, T1, U2, W3, Y1(sw[7]) | DIPSW8..DIPSW1 (DIPSW1..8 = sw[7]..sw[0]) |
| led[7:0] | N5(led[0]), M1, M3, M7, N7, M2, M4, L4(led[7]) | LED0..LED7 |

## VS Code 실행 과정

1. File → New Window → File → Open Workspace from File...로 `LAB1.code-workspace`를 연다. 확장(slang, VaporView, vscode-pdf)을 설치한다.
2. RTL·TB·XDC·`simulation.json`을 직접 입력하고 File → Save All.
3. Terminal → Run Task... → `01 Check tools`로 Git·Python·iverilog·vvp 버전을 확인한다.
4. `02 Simulate`를 실행해 `LAB2_PASS`와 종료 시각을 확인한다.
5. `03 Open waveform`으로 `build/sim/wave.vcd`를 VaporView로 연다. 신호: clk, rst, enable, down, value[3:0] (value는 16진수).

정상 실행 로그(본인 로그로 교체하고 `evidence/pre/`에 복사):

```text
LAB2_PASS counter4 checks=36
sim/tb_counter4.sv:19: $finish called at 356000 (1ps)
```

- 본인 실행 로그: [`../../evidence/pre/lab2_01_normal.log`](../../evidence/pre/lab2_01_normal.log)
- VCD: [`../../evidence/pre/lab2_01_wave_normal.vcd`](../../evidence/pre/lab2_01_wave_normal.vcd)
- 파형 캡처(VaporView): `evidence/pre/lab2_01_wave_zoom.png`(상승 에지 확대)
- 오류: 첫 실행에서 발생한 오류가 있으면 첫 오류 → 수정 → 재실행 로그 순서로 기록한다. (없으면 "없음", Python 실행 경로를 고쳤다면 그 내용 기입)

### 사전 파형 해석

| 시간 구간 | 입력 | 예상 | 실제 파형 | 해석 |
|---|---|---|---|---|
| 6 ns | rst=1, enable=0 | value=0 | value=0 | 첫 상승 에지(5 ns)에서 동기 리셋. 리셋 해제 전이라 값은 0. |
| 16 ns | rst=0, enable=1, down=0 · 에지 15 ns | value=1 | value=1 | 리셋 해제 뒤 첫 유효 에지에서 +1. |
| 26 ns | 에지 25 ns | value=2 | value=2 | 에지마다 1씩 증가. |
| 156 ns | 증가 15번째 에지(155 ns) | value=F | value=F | 4비트 최댓값 15(F). |
| 166 ns | 증가 16번째 에지(165 ns) | value=0 | value=0 | 경계: 15+1이 4비트를 넘어 0으로 순환. |
| 176 ns | down=1 · 첫 감소 에지(175 ns) | value=F | value=F | 경계: 0−1이 15(F)로 순환. |
| 326 ns | 감소 16번째 에지(325 ns) | value=0 | value=0 | F부터 감소해 다시 0. |
| 336 ns | enable=0 · 에지 335 ns | value=0 | value=0 | enable=0이므로 down=1이어도 값 유지. |
| 346 ns | enable=1, down=1 · 에지 345 ns | value=F | value=F | 0에서 감소하여 F. |
| 356 ns | rst=1, enable=1 · 에지 355 ns | value=0 | value=0 | 리셋이 enable보다 우선하므로 F가 아니라 0. |

표의 값은 상승 에지 직후 안정된 값이다. `LAB2_PASS`만 적지 않고 각 행에서 입력, 이전 상태, 다음 상태를 비교한다. 파형 캡처에서 위 시각을 확대해 본인 화면으로 확인한다.

## 코드 수정·실패·복구 실험

- 변경: 증가량을 1에서 2로 바꾼다(`value + 4'd1` → `value + 4'd2`).
- 변경한 파일과 위치: `src/counter4.v` 10행
- 테스트벤치 기대값은 바꾸지 않는다.

```diff
-    else value <= value + 4'd1;
+    else value <= value + 4'd2;
```

실행 전 계산: 리셋 해제 후 첫 증가 에지(15 ns)의 기대값은 1이지만 변경 회로는 0+2=2가 된다. TB는 16 ns에 `value===(1%16)`을 검사하므로 첫 증가 검사에서 실패한다.

| 단계 | 소스 커밋 또는 해시 | 실행 폴더·로그 링크 | 입력·기대값·실제값 | 해석 |
|---|---|---|---|---|
| 정상 코드 | `d68bd08` | [normal.log](../../evidence/pre/lab2_01_normal.log) | 16 ns 기대 value=1, 실제 value=1. `LAB2_PASS counter4 checks=36` | 모든 검사 통과, 356 ns 종료. |
| 지정한 RTL 변경 | 미커밋 수정본(`d68bd08` 기준, 로컬 실행) | [mod.log](../../evidence/pre/lab2_01_mod.log) | 16 ns 기대 value=1, 실제 value=2. `LAB2_FAIL up including 15 to 0 time=16000`, `FATAL: sim/tb_counter4.sv:12: check failed` | `up including 15 to 0` 검사가 변경을 발견했다(로그의 time은 ps 단위, 16000 ps = 16 ns). |
| 원래 코드로 복구 | `d68bd08` | [recover.log](../../evidence/pre/lab2_01_recover.log) | 복구 후 전체 검사 재실행. `LAB2_PASS counter4 checks=36`, `$finish called at 356000 (1ps)` | PASS와 종료 시각이 정상 실행과 같고 새 VCD를 확인한다. |

- 첫 실패 이후에는 `$fatal`로 시뮬레이션이 끝나므로 뒤의 검사는 실행되지 않는다. 변경 전후 파형은 각각 별도 폴더에 보관한다.
- 문법 오류를 경험했다면 오류 위치로 이동한 화면, 원인, 수정 내용과 재실행 로그도 이 절에 연결한다.

## 보드 실험 계획

- 부품·프로젝트: Vivado 2026.1, RTL Project `lab2_counter`, 부품 `xc7s75fgga484-1`(정확히 -1). Design Sources: `counter4.v`, `input_frontend.v`, `lab2_counter.v`(Copy sources 해제). Simulation Sources: `tb_counter4.sv`(Set as Top: `tb_counter4`). Constraints: `lab2_counter.xdc`. Project Summary의 Top module name은 `lab2_counter`이다.
- 장비 클록·제약: Combo II-DLD S75 주 클록 B6을 1 kHz로 맞추고 XDC의 `trainer_1khz`(1,000,000 ns)와 일치시킨다.
- 입력: K4=리셋, N8=한 번 누름당 한 걸음(press), DIPSW8=`sw[0]`=방향(0 증가, 1 감소). 주 클록은 1 kHz.
- 출력: LED[3:0]=value, LED[7:4]=항상 0.

| 조작 | 예상 LED / 동작 |
|---|---|
| K4 초기화 | LED[3:0]=0 |
| sw[0]=0, N8 한 번 | 1 증가 (LED=0001) |
| 15에서 N8 한 번 | 0으로 순환 |
| sw[0]=1, N8 한 번 | 1 감소, 0에서는 15로 순환 |
| N8를 길게 누름 | 연속 증가하지 않고 한 번만 반영 |

버튼은 프런트엔드의 2단 동기화와 20 ms(=20클록) 안정 확인을 거친 뒤 한 클록짜리 press가 된다. 그래서 누른 뒤 약 20 ms 지나서 반영되고, 손을 떼는 순간에는 press가 생기지 않는다.

촬영할 장면: 보드 전체(배선·입력·출력이 함께 보이는 사진)와 위 표의 조작별 LED 상태 사진·영상. 예상되는 차이: 버튼·스위치는 동기화·안정 확인 지연이 있고, 빠른 신호는 눈으로 구분되지 않을 수 있다.

이 단계에서는 Vivado GUI와 실물 보드의 결과를 수행한 것처럼 기록하지 않는다. 합성·구현·bit 생성과 실제 장치 기록은 실험 후 레포트에서 다룬다.
