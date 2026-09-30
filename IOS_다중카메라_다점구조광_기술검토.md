# Intraoral Scanner(IOS) 다중 카메라·다점 구조광 광학계 기술 검토

-   작성일: 2026-09-30
-   프로젝트: Intraoral Scanner (IOS)
-   관련 기존 문서: `IOS_개발로드맵_BOM.md`, `IOS_기술검토_보고서.md`
-   핵심 방향: **6-Camera + 3\~5-Projector + Aperiodic Hexagonal Dot
    Pattern + FPGA Synchronization + Multi-view Triangulation**

------------------------------------------------------------------------

## 1. 검토 배경

기존 IOS 개발안은 **스테레오포토그래메트리 + 구조광 하이브리드**를
1단계의 현실적인 출발점으로 설정하였다.

기존 로드맵의 1단계는 다음과 같다.

-   소형 카메라 모듈 2\~3개
-   패턴 프로젝터
-   스테레오 삼각측량
-   구조광 디코딩
-   FPGA 기반 다중 카메라 동기화
-   ARM SoC 기반 3D 재구성

이번 검토에서는 이를 한 단계 확장하여, **카메라를 6개 이상 사용하는 다중
시점 취득 구조**와 **원형 dot을 격자 형태로 다수 투사하는 다점
구조광**을 결합하는 방향을 검토하였다.

핵심적인 질문은 다음과 같다.

1.  왜 2\~3개가 아닌 6개 이상의 카메라를 사용할 수 있는가?
2.  여러 카메라를 사용하면 어떤 광학적·계산적 이점이 있는가?
3.  광원은 단순한 면광원이나 단일 패턴이 아니라 dot pattern을 사용하는
    것이 어떤 의미가 있는가?
4.  dot의 형태, 배열, 밀도, 간격은 어느 정도가 적절한가?
5.  실제 연구·특허·상용 IOS에서 유사한 구조가 확인되는가?
6.  이를 우리 IOS의 광학계와 FPGA/ARM 시스템에 어떻게 연결할 것인가?

------------------------------------------------------------------------

# 2. 6개 이상 카메라를 사용하는 이유

## 2.1 핵심은 카메라 수 자체가 아니라 Multi-view Acquisition

6개의 카메라를 단순히 병렬로 설치하는 것이 목적은 아니다.

각 카메라가 서로 다른 방향에서 동일한 치아 표면을 관측하도록 하여 다음
문제를 줄이는 것이 핵심이다.

-   치아의 가림(occlusion)
-   구치부의 제한된 시야
-   치아 사이의 사각 영역
-   협측/설측/교합면의 동시 취득 한계
-   좁은 구강 내에서의 FOV 제한

개념적으로는 다음과 같다.

``` text
       C1 ↓             ↓ C2

              \       /
               \     /
        C3 ───→ [TOOTH] ←─── C4
               /     \
              /       \

       C5 ↑             ↑ C6
```

한 카메라에서 보이지 않는 표면을 다른 카메라가 관측할 수 있다.

------------------------------------------------------------------------

## 2.2 Multi-camera의 장점

  항목                         2 Camera         6 Camera
  ------------------- ----------------- ----------------
  관측 방향                      제한적           다방향
  Occlusion               상대적으로 큼             감소
  FOV 구성                       제한적   넓게 설계 가능
  Stereo 조합                    제한적        여러 조합
  대응점 redundancy                낮음             높음
  Calibration               비교적 단순             복잡
  MIPI 처리량                      낮음             높음
  FPGA 요구량                      낮음             높음
  3D fusion             상대적으로 단순             복잡
  발열/전력                        낮음             증가
  광학 정렬 난이도                 낮음             높음

중요한 점은 **카메라 수를 늘린다고 정확도가 자동으로 증가하는 것은
아니라는 것**이다.

카메라가 많아지면 오히려 다음 문제가 중요해진다.

-   Multi-camera intrinsic calibration
-   Camera-to-camera extrinsic calibration
-   시간 동기화
-   노출 동기화
-   렌즈 왜곡 보정
-   공통 3D 좌표계 변환
-   다중 point-cloud fusion

따라서 6-camera 시스템은 본질적으로 **광학계 + 동기화 + 영상처리 + 3D
재구성 시스템**으로 접근해야 한다.

------------------------------------------------------------------------

# 3. 6 Camera + Multi-projector 구조

다중 카메라 구조를 구조광과 결합하면 다음과 같은 형태가 된다.

``` text
             C1        C2

                 P1
          ● ● ● ● ● ●
        ● ● ● ● ● ● ●
          ● ● ● ● ● ●
                 P2

        C3      TOOTH      C4

                 P3
          ● ● ● ● ● ●
        ● ● ● ● ● ● ●
          ● ● ● ● ● ●

             C5        C6
```

여기서 중요한 것은 각 projector의 illumination field와 camera의 FOV가
서로 충분히 겹쳐야 한다는 점이다.

공개된 관련 특허에서는 여러 structured-light projector와 여러 camera를
rigid structure에 배치하고, projector 패턴과 인접 camera FOV의 overlap을
크게 확보하는 구조가 설명된다.

------------------------------------------------------------------------

# 4. 광원: 원형 Dot Pattern

이번 검토에서 중요한 아이디어는 **단일 선(line), 단일 점(point), 전체
면(pattern image)**이 아니라,

> **작은 원형 광점(dot)을 다수 생성하여 2차원 패턴으로 투사하는 방식**

이다.

개념은 다음과 같다.

``` text
●   ●   ●   ●   ●
  ●   ●   ●   ●
●   ●   ●   ●   ●
  ●   ●   ●   ●
●   ●   ●   ●   ●
```

각 dot의 위치가 치아 표면에서 변형되는 것을 camera가 관측하고, 이를
이용하여 3D 정보를 계산한다.

------------------------------------------------------------------------

# 5. 단순 Grid보다 Hexagonal / Aperiodic Pattern이 중요한 이유

## 5.1 정사각형 periodic grid

가장 단순한 형태는 다음과 같다.

``` text
● ● ● ● ●
● ● ● ● ●
● ● ● ● ●
● ● ● ● ●
```

이 구조는 제작과 해석이 단순하지만, 반복 패턴이 많아 correspondence를
찾을 때 주변 패턴이 서로 유사해질 수 있다.

------------------------------------------------------------------------

## 5.2 Hexagonal / Honeycomb 배열

보다 효율적인 2D packing을 위해 다음과 같은 hexagonal 배열을 사용할 수
있다.

``` text
●   ●   ●   ●
  ●   ●   ●   ●
●   ●   ●   ●
  ●   ●   ●   ●
```

육각 배열은 동일한 최소 이웃거리에서 점을 효율적으로 배치할 수 있다는
장점이 있다.

------------------------------------------------------------------------

## 5.3 Aperiodic / Quasi-random Dot Pattern

더 발전된 형태는 완전히 규칙적인 hexagonal grid에서 각 dot 위치를 조금씩
변형시키는 방식이다.

``` text
●     ●  ●      ●
   ●      ● ●
●   ●   ●      ●
   ● ●       ●
●       ●  ●    ●
```

중요한 점은 **완전히 랜덤하게 만드는 것이 아니라, 기본적인 육각 배열을
유지하면서 각 점을 약간 이동시키는 것**이다.

이렇게 하면 local sub-pattern이 더 독특해져서 카메라 영상에서 특정 dot
주변의 패턴을 식별하기 쉬워질 수 있다.

관련 특허에서는 honeycomb/hexagonal 구조를 기본으로 하고 microlens
중심을 일정 범위 내에서 비주기적으로 이동시키는 개념이 설명되어 있다.

------------------------------------------------------------------------

# 6. Dot Pattern의 광학 생성 방법

가능한 구조는 크게 다음과 같다.

  ----------------------------------------------------------------------------
  방식           광원           패턴 생성      장점           단점
  -------------- -------------- -------------- -------------- ----------------
  A              Laser diode    Microlens      높은 광량,     Speckle 가능
                                Array          구조 단순      

  B              Laser / VCSEL  DOE            패턴 설계      DOE 제작 필요
                                               자유도 높음    

  C              LED/VCSEL      MLA            다채널화 가능  광량/균일성 문제
                 array                                        

  D              LED/Laser      DLP/DMD        패턴 변경 가능 부피/전력/가격
                                                              증가
  ----------------------------------------------------------------------------

초기 IOS prototype에서는 **Laser diode 또는 VCSEL + MLA/DOE**를 우선
검토할 가치가 있다.

------------------------------------------------------------------------

# 7. 예상 광학 경로

하나의 projector를 다음과 같이 구성할 수 있다.

``` text
Laser diode / VCSEL
        │
        ▼
Beam shaping
        │
        ▼
Rod lens / Homogenizer
        │
        ▼
Mirror / Fold optics
        │
        ▼
Microlens Array / DOE
        │
        ▼
Aperiodic Hexagonal Dot Pattern
        │
        ▼
Dental Surface
```

카메라 쪽에서는

``` text
Dental Surface
       ↓
Multiple Cameras
       ↓
MIPI CSI-2
       ↓
FPGA
       ↓
Frame Sync / Pre-processing
       ↓
ARM SoC
       ↓
Multi-view Triangulation
       ↓
Point Cloud Fusion
       ↓
Mesh Reconstruction
```

의 구조를 생각할 수 있다.

------------------------------------------------------------------------

# 8. 관련 자료에서 확인되는 Microlens 수치

관련 특허 자료에서는 다음과 같은 microlens density 범위가 제시된다.

-   약 **1,200\~6,000개/cm²**
-   이를 mm² 기준으로 환산하면 약 **12\~60개/mm²**

육각 배열을 단순화하여 pitch를 추정하면 대략 다음 범위가 된다.

    Microlens density   단순 환산 pitch
  ------------------- -----------------
            1,200/cm²        약 0.31 mm
            2,000/cm²        약 0.24 mm
            3,000/cm²        약 0.20 mm
            4,000/cm²        약 0.17 mm
            6,000/cm²        약 0.14 mm

**주의:** 위 값은 microlens array의 pitch에 대한 근사치이며, 실제 치아
표면에 형성되는 dot pitch와 동일하지 않다.

------------------------------------------------------------------------

# 9. Microlens Pitch와 실제 Dot Pitch의 관계

간단한 paraxial model에서는 다음과 같이 생각할 수 있다.

\[ `\theta `{=tex}`\approx `{=tex}`\frac{p}{f}`{=tex} \]

그리고 working distance가 충분히 단순화된 조건에서는

\[ P\_{object}`\approx `{=tex}WD`\frac{p}{f}`{=tex} \]

로 근사할 수 있다.

예를 들어,

``` text
Microlens pitch p = 0.20 mm
Microlens focal length f = 5 mm
Working distance WD = 10 mm
```

라면

\[ P\_{object}`\approx10`{=tex}`\times`{=tex}`\frac{0.20}{5}`{=tex}
=0.40mm \]

정도의 값이 된다.

WD가 20 mm이면 약 0.8 mm가 된다.

따라서 **MLA pitch와 실제 치아 표면에서의 dot spacing은 별개의
설계변수**로 취급해야 한다.

실제 설계에서는 다음 요소를 포함한 OpticStudio/Zemax 모델이 필요하다.

-   Beam divergence
-   MLA pitch
-   MLA focal length
-   Projector aperture
-   Working distance
-   Projection angle
-   Surface curvature
-   Lens distortion
-   Dot intensity
-   Camera pixel pitch

------------------------------------------------------------------------

# 10. Dot Diameter와 Dot Pitch

Dot pattern에서는

> **Dot diameter ≠ Dot pitch**

라는 점이 중요하다.

예를 들어,

``` text
Pitch = 0.5 mm

      ●
      │
  ────┼────
      │
      ●

Dot diameter = 0.08~0.15 mm
```

처럼 충분한 separation을 확보할 수 있다.

Dot이 지나치게 크면 인접 dot이 합쳐질 수 있고,

``` text
●●●●
●●●●
```

반대로 지나치게 작으면 camera noise나 반사 때문에 검출이 어려워질 수
있다.

따라서 다음 비율을 독립적인 설계 변수로 관리하는 것이 좋다.

\[ R\_{dot}=`\frac{D_{dot}}{P_{dot}}`{=tex} \]

------------------------------------------------------------------------

# 11. 구강 환경에서 Dot Pattern이 가지는 의미

IOS에서는 다음과 같은 표면을 동시에 처리해야 한다.

-   법랑질
-   상아질
-   금속 보철
-   레진
-   습윤 표면
-   타액
-   매우 밝은 반사 영역
-   어두운 영역
-   좁은 fissure

따라서 단순한 조명 균일성보다 **패턴의 검출 가능성**이 중요하다.

고려해야 할 요소는 다음과 같다.

1.  높은 반사율
2.  specular reflection
3.  타액에 의한 highlight
4.  투명/반투명 영역
5.  주변 외광
6.  광원의 wavelength
7.  camera의 spectral response
8.  laser speckle
9.  dot intensity uniformity

------------------------------------------------------------------------

# 12. 6 Camera + Dot Pattern의 핵심 장점

6개의 camera가 여러 방향에서 같은 dot pattern을 관측하면 다음과 같은
구조가 가능하다.

``` text
              Projector
                  ↓
       ● ● ● ● ● ● ● ●
     ● ● ● ● ● ● ● ● ●
       ● ● ● ● ● ● ●
                  ↓
                TOOTH

       C1 ────────┐
       C2 ────────┤
       C3 ────────┼──→ Dot correspondence
       C4 ────────┤
       C5 ────────┤
       C6 ────────┘
```

즉 하나의 dot을 여러 카메라에서 관측할 수 있으므로 **correspondence
redundancy**가 생긴다.

이를 이용하여:

-   Multi-view triangulation
-   Depth estimation
-   Outlier rejection
-   Point-cloud fusion
-   Surface reconstruction

을 수행할 수 있다.

------------------------------------------------------------------------

# 13. Multi-projector의 의미

Projector를 하나만 사용하는 대신 3\~5개 정도로 분산할 수 있다.

예:

``` text
        C1       C2

           P1
           ↓
      ● ● ● ● ●

    P2 ↓       ↓ P3

        TOOTH

    P4 ↓       ↓ P5

        C3       C4
```

또는 시간적으로 projector를 분리할 수 있다.

``` text
Frame 0 : P1 ON → C1~C6 capture
Frame 1 : P2 ON → C1~C6 capture
Frame 2 : P3 ON → C1~C6 capture
Frame 3 : P4 ON → C1~C6 capture
Frame 4 : P5 ON → C1~C6 capture
```

이 방식은 FPGA가 담당하기 좋은 영역이다.

------------------------------------------------------------------------

# 14. FPGA/ARM 시스템과의 연결

현재 프로젝트의 하드웨어 경험을 고려하면 다음 구조가 적합하다.

``` text
              6 × Camera
                  │
             MIPI CSI-2
                  │
        ┌─────────▼─────────┐
        │        FPGA       │
        │                   │
        │ Frame Sync        │
        │ Exposure Sync     │
        │ Trigger Control   │
        │ Image Preprocess  │
        │ Dot Detection     │
        │ Pattern Decode    │
        └─────────┬─────────┘
                  │
              DDR / AXI
                  │
        ┌─────────▼─────────┐
        │      ARM SoC      │
        │                   │
        │ Stereo/Multi-view │
        │ Triangulation     │
        │ ICP               │
        │ Point Cloud       │
        │ Mesh              │
        └─────────┬─────────┘
                  │
               USB 3.0
                  │
             PC / Tablet
```

특히 FPGA에서는 다음 기능을 먼저 구현할 수 있다.

-   6-camera hardware trigger
-   Frame synchronization
-   Exposure synchronization
-   LED/Laser/Projector timing
-   MIPI frame routing
-   ROI extraction
-   Dot detection
-   Pattern decoding
-   Timestamping

------------------------------------------------------------------------

# 15. 초기 Prototype 제안

아직 최종 설계값으로 확정하지 않고, 광학 시뮬레이션을 시작하기 위한
**Design Target**으로 다음 정도를 검토한다.

  항목                      Prototype-A             Prototype-B
  ------------------ ------------------ -----------------------
  Camera                              6                       6
  Projector                           3                       5
  광원                       Blue Laser              Blue Laser
  파장 후보                      450 nm         405/450 nm 비교
  MLA focal length              4\~6 mm                 4\~6 mm
  MLA density          1,200\~3,000/cm²        3,000\~6,000/cm²
  MLA pitch            약 0.31\~0.20 mm        약 0.20\~0.14 mm
  배열                        Hexagonal   Hexagonal + Aperiodic
  Pattern                        규칙형               Aperiodic
  Working distance       5\~20 mm sweep          5\~25 mm sweep
  Camera FPS                    ≥60 fps            ≥75\~120 fps
  검증                           Stereo       Multi-view fusion

**주의:** 위 수치는 제품 사양으로 확정된 값이 아니라 초기 광학
시뮬레이션을 위한 후보 범위이다.

------------------------------------------------------------------------

# 16. 특허/IP 검토 포인트

이번 조사에서 IOS의 다중 카메라 및 dot-pattern 구조와 직접적으로 관련된
특허들이 확인된다.

### 16.1 Sopro 계열

관련 기술 요소:

-   Microlens array
-   Dot pattern
-   Non-periodic / aperiodic pattern
-   Active stereo
-   Multiple camera
-   Intraoral scanner

특히 honeycomb/hexagonal 기반의 dot 배열과 dot 위치를 비주기적으로
변형하는 개념이 중요하다.

### 16.2 Align / iTero 계열

관련 기술 요소:

-   Multiple cameras
-   Multiple structured-light projectors
-   FOV overlap
-   Multi-camera / multi-projector architecture
-   iTero Lumina 계열의 다중 취득 구조

### 16.3 개발 시 주의사항

따라서 다음과 같은 구성을 그대로 제품화하는 것은 피해야 한다.

> **6-camera + 5-projector + 특정 hexagonal dot pattern**

특허의 권리범위는 단순히 부품 수만으로 판단할 수 없으므로, 실제 제품화
단계에서는 특허 청구항과 출원일, 우선권, 존속 여부 및 회피설계를 별도로
검토해야 한다.

------------------------------------------------------------------------

# 17. 우리 프로젝트에서의 권장 방향

현재까지의 검토를 종합하면 기존의

> **2\~3 Camera + Structured Light**

구조를 폐기하기보다는 다음과 같이 확장하는 것이 적절하다.

``` text
기존 1단계
Stereo Camera
      +
Structured Light
      ↓
3D Reconstruction

             ↓ 확장

차세대 Prototype
6 Camera
   +
3~5 Projector
   +
Aperiodic Hexagonal Dot Pattern
   +
FPGA Synchronization
   +
Multi-view Triangulation
   ↓
High-density Point Cloud
   ↓
Surface Reconstruction
```

핵심 연구 과제는 다음 6개로 정리할 수 있다.

1.  **Camera geometry**
    -   6 camera의 위치와 viewing angle 최적화
2.  **Projector geometry**
    -   3\~5 projector의 위치와 FOV 최적화
3.  **Dot pattern**
    -   Circular dot
    -   Hexagonal arrangement
    -   Aperiodic perturbation
    -   Dot diameter / pitch 최적화
4.  **Optical engine**
    -   Laser/VCSEL
    -   Rod lens
    -   MLA 또는 DOE
    -   Fold mirror
5.  **Real-time electronics**
    -   FPGA
    -   MIPI CSI-2
    -   Camera synchronization
    -   Projector synchronization
6.  **3D reconstruction**
    -   Multi-view correspondence
    -   Triangulation
    -   Outlier rejection
    -   ICP
    -   Point-cloud fusion
    -   Mesh reconstruction

------------------------------------------------------------------------

# 18. 향후 광학 시뮬레이션에서 결정해야 할 변수

다음 단계에서는 실제 제품 이미지를 그리는 것보다 먼저 아래 변수를
수치화하는 것이 좋다.

``` text
Camera
 ├─ Sensor size
 ├─ Pixel pitch
 ├─ Resolution
 ├─ Lens FOV
 ├─ Working distance
 └─ Baseline

Projector
 ├─ Wavelength
 ├─ Optical power
 ├─ Beam divergence
 ├─ MLA pitch
 ├─ MLA focal length
 ├─ Dot diameter
 ├─ Dot pitch
 ├─ Pattern size
 └─ Projection angle

System
 ├─ Camera-projector baseline
 ├─ FOV overlap
 ├─ Depth range
 ├─ Expected Z resolution
 ├─ Frame rate
 ├─ Exposure time
 └─ Illumination sequence
```

특히 **Dot Pitch / Dot Diameter / Camera Pixel Size / Working Distance /
Stereo Baseline**의 관계를 먼저 계산해야 한다.

------------------------------------------------------------------------

# 19. 최종 개념

현재까지 논의한 IOS의 방향은 다음과 같이 정리할 수 있다.

> ## **6-Camera Multi-view + Multi-projector Aperiodic Dot Structured Light IOS**

``` text
             ┌──────────────────────┐
             │ C1    P1 P2    C2   │
             │                      │
             │ P3   DOT PATTERN  P4 │
             │                      │
             │ C3    P5      C4     │
             │                      │
             │      C5    C6        │
             └──────────────────────┘
                       ↓
                    TOOTH
                       ↓
             Multi-view Acquisition
                       ↓
               FPGA Synchronization
                       ↓
              ARM 3D Reconstruction
                       ↓
                Point Cloud / Mesh
```

최종적으로는 단순한 "구강 카메라"가 아니라,

> **다중 시점 광학 취득 + 능동 구조광 + FPGA 실시간 동기화 + 3D
> reconstruction**

을 하나의 플랫폼으로 개발하는 방향이다.

------------------------------------------------------------------------

# 20. 기존 IOS 개발 로드맵과의 연결

기존 로드맵은 다음과 같이 유지하면서 확장할 수 있다.

  -----------------------------------------------------------------------
  단계                    기존 방향               확장 방향
  ----------------------- ----------------------- -----------------------
  1단계                   Stereo + Structured     **6 Camera + 3\~5
                          Light                   Projector + Dot
                                                  Pattern**

  2단계                   Line Scan + Variable    Multi-view + 고정밀
                          Focus                   depth refinement

  3단계                   PZT MEMS mirror         초소형 optical engine
  -----------------------------------------------------------------------

기존 문서의 2단계는 가변초점 렌즈와 라인스캔을 이용하여 공초점 원리를
부분적으로 도입하는 방향이며, 3단계는 PZT MEMS mirror를 이용한 소형화를
목표로 한다.

따라서 이번 검토는 기존 로드맵을 변경한다기보다 **1단계의 광학 취득
구조를 6-camera/multi-projector architecture로 고도화하는 제안**으로
보는 것이 적절하다.

------------------------------------------------------------------------

## 참고 자료 / 추가 검토 대상

-   Align Technology / iTero Lumina 관련 공개 자료
-   Align Technology 관련 multi-camera / multi-projector 특허
-   Sopro 관련 microlens-array / dot-pattern / active-stereo 특허
-   Intraoral Scanner의 structured-light, stereophotogrammetry 및
    confocal acquisition 관련 연구문헌

> **주의:** 본 문서의 특허 관련 내용은 기술 검토를 위한 요약이며, 법률적
> 특허 침해 또는 회피설계 판단을 의미하지 않는다. 또한 Prototype-A/B의
> 수치는 초기 광학 설계를 위한 후보값이며 실제 부품 선정 전에 광학
> 시뮬레이션과 실험 검증이 필요하다.
