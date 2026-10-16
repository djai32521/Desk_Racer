# [Google AI Studio + Three.js] 내 방 책상 위를 질주하는 3D 미니어처 레이싱 게임 '데스크 레이서(Desk Racer)' 개발기 및 GitHub Pages 배포 가이드

안녕하세요! 이번 포스팅에서는 **Google AI Studio**를 적극 활용하여 웹 브라우저에서 60FPS로 매끄럽게 구동되는 3D 미니어처 레이싱 게임 **「데스크 레이서(Desk Racer)」**를 개발한 전 과정과 핵심 코드 분석, 그리고 **GitHub Pages**로 원클릭 배포하는 방법까지 상세하게 공유해보려 합니다.

평소 3D 그래픽스나 물리 시뮬레이션 게임 개발에 관심이 많으셨던 분들, 특히 AI 코딩 도구를 활용해 복잡한 Three.js 프로젝트를 현실로 완성해보고 싶으신 분들께 큰 도움이 되기를 바랍니다!

---

## 📌 목차
1. [프로젝트 소개: '데스크 레이서(Desk Racer)'란?](#1-프로젝트-소개-데스크-레이서desk-racer란)
2. [Google AI Studio와 함께한 개발 여정](#2-google-ai-studio와-함께한-개발-여정)
3. [시스템 아키텍처 및 주요 기술 스택](#3-시스템-아키텍처-및-주요-기술-스택)
4. [핵심 기능 및 상세 코드 분석](#4-핵심-기능-및-상세-코드-분석)
   - [4-1. 롤러코스터 트랙 Catmull-Rom 스플라인 & 3D 레이더 미니맵](#4-1-롤러코스터-트랙-catmull-rom-스플라인--3d-레이더-미니맵)
   - [4-2. 실시간 카트 물리 엔진: 뱅크각, 다운포스, 드리프트](#4-2-실시간-카트-물리-엔진-뱅크각-다운포스-드리프트)
   - [4-3. Web Audio API 기반의 절차적 F1 엔진 사운드 합성](#4-3-web-audio-api-기반의-절차적-f1-엔진-사운드-합성)
   - [4-4. 시네마틱 비주얼: ACES Filmic, Unreal Bloom & Bokeh DOF](#4-4-시네마틱-비주얼-aces-filmic-unreal-bloom--bokeh-dof)
   - [4-5. 한국어(기본) / 영문 원클릭 다국어(i18n) 시스템](#4-5-한국어기본--영문-원클릭-다국어i18n-시스템)
5. [성능 최적화: 1랩 렉(GPU Spike) 없는 GPU 웜업 기법](#5-성능-최적화-1랩-렉gpu-spike-없는-gpu-웜업-기법)
6. [GitHub 소스 배포 & GitHub Pages 라이브 호스팅 가이드](#6-github-소스-배포--github-pages-라이브-호스팅-가이드)
7. [마무리 및 개발 소감](#7-마무리-및-개발-소감)

---

## 1. 프로젝트 소개: '데스크 레이서(Desk Racer)'란?

**데스크 레이서(Desk Racer)**는 컴퓨터 책상 위 소품들(LG 울트라기어 모니터, 로지텍 기계식 키보드, 게이밍 마우스패드, 방송용 마이크, 고양이 피규어 등) 사이를 롤러코스터처럼 입체적으로 휘감고 돌아가는 트랙 위에서 1:87 HO / 1:150 N 스케일의 마이크로 카트를 운전하는 **3D 타임 트라이얼 레이싱 게임**입니다.

### 🎮 주요 특징
- **현장감 넘치는 4가지 시점**:
  1. 후방 추적 뷰 (Chase Cam)
  2. 1인칭 콕핏 뷰 (Cockpit POV - 시선 정렬 및 헬멧 바이저 지원)
  3. 방 전체 디오라마 뷰 (Full Desk Orbit)
  4. 상공에서 자동차를 추적하는 드론 뷰 (Drone Cam)
- **F1 5-레드 라이트 스타트 시퀀스**:
  - 실제 F1 그랑프리처럼 5개의 레드 램프가 순차적으로 점등된 후 일제히 소등(Lights Out)되며 레이스가 시작됩니다.
- **물리 기반 주행감**:
  - 뱅크각(커브에서 기울어짐), 중력 다운포스 G-Force, 드리프트 시 방출되는 스파크 파티클, 가드레일 충돌 반발 및 카메라 셰이크 구현.
- **풍부한 오디오**:
  - 가속, 감속, 최고속도 부스트, 배기음 팝콘(오버런 백파이어) 사운드를 Web Audio API로 실시간 동적 합성.
- **다국어 지원**:
  - 기본 한국어 제공 및 영문 원클릭 토글(KO/ENG) 지원.

---

## 2. Google AI Studio와 함께한 개발 여정

이번 프로젝트는 **Google AI Studio**를 적극적으로 활용하여 초기 기획부터 Three.js 씬 구축, 물리 파라미터 튜닝, 오디오 합성 알고리즘, 빌드 및 배포 파이프라인 구성까지 단계적으로 완성했습니다.

### 💡 AI Studio를 활용하며 느낀 강점
1. **복잡한 3D 수학 연산의 신속한 구현**:
   - 트랙의 3차원 스플라인 곡선에서 차량의 진행 방향(Tangent), 법선(Normal), 종법선(Binormal) 벡터를 계산하여 차량이 트랙 표면에 딱 달라붙은 채 부드럽게 뱅크각을 형성하도록 하는 프레네-세레(Frenet-Serret) 프레임 수학 공식을 AI Studio를 통해 빠르게 도출하고 버그를 잡을 수 있었습니다.
2. **반복적인 파라미터 튜닝의 자동화**:
   - 카트의 최고 속도, 가속도, 제동력, 공기 저항 계수, 카메라 보간 속도(LERP) 등 수십 가지 물리 상수를 상황에 맞춰 질의하고 최적의 밸런스를 즉각 도출할 수 있었습니다.
3. **완전한 엔드투엔드(End-to-End) 솔루션**:
   - 단순 코드 텍스트 생성을 넘어, Vite 빌드 설정(`vite.config.ts`), GitHub Actions 워크플로, GitHub Pages 서브패스 호환성까지 포괄적으로 고려된 프로덕션 코드를 완성할 수 있었습니다.

---

## 3. 시스템 아키텍처 및 주요 기술 스택

```
┌─────────────────────────────────────────────────────────────┐
│                      HTML5 Canvas / HUD                     │
│  (Orbitron / Noto Sans KR, SVG Speedometer, Minimap Radar)   │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                    Three.js R183+ Scene                     │
│  - EffectComposer: ACESFilmic, Vogel Spiral Bokeh, Bloom    │
│  - DeskDioramaBuilder: 모니터, 키보드, 마우스, 책상 소품    │
│  - CoasterTrackBuilder: 3D 롤러코스터 레일, LED 언더글로우 │
│  - MicroKartBuilder: 1:87 HO 스케일 카트 바디, 휠, 스파크  │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                  Core Engine Subsystems                     │
│  ├─ KartPhysicsController: Rapier3D / Spline Curve LERP     │
│  ├─ WebAudioSynth: 절차적 F1 엔진음 & 오버런 백파이어 합성  │
│  └─ i18n Engine: 한국어(기본) / 영어 실시간 UI 번역         │
└─────────────────────────────────────────────────────────────┘
```

- **Runtime & Bundler**: Vite 6, TypeScript
- **3D Graphics Engine**: Three.js (r183.2)
- **Physics**: @dimforge/rapier3d-compat + 커스텀 스플라인 키네마틱스
- **Audio Engine**: Web Audio API (LFE Sub-bass + Piezo Filter + Transient Slices)
- **Deployment**: GitHub Pages, GitHub Actions CI/CD

---

## 4. 핵심 기능 및 상세 코드 분석

### 4-1. 롤러코스터 트랙 Catmull-Rom 스플라인 & 3D 레이더 미니맵

트랙은 720개의 정밀 3D 샘플 포인트(`track_spline.json`)로 구성된 닫힌 C2 연속 스플라인 곡선입니다. HUD 좌측 하단에 표시되는 3D 틸트 미니맵은 고도차(Elevation)를 반영하여 입체적으로 렌더링됩니다.

```javascript
// 미니맵 2D 투영 함수: Y축 고도를 1.48배 과장하여 3D 롤러코스터 입체감을 극대화
const projMinimapRawX = (x, y, z) => x - y * 0.36 + z * 0.08;
const projMinimapRawY = (_x, y, z) => z * 1.08 - y * 1.48;

// 미니맵 상에 실시간 차량 위치 및 네온 시안 발광 코어 렌더링
function drawMinimapRadar(st) {
  if (!radarCtx || !radarCanvas) return;
  radarCtx.clearRect(0, 0, radarCanvas.width, radarCanvas.height);
  if (radarBgCanvas) radarCtx.drawImage(radarBgCanvas, 0, 0);

  const kx = mapMinimapX(st.position.x, st.position.y, st.position.z);
  const ky = mapMinimapY(st.position.x, st.position.y, st.position.z);

  radarCtx.save();
  // 1. 네온 시안 원형 방사형 글레이즈
  const haloGrad = radarCtx.createRadialGradient(kx, ky, 3.0, kx, ky, 26.0);
  haloGrad.addColorStop(0.0, 'rgba(0, 240, 255, 0.90)');
  haloGrad.addColorStop(1.0, 'rgba(0, 210, 255, 0.0)');
  radarCtx.fillStyle = haloGrad;
  radarCtx.beginPath();
  radarCtx.arc(kx, ky, 26.0, 0, Math.PI * 2);
  radarCtx.fill();

  // 2. 중심부 고휘도 코어 렌더링
  radarCtx.fillStyle = '#00e8ff';
  radarCtx.beginPath();
  radarCtx.arc(kx, ky, 10.2, 0, Math.PI * 2);
  radarCtx.fill();
  radarCtx.restore();
}
```

---

### 4-2. 실시간 카트 물리 엔진: 뱅크각, 다운포스, 드리프트

카트는 트랙의 곡률(Curvature)과 진행 속도에 따라 자연스럽게 차체가 기울어지는 뱅크각(Bank Angle)과 노면 밀착력을 높여주는 다운포스(Downforce)를 계산합니다.

```javascript
// 뱅크각 및 원심력에 기반한 다운포스 계산 로직 스니펫
const curvature = sampleTrackCurvature(s);
const centripetalAcc = (speed * speed) * curvature;
const bankAngleRad = Math.atan2(centripetalAcc, 9.81) * 0.72; // 적정 뱅크 계수

// 커브 주행 시 드리프트 임계치 초과 판정
const lateralG = Math.abs(centripetalAcc / 9.81);
const isDrifting = Math.abs(steerInput) > 0.4 && lateralG > 1.25 && speed > 0.15;

if (isDrifting) {
  // 드리프트 마찰 스파크 파티클 방출 및 타이어 스크럽 사운드 트리거
  triggerDriftSparks(st.position, st.tangent);
}
```

---

### 4-3. Web Audio API 기반의 절차적 F1 엔진 사운드 합성

단순한 오디오 루프 재생이 아니라, 차량의 실시간 속도와 RPM에 따라 엔진 기저 주파수(Fundamental Frequency: 75Hz ~ 268Hz)를 변조하고, 가속 페달에서 발을 뗄 때 발생하는 '팝콘 사운드'(배기 오버런 백파이어)를 절차적으로 합성합니다.

```javascript
// 속도에 따른 RPM 및 피치 동적 계산
function updateEnginePitch(speedRatio) {
  // Idle(공회전 75Hz) -> Flatout(고속 204Hz) -> Boost(부스트 268Hz)
  const baseFreq = THREE.MathUtils.lerp(75.0, 240.0, Math.pow(speedRatio, 1.35));
  
  if (engineOscillator) {
    engineOscillator.frequency.setTargetAtTime(baseFreq, audioCtx.currentTime, 0.04);
  }
  
  // 감속(급격한 스로틀 오프) 발생 시 팝콘 백파이어 트리거
  if (prevSpeed - speedRatio > 0.18 && audioCtx.currentTime > nextSputterTime) {
    triggerExhaustPop();
    nextSputterTime = audioCtx.currentTime + 0.35;
  }
}
```

---

### 4-4. 시네마틱 비주얼: ACES Filmic, Unreal Bloom & Bokeh DOF

1:87 스케일 미니어처의 아기자기한 매력을 극대화하기 위해 초점면 앞뒤를 부드럽게 흐리는 **25-tap Vogel Spiral Bokeh 피사계 심도(Depth of Field)** 셰이더와 야간 네온 조명을 위한 **UnrealBloomPass**를 적용했습니다.

```javascript
// 하드웨어 깊이 버퍼(DepthTexture)를 직접 읽어 지오메트리 추가 렌더링 비용이 없는 BokehPass
const bokehPass = new BokehPass(scene, camera, {
  focus: 0.12,         // 카메라 앞 12cm 초점
  aperture: 0.0023,    // 조리개 크기
  maxblur: 0.0016      // 최대 블러 직경
});

// 톤매핑 설정: 사실적인 명암비와 하이라이트 롤오프를 위한 ACESFilmic 적용
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 0.77;
```

---

### 4-5. 한국어(기본) / 영문 원클릭 다국어(i18n) 시스템

사용자 편의성을 위해 기본 언어는 친숙한 **한국어(한글)**로 설정되어 있으며, 언제든지 우측 상단의 `한글 / ENG` 버튼을 클릭하여 영문으로 실시간 전환할 수 있습니다. 설정된 언어는 `localStorage`에 자동 보관됩니다.

```javascript
// 다국어 딕셔너리 예시
const I18N_STRINGS = {
  ko: {
    langBtn: '한글',
    brandTitle: '데스크 레이서',
    welcomeEyebrow: '데스크톱 서킷 · 3랩 타임 트라이얼',
    startBtn: '타임 트라이얼 시작',
    finishTitle: '완주!',
    bestLap: '최고 랩'
  },
  en: {
    langBtn: 'ENG',
    brandTitle: 'Desk Racer',
    welcomeEyebrow: 'DESKTOP CIRCUIT · 3 LAPS',
    startBtn: 'START TIME TRIAL',
    finishTitle: 'FINISH',
    bestLap: 'BEST LAP'
  }
};

function applyLanguage(lang) {
  currentLang = lang === 'en' ? 'en' : 'ko';
  localStorage.setItem('desk_racer_lang', currentLang);
  const t = I18N_STRINGS[currentLang];
  
  document.getElementById('hud-lang-label').textContent = t.langBtn;
  document.getElementById('hud-brand-logo').textContent = t.brandTitle;
  document.getElementById('welcome-start-text').textContent = t.startBtn;
  // ... 모든 UI 텍스트 실시간 치환
}
```

---

## 5. 성능 최적화: 1랩 렉(GPU Spike) 없는 GPU 웜업 기법

3D 웹 게임에서 흔히 겪는 고질적인 문제 중 하나는 **"1랩 첫 코너를 돌 때 화면이 뚝뚝 끊기는 현상"**입니다. 이는 처음 시야에 들어오는 3D 오브젝트의 셰이더 컴파일과 GPU 텍스처 업로드가 런타임에 발생하기 때문입니다.

이를 해결하기 위해 데스크 레이서는 시작 버튼을 누르기 전, 보이지 않는 **64x64 오프스크린 타깃으로 전체 360도 씬의 셰이더와 VBO를 사전 컴파일하는 `warmUpSceneOnGpu()`** 기법을 적용했습니다. 그 결과 1랩 시작부터 끝까지 60FPS가 완벽하게 유지됩니다!

---

## 6. GitHub 소스 배포 & GitHub Pages 라이브 호스팅 가이드

### 🚀 1. GitHub 원격 저장소 푸시
로컬 저장소의 소스 코드를 본인의 GitHub 저장소로 푸시합니다.

```bash
# 1. git 초기화 및 전체 소스 스테이징
git init
git add .

# 2. 커밋 메시지 작성
git commit -m "feat: Desk Racer v1.0 - Korean/English i18n & GitHub Pages deployment"

# 3. 기본 브랜치를 main으로 설정
git branch -M main

# 4. GitHub 원격 저장소 연결 및 푸시
git push https://<GITHUB_PAT>@github.com/djai32521/Desk_Racer.git main
```

---

### 🌐 2. GitHub Pages 배포 설정
본 프로젝트에는 `.github/workflows/deploy.yml` 파일이 포함되어 있어, `main` 브랜치에 코드를 푸시하기만 하면 **GitHub Actions가 자동으로 빌드하여 GitHub Pages에 즉각 배포**합니다.

1. GitHub 저장소(`https://github.com/djai32521/Desk_Racer`)의 **Settings** 탭으로 이동합니다.
2. 좌측 메뉴에서 **Pages**를 클릭합니다.
3. **Build and deployment > Source** 옵션에서 **GitHub Actions**를 선택합니다.
4. 잠시 후 상단에 생성되는 라이브 URL로 접속하면 즉시 게임을 즐기실 수 있습니다!

> 🔗 **배포 완료 URL**: [https://djai32521.github.io/Desk_Racer/](https://djai32521.github.io/Desk_Racer/)

---

## 7. 마무리 및 개발 소감

Google AI Studio와 함께 작업하면서 가장 놀라웠던 점은 **"개발자의 머릿속에 있는 아이디어를 코드로 실체화하는 속도가 비약적으로 단축된다"**는 점이었습니다. 3D 스플라인 보간, 복잡한 셰이더 파이프라인, Web Audio의 오실레이터 튜닝처럼 손이 많이 가고 수학적 계산이 복잡한 작업들을 AI와 대화하며 실시간으로 검증해나갈 수 있었습니다.

여러분도 나만의 아이디어가 있다면 Google AI Studio를 켜고 지금 바로 첫 3D 웹 프로젝트를 시작해보세요!

궁금한 점이나 추가하고 싶은 기능 아이디어가 있다면 언제든지 댓글로 남겨주세요. 긴 글 읽어주셔서 감사합니다! 😊
