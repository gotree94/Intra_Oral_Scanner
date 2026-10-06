Unreal Engine 구강 스캐너 시뮬레이션용 Dental Scan 데이터 조사
1. 목적

Unreal Engine을 이용하여 구강 스캐너(Intraoral Scanner) 시뮬레이션을 개발하기 위해 실제 Dental Scan 데이터를 확보한다.

단순한 3D Mouth 모델보다는 실제 intraoral scanner로 취득된 STL / PLY / OBJ 데이터를 사용하는 것이 실제 스캔 과정을 시뮬레이션하는 데 적합하다.

주요 활용 목적:

실제 치아 표면을 Ground Truth로 활용

가상 intraoral scanner 구현

Raycast / Depth 기반 스캔 시뮬레이션

Point Cloud 생성

Scan Mesh 재구성

실제 Scan 데이터와 시뮬레이션 결과 비교

치아별 segmentation 및 개별 치아 조작

다양한 환자/교합 상태를 이용한 Scanner Training Simulator 구현

2. 추천 데이터셋

현재 목적에 적합한 데이터셋은 다음과 같다.

데이터셋	포맷	규모	주요 특징	Unreal 적합성
Teeth3DS	OBJ + JSON	1,800 scans / 900명	실제 intraoral scan, 치아별 annotation	⭐⭐⭐⭐⭐
3Shape FDI 16	PLY	7,732 meshes/point clouds	실제 TRIOS scan	⭐⭐⭐⭐⭐
Bits2Bites	STL	200쌍	실제 상·하악 및 교합 관계	⭐⭐⭐⭐⭐
3. Teeth3DS
3.1 개요

Teeth3DS는 MICCAI 2022 3DTeethSeg Challenge를 위해 만들어진 실제 dental scan 데이터셋이다.

약 900명의 환자

총 1,800개의 intraoral scan

Upper jaw / Lower jaw 제공

실제 intraoral scan 기반

OBJ mesh 제공

JSON 형태의 tooth segmentation 정보 제공

데이터셋:

https://osf.io/xctdy/

GitHub:

https://github.com/abenhamadou/3DTeethSeg_MICCAI_Challenges

3.2 데이터 구조

대략적으로 다음과 같은 구조로 사용할 수 있다.

Patient_XXXX
├── upper.obj
├── upper.json
├── lower.obj
└── lower.json


JSON에는 각 vertex가 어느 치아 또는 gingiva에 속하는지에 대한 segmentation 정보가 포함된다.

예:

vertex
 ├── gingiva
 ├── tooth 11
 ├── tooth 12
 ├── tooth 13
 ├── ...
 └── tooth 48


이 구조를 이용하면 실제 dental scan을 치아별로 분리할 수 있다.

3.3 Unreal Engine 활용

Teeth3DS의 가장 큰 장점은 실제 scan mesh + tooth segmentation을 동시에 활용할 수 있다는 것이다.

예를 들어 Python을 이용하여 다음과 같이 전처리할 수 있다.

Teeth3DS
    │
    ├── upper.obj
    ├── lower.obj
    │
    └── labels.json
           │
           ▼
    Python preprocessing
           │
           ▼
    ┌─────────────────────┐
    │ Tooth_11.obj        │
    │ Tooth_12.obj        │
    │ Tooth_13.obj        │
    │ ...                 │
    │ Tooth_48.obj        │
    │ Gingiva.obj         │
    └─────────────────────┘
           │
           ▼
      Unreal Engine


이렇게 전처리하면 Unreal Engine에서 개별 치아를 쉽게 제어할 수 있다.

예:

Tooth_11
Tooth_12
Tooth_13
...
Tooth_48

4. 3Shape FDI 16
4.1 개요

3Shape FDI 16은 실제 3Shape TRIOS intraoral scanner를 이용하여 취득된 anonymized intraoral scan 데이터셋이다.

데이터셋:

https://data.dtu.dk/articles/dataset/3Shape_FDI_16_Meshes_from_Intraoral_Scans/23626650

주요 특징:

실제 intraoral scan

3Shape TRIOS 기반

TRIOS 3 사용 데이터 포함

실제 임상 데이터

PLY 형식

총 7,732개의 mesh / point cloud

약 6.67 GB

Train / Validation / Test 데이터가 구분되어 있다.

4.2 장점

이 데이터셋은 실제 intraoral scanner가 생성하는 mesh의 특성을 확인하는 데 적합하다.

특히 다음 연구에 유용하다.

실제 scan mesh 분석

Point Cloud 분석

Mesh reconstruction

Scan quality 분석

Scanner noise simulation

실제 데이터와 simulated scan 비교

5. Bits2Bites
5.1 개요

Bits2Bites는 실제 intraoral scan pair를 제공하는 비교적 최신 dental dataset이다.

웹사이트:

https://ditto.ing.unimore.it/bits2bites/

주요 특징:

200명의 registered intraoral scan pair

Upper / Lower jaw

STL 형식

Carestream 및 3Shape TRIOS scanner 사용

실제 교합 관계 유지

평균 약 92,000 vertices

평균 약 182,000 faces

5.2 데이터 구조

예상 구조:

Patient_001
├── upper.stl
└── lower.stl

Patient_002
├── upper.stl
└── lower.stl

...


상악과 하악의 관계가 유지되어 있기 때문에 Occlusion simulation에 특히 유용하다.

6. 데이터셋 선택 기준

개발 목적에 따라 다음과 같이 선택할 수 있다.

실제 Scanner 데이터를 분석하고 싶은 경우
추천: 3Shape FDI 16
실제 TRIOS
    ↓
실제 intraoral scan
    ↓
PLY
    ↓
Unreal Engine


실제 scanner가 생성하는 mesh의 특성을 분석하기 좋다.

Scanner Simulator를 개발하는 경우
추천: Teeth3DS
실제 intraoral scan
        ↓
       OBJ
        ↓
Tooth segmentation
        ↓
개별 치아 인식
        ↓
Unreal Engine
        ↓
Scanner simulation


특히 치아별 annotation이 있기 때문에 가장 활용도가 높다.

상악/하악 교합을 시뮬레이션하는 경우
추천: Bits2Bites
Upper Jaw
    │
    │ Occlusion
    │
Lower Jaw


실제 상·하악 관계를 이용한 교합 시뮬레이션에 적합하다.

7. Unreal Engine용 추천 아키텍처

단순히 dental scan mesh를 Unreal Engine에 넣는 것보다 원본 Ground Truth와 Scan Result를 분리하는 것을 추천한다.

                   Dental Scan
                       │
                       ▼
               Ground Truth Mesh
                       │
                       ▼
              ┌─────────────────┐
              │ Virtual Scanner │
              └─────────────────┘
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
            FOV      Raycast   Depth
             │         │         │
             └─────────┼─────────┘
                       ▼
                 Scan Point Cloud
                       │
                       ▼
              Accumulated Points
                       │
                       ▼
                Mesh Reconstruction
                       │
                       ▼
                  Scan Result
                       │
                       ▼
              Compare with Ground Truth

8. 전체 개발 흐름

추천하는 개발 흐름은 다음과 같다.

[1] Dental Scan 데이터 확보
          ↓
[2] Python 전처리
          ↓
[3] 치아별 Mesh 분리
          ↓
[4] OBJ / FBX 변환
          ↓
[5] Unreal Engine Import
          ↓
[6] Virtual Scanner 구현
          ↓
[7] Raycast / Depth Simulation
          ↓
[8] Point Cloud 생성
          ↓
[9] Scan Point 누적
          ↓
[10] Mesh Reconstruction
          ↓
[11] Ground Truth와 비교

9. Unreal Engine에서의 권장 데이터 구조

예를 들어 Unreal 프로젝트를 다음과 같이 구성할 수 있다.

Content/
│
├── Dental/
│   ├── Patients/
│   │   ├── Patient_001/
│   │   │   ├── Tooth_11
│   │   │   ├── Tooth_12
│   │   │   ├── Tooth_13
│   │   │   └── ...
│   │   │
│   │   └── Patient_002/
│   │
│   └── Materials/
│       ├── ToothMaterial
│       ├── GumMaterial
│       └── TongueMaterial
│
├── Scanner/
│   ├── BP_IntraoralScanner
│   ├── ScannerHead
│   ├── ScannerCamera
│   └── ScannerSensor
│
├── Simulation/
│   ├── ScanManager
│   ├── ScanPointCloud
│   ├── ScanMesh
│   └── ScanReconstruction
│
└── UI/
    ├── ScannerUI
    ├── PatientSelector
    └── ScanProgress

10. Virtual Scanner 시뮬레이션

가상 Scanner는 다음과 같은 요소를 갖도록 설계할 수 있다.

Virtual Scanner
│
├── Camera
│
├── FOV
│
├── Working Distance
│
├── Tracking
│
├── Depth Sensor
│
├── Scan Noise
│
├── Occlusion
│
└── Motion


스캐너가 치아 표면을 바라보면:

Scanner
   │
   │ Raycast
   ▼
Tooth Surface
   │
   ▼
Depth Point
   │
   ▼
Point Cloud


스캐너를 움직이면 새로운 point가 계속 누적된다.

Scan #1
   ↓
Point Cloud A

Scan #2
   ↓
Point Cloud A + B

Scan #3
   ↓
Point Cloud A + B + C

...


최종적으로:

Accumulated Point Cloud
          ↓
Mesh Reconstruction
          ↓
Final Scan Mesh


를 구현한다.

11. Ground Truth 비교

실제 Dental Scan을 Ground Truth로 사용할 수 있다.

                 Ground Truth
                      │
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
      Original Mesh      Simulated Scan
                               │
                               ▼
                         Scan Mesh
                               │
                               ▼
                         Comparison


비교 가능한 지표:

Point-to-point distance

Hausdorff distance

RMSE

Surface deviation

Missing area

Scan coverage

Scan time

Point density

예를 들어 Unreal에서 시각화할 수도 있다.

Green  = 정확하게 스캔됨
Yellow = 오차가 있음
Red    = 큰 오차
Black  = 아직 스캔되지 않음


이렇게 하면 단순한 시각화 프로그램이 아니라 Scanner algorithm을 평가하는 시뮬레이터로 발전시킬 수 있다.

12. 라이선스 주의사항

데이터셋을 실제 제품이나 상업용 시뮬레이터에 사용할 경우에는 반드시 각각의 라이선스를 확인해야 한다.

특히 Teeth3DS는 CC BY-NC-ND 4.0으로 명시되어 있으므로 연구 및 비상업적 사용에는 유용하지만, 상업 제품에 데이터를 포함하는 경우에는 별도의 라이선스 검토가 필요하다.

따라서 프로젝트가 상업용이라면 다음 순서로 검토하는 것이 좋다.

Dataset 확보
      ↓
License 확인
      ↓
Commercial use 가능 여부
      ↓
Redistribution 가능 여부
      ↓
Derivative work 가능 여부
      ↓
Unreal 프로젝트에 포함 가능 여부

13. 최종 추천

현재 Unreal 기반 구강 스캐너 시뮬레이션을 개발한다는 목적을 기준으로 하면 다음 순서를 추천한다.

1순위 — Teeth3DS

이유:

실제 intraoral scan

OBJ

1,800 scans

900명

tooth segmentation 제공

치아별 분석 가능

Unreal에서 Ground Truth로 사용하기 좋음

2순위 — 3Shape FDI 16

이유:

실제 TRIOS scan

PLY

실제 scanner 결과 분석에 적합

대규모 데이터

3순위 — Bits2Bites

이유:

STL

실제 상·하악

교합 관계

서로 다른 scanner 데이터

Occlusion simulation에 적합

14. 다음 개발 단계

가장 현실적인 첫 번째 Prototype은 다음과 같이 구성하는 것을 추천한다.

Teeth3DS
   ↓
OBJ + JSON
   ↓
Python preprocessing
   ↓
치아별 Mesh 분리
   ↓
Unreal Engine 5.x
   ↓
Dental Arch 배치
   ↓
Virtual Intraoral Scanner
   ↓
Raycast / Depth Simulation
   ↓
Point Cloud
   ↓
Scan accumulation
   ↓
Scan Mesh
   ↓
Ground Truth와 비교


이 구조를 먼저 구현하면 이후에는 다음 기능을 단계적으로 추가할 수 있다.

Scanner FOV

Scanner tip collision

Scanner movement

Tracking error

Depth noise

Missing data

Occlusion

Scan stitching

Scan path optimization

AI-based scan quality evaluation

실제 scanner와 유사한 UI

Training mode

환자/케이스 선택

실시간 scan coverage 표시

참고 링크

Teeth3DS: https://osf.io/xctdy/

Teeth3DS GitHub: https://github.com/abenhamadou/3DTeethSeg_MICCAI_Challenges

3Shape FDI 16: https://data.dtu.dk/articles/dataset/3Shape_FDI_16_Meshes_from_Intraoral_Scans/23626650

Bits2Bites: https://ditto.ing.unimore.it/bits2bites/