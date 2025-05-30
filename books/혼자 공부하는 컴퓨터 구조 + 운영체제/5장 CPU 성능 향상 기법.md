# ■ 05-1 빠른 CPU를 위한 설계 기법

1. **클럭** — 클럭 신호가 얼마나 빠른지?
2. **코어 & 멀티 코어** — 코어가 몇 개인지?
3. **스레드 & 멀티 스레드** — 스레드가 몇 개인지?
## □ 클럭(Clock) 신호를 빠르게 늘리는 방법
컴퓨터 부품들은 클럭 신호에 맞춰 일사불란하게 움직임 → 일반적으로 Yes.

#### 클럭 속도
헤르츠 단위로 측정(Hz)(헤르츠 : 1초에 클럭이 반복되는 횟수)
클럭이 ‘똑-딱-’ 하고 1초에 한 번 반복되면 1Hz(클럭이 1초에 100번 반복되면 100Hz)
```ad-info
실제 CPU의 클럭 속도는... - ex. Intel Core i7(4.9GHz): 1초에 클럭이 기본적으로 25억번, 순간적으로 최대 49억번 반복되는 속도
```
`클럭 신호를 마냥 높이면 CPU는 완전 빨라지나? - *ㄴㄴ.* 필요 이상으로 클럭을 높이면 발열이 심해지기 때문.`
## □ 코어 수를 늘리는 방법 (듀얼 코어, 멀티 코어 등)

#### 코어(Core)
전통적 의미의 CPU : 명령어를 실행하는 부품 (전통적으로 명령어를 실행하는 부품은 원칙적으로 하나만 존재)
현대적 의미의 CPU : 오늘날 CPU에는 명령어를 실행하는 부품이 **여러개 존재**(전통적 의미의 CPU와는 다르게 새롭게 해석)  
‘**명령어를 실행하는 부품**'을 오늘날 코어라고 부름
▸ 멀티 코어(싱글, 듀얼, 트리플, 쿼드, 헥사, 옥타, 데카, 도데카…) : 두 개 이상이면 멀티 코어라 부름
```ad-info
코어를 두 개, 세 개, 100개 늘리면 연산 속도도 비례해서 빨라지나? - 비례하여 증가하ㄴㄴ. 연산이 효율적으로 분배되지 않으면 성능이 비례하여 증가 X
```
## □ 스레드 수를 늘리는 방법 (멀티 스레드 등)
▸ 스레드 : 실행 흐름의 단위
▸ 하드웨어적 스레드 : 하나의 코어가 동시에 처리하는 명령어 단위

`ex. 1코어 1스레드 CPU`
코어 안에 레지스터 세트가 4개(저장하는 곳이 4개~)
`ex. 2코어 4스레드 CPU`
`ex. 멀티스레드 프로세서 or 멀티스레드 CPU`
하나의 코어가 여러 개의 스레드를 지니고 있어 여러 명령어를 동시에 처리하는 것

멀티스레드 프로세서를 실제로 설계하는 일은 매우 복잡. 가장 큰 핵심은 *레지스터*  
하나의 코어로 여러 명령어를 동시에 처리하도록 만들려면 — 하나의 명령어를 처리하기 위해 꼭 필요한 레지스터를 여러 개 가지고 있으면 된다.

#### **하드웨어 스레드** aka. 논리 프로세서(보이지 않는 CPU 개수)  
메모리에 있는 어플리케이션 프로그램 입장에서는 하드웨어 스레드 개수만큼 프로세서가 있다고 생각을 하기 때문. 2코어 4스레드인지 1코어 8스레드인지는 관심 없음.
소프트웨어적 스레드 : 하나의 프로그램에서 독립적으로 실행되는 단위 (최소 작업 단위)
`1코어 1스레드 CPU도 시스템 자원 한도 내에서 여러 개의 소프트웨어적 스레드를 만들 수 있다.`

사실상 1코어당 1개의 스레드를 처리하는 것이지만 속도가 엄청 빠르기 때문에 마치 동시에 여러 스레드를 처리하는 것 처럼 사용자 입장에서는 보이도록 할 수 있음.

  

# ■ 05-2 명령어 병렬 처리 기법

설계 못지않게 중요한 것? ⇢ CPU가 놀지 않고 시간을 효율적으로 사용하여 명령어를 처리하도록 하는 것
1. 명령어 파이프라이닝
2. 슈퍼스칼라
3. 비순차적 명령어 처리

## □ 명령어 파이프라인

#### 명령어가 처리되는 과정을 비슷한 시간(클럭 단위) 간격으로 나누면?
1. 명령어 인출 (Instruction Fetch)
2. 명령어 해석 (Instruction Decode)
3. 명령어 실행 (Execute Instruction)
4. 결과 저장 (Write Back)
`위 단계가 겹치지만 않는다면 CPU는 각 단계를 동시에 실행 가능!`

#### 파이프라인 위험 : 성능 향상에 실패하는 경우
1. **데이터 위험** : 데이터 의존성에 의해 발생
	1. *각 명령어가 끝까지 실행되어야만 다음 단계로 넘어갈* 수 있는 경우
	2. 이런 경우를 무시하고 무작정 동시 실행하려하면 파이프라인이 제대로 작동하지 않는다.
2. **제어 위험**
	1. 프로그램 카운터의 갑작스러운 변화 – 갑작스럽게 메모리 주소 번지로 점프를 하거나 홀드 명령을 하는 경우 등
	2. 이렇게 되면 명령어 파이프라인에 미리 가지고 와서 처리 예정이었던 명령어들이 아무 쓸모 없어진다.
	3. 이를 예측하는 기술을 분기 예측이라고 한다.
3. **구조적 위험**
	1. 명령어를 겹쳐 실행하는 과정에서 서로 다른 명령어가 동시에 ALU, 레지스터 등과 같은 CPU 부품을 사용하려고 할 때 발생
	2. 자원 위험


#### *슈퍼스칼라 (Superscalar)*
CPU 내부에 여러 개의 명령어 파이프라인을 포함한 구조
파이프라인 개수가 늘어나며 성능이 증가하지만, 파이프라인 위험도 역시 함께 증가한다. (완전한 비례적 성능 증가는 아님)

#### *비순차적 명령어 처리* (Out-of-order execution : 명령어간의 합법적인 새치기)
파이프라인 중 데이터 의존성이 없는 것들을 의존성이 있는 것들 사이에 배치하여 CPU 가 효율적으로 연산하도록 하는 처리법
`오늘날 CPU 성능 향상에 크게 기여한 기법이며 대부분의 CPU가 차용중`


# ■ 05-3 명령어 집합 구조, CISC와 RISC

- ISA
- CISC
- RISC

## □ *ISA(Instruction Set Architecture)*
명령어 집합(CPU가 이해할 수 있는 명령어들의 모음)
같은 소스코드로 만들어진 같은 코드라하더라도 ISA 가 다르면 CPU가 이해할 수 있는 명령어도, 어셈블리어도 달라진다. (3장 참조. 명령어를 읽기 편하게 표현한 게 어셈블리어기 때문에 명령어가 다르면 당연히 다름) `ex. 인텔 x86, 애플 ARM`

## □ *CISC (씨스크, Complex Instruction Set Computer)*
복잡한 명령어 집합을 활용하는 컴퓨터(CPU) : x86, x86-64 — CISC 기반
명령어의 형태와 크기가 다양한 가변 길이 명령어를 활용
```ad-advantage
1. 상대적으로 적은 수의 명령어로도 프로그램을 실행할 수 있다. (하나의 명령어가 강력하다고 함)
2. 따라서 메모리를 최대한 아끼며 개발해야 했던 시절 인기가 높았음
```
```ad-disadvantage
1. 명령어 파이프라이닝이 불리하다는 치명적 단점 (파이프라이닝이 가능하려면 가급적 명령어 형태가 정형화되어야 함)
2. 명령어가 워낙 복잡하고 다양한 기능을 제공하는 탓에(강력한 기능)
	1. 명령어의 크기와 실행되기까지의 시간이 일정하지 않음
	2. 명령어 하나를 실행하는 데에 여러 클럭 주기가 필요
	3. 대다수의 복잡한 명령어는 되려 사용 빈도가 낮다.
```
## □ *RISC(리스크, Reduced Instruction Set Computer)*
Simple : 명령어의 종류가 적고, 짧고 규격화됨. 고정 길이 명령어
```ad-advantage
1. 명령어 파이프라이닝에 특화되어 있음.
2. 메모리 접근 최소화하는 대신 레지스터를 십분 활용. (따라서 범용 레지스터의 종류가 다양)
```
```ad-disadvantage
명령어 종류가 CISC보다 적기 때문에 더 많은 명령어로 프로그램을 동작시키게 된다.
```

| CISC                 | RISC                 |
| -------------------- | -------------------- |
| 복잡하고 다양한 명령어         | 단순하고 적은 명령어          |
| 가변길이 명령어             | 고정길이 명령어             |
| 다양한 주소 지정 방식         | 적은 주소 지정 방식          |
| 프로그램을 이루는 명령어의 수가 적음 | 프로그램을 이루는 명령어의 수가 많음 |
| 여러 Clock에 걸쳐 명령어 수행  | 1 Clock 내외로 명령어 수행   |
| 파이프라이닝하기 어려움         | 파이프라이닝하기 쉬움          |

> 참고
현대 CPU에서 명령어 파이프라이닝은 절대 놓칠 수 없는 기술. 따라서 CISC 가 실제로 실행될 때 내부적으로 명령어를 매우  잘게 쪼개어 실행함.(클럭 단위를 쪼개어 파이프라이닝하기 쉬우라고) 그래서 <u>내부적으로는 마치 RISC 처럼 사용</u>하기도 함. 위 비교는 이론적인것임.
