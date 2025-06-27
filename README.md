# 프로젝트 기술 설계서: Amazing Core

## 📘 보자마자 피트니스 - 어메이징 코어 (Amazing Core)

![대표 이미지](https://github.com/JISUSAMA/JISUSAMA/assets/38304918/35dc59a4-294a-4c78-a4a8-202fdb85e399)

‘어메이징 코어’는 Unity 기반으로 제작된 **홈 피트니스 운동 콘텐츠 앱**입니다.
사용자가 BLE 센서를 착용하고 실제 코어 운동을 수행하면, 실시간 데이터에 따라 트레이닝/게임 콘텐츠가 동작합니다.

\*\*게임 요소(Gamification)\*\*와 \*\*운동 루틴(Training System)\*\*을 결합해 사용자 몰입도와 지속성을 높이는 것을 목표로 개발되었습니다.

---

## 📅 개발 기간

* **2022년 10월 \~ 2023년 3월 (6개월)**
* 팀원 3인 협업: 기획 1, 디자인 1, 개발 1
* **기술 총괄 및 콘텐츠 개발 100% 담당**

---

## 🛠️ 기술 스택 및 구조

| 영역     | 기술                         | 설명                              |
| ------ | -------------------------- | ------------------------------- |
| 게임 엔진  | Unity3D (C#)               | 전체 콘텐츠 시스템 설계 및 UI/UX 흐름 구현     |
| 영상 제어  | VideoPlayer, RenderTexture | 튜토리얼/운동 영상 재생 및 타이머 제어          |
| 센서 연동  | BLE (ESP32)                | 코어 자세/움직임 인식, Unity ↔ 외부 기기 연동  |
| UI 연출  | Unity UI, DOTween          | 슬라이더, 깜빡임 효과, 버튼 트랜지션 등 인터랙션 적용 |
| 데이터 저장 | PlayerPrefs                | 운동 횟수, 레벨, 프로필 정보 저장 및 불러오기     |
| 멀티랭귀지  | Unity Localization         | 한/영 텍스트 자동 변경, 언어 선택 기능 구현      |
| 3D 뷰어  | Cinemachine, Pinch Zoom    | 3D 모델 프레임 재생 및 카메라 조작 구현        |

---

## 💼 주요 기여

### 🎨 UI/UX 콘텐츠 설계 및 적용

* 메인 메뉴, 운동 모드, 게임 모드, 튜토리얼, 결과 페이지 등 전체 화면 설계 및 스크립트 적용
* DOTween으로 자연스러운 인터랙션 연출 (버튼 강조, 슬라이더 증가 등)
* PlayerPrefs를 활용한 **자동 로그인 기능** 및 **운동 이력 유지** 로직 구현

| 메인 화면                                                                                             | 트레이닝 진행                                                                                           | 게임 선택                                                                                             |
| ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ![UI1](https://github.com/JISUSAMA/JISUSAMA/assets/38304918/7619de91-8280-44f7-99a4-0d5a9cee3220) | ![UI2](https://github.com/JISUSAMA/JISUSAMA/assets/38304918/c5c5ca44-5010-4343-a0e5-11b4806658f9) | ![UI3](https://github.com/JISUSAMA/JISUSAMA/assets/38304918/bf41bb63-7ec7-498d-927d-a52ccc14d47b) |

### 🧠 트레이닝 콘텐츠 시스템

* 난이도별 3단계 루틴(초급/중급/고급) 반복 구성 (1/2/3회)
* 영상 설명 → 튜토리얼 → 본 운동 → 결과 요약으로 이어지는 **운동 흐름 구조**
* `VideoHandle.cs`를 통한 RawImage 기반 영상 재생 관리 및 상태 제어

```csharp
// 트레이닝 루틴 제어 구조
public void StartTrainingRoutine()
{
    ShowDescriptionUI();
    PlayTutorialClip();
    PlayMainWorkout();
    ShowClearPopup();
}
```

* 각 단계는 `Training_AppManager.cs`에서 Coroutine 기반 흐름으로 구성되며,
  `Training_UIManager.cs`는 타이머, 음성 카운트, 버튼 인터랙션 등을 전담합니다.

---

### 🎮 게임 콘텐츠 모듈 개발

총 3가지 미니 게임 콘텐츠를 직접 구현:

| 게임명          | 컨셉    | 핵심 로직                     |
| ------------ | ----- | ------------------------- |
| Tower Maker  | 블럭 쌓기 | 높이 기반 카메라 위치 계산, 블럭 Y축 이동 |
| Delivery Man | 중심 유지 | 좌우 기울기 판단, 도착 지점 이동 시 성공  |
| Wire Walking | 줄타기   | 일정 범위 이상 중심 벗어나면 실패 처리    |

* `Game_UIManagaer.cs`에서 각 게임별 타이머, 성공/실패 여부를 판단
* `Game_DataManager.cs`에 클리어 기록 저장 → 퍼즐 해금으로 보상 제공
* `TikTokCtrl.cs`는 TowerMaker에서 블럭 움직임과 카메라 이동을 부드럽게 처리

---

### 📦 자세 3D 뷰어 구현

* 3D 모델링된 자세를 프레임 단위로 나눠 순차 재생하는 시스템 개발 (`MediaPlayer.cs`)
* `ExerciseDataSet.cs` ScriptableObject를 통해 각 자세의 프레임 프리셋 관리
* Pinch/Zoom 제스처로 회전 및 확대 가능 (`ObjectZoomControls.cs`, `PinchDetection.cs`)

```csharp
foreach (var frame in dataSet.objPerFrame)
{
    Instantiate(frame, parent);
}
```

* 사용자는 360도 회전이 가능한 UI에서 정확한 운동 자세를 사전에 확인 가능

---

### 🌍 현지화 및 시스템 연동

* `LocalizeStringEventManager.cs`와 Unity Localization 시스템을 이용해 한/영 전환 가능
* 언어 선택 시 전 UI 텍스트 동기화 및 즉시 적용
* BLE 센서 연동 여부 체크 후 상태에 따라 경고 메시지 및 버튼 비활성화 처리 구현

---

## 🔁 전체 흐름도

```
▶ 로그인 → 모드 선택
   ├ 트레이닝 모드
   │   └ 설명 → 튜토리얼 → 본 운동 → Clear 팝업 → 데이터 저장
   ├ 게임 모드
   │   └ 게임 선택 → 진행 → 성공/실패 → 보상 퍼즐 조각 획득
   └ 자세 뷰어 모드
       └ 프레임 기반 자세 재생, 확대/회전 가능
▶ 프로필 수정 / 기록 확인 / 언어 전환
```

---

## ✅ 요약 및 핵심 성과

| 항목       | 내용                                            |
| -------- | --------------------------------------------- |
| 총 개발 기간  | 6개월                                           |
| 기여도      | 전체 기능 개발 100%                                 |
| 트레이닝 콘텐츠 | 7종 x 3단계 루틴                                   |
| 게임 콘텐츠   | 3종 (Tower Maker / Delivery Man / Wire Walk)   |
| BLE 연동   | 실시간 중심/자세 인식 처리                               |
| 언어 지원    | 한국어, 영어                                       |
| UI 구현 수  | 약 30개 화면 / 팝업 / 슬라이더 UI 구현                    |
| 주요 특징    | 실시간 운동 피드백, 영상 기반 루틴, 반복 학습, 다국어 지원, 3D 모델 뷰어 |

---

## 🎬 영상 및 스토어 링크

* [📺 유튜브 시연 영상](https://www.youtube.com/watch?v=Un5JtJjEnXU)
* [📲 Google Play 다운로드](https://play.google.com/store/apps/details?id=com.gateways.amazingcore&hl=ko-KR)
* [🌐 공식 웹사이트](https://bojamajafitness.com/article/%EB%B3%B4%EC%9E%90%EB%A7%88%EC%9E%90-%ED%94%BC%ED%8A%B8%EB%8B%88%EC%8A%A4-%EC%BD%98%ED%85%90%EC%B8%A0/8/9/)

---

**“어메이징 코어”는 게임처럼 재미있게, 콘텐츠처럼 꾸준히 운동하게 만드는 홈 피트니스 솔루션입니다.
기획, 설계, 개발 전반을 스스로 담당하며 실전 프로젝트 수준의 시스템 구축을 경험했습니다.**


