# Matchday

가상의 풋볼 리그 경기를 실시간으로 관전하는 브라우저 기반 풋볼 관전 게임.
팀명·선수명은 모두 가상. 순수 프론트엔드 (HTML5 Canvas + Vanilla JS), 백엔드 없음.

## 파일 구조

```
frontend/index.html   ← 유일한 소스 파일 (전체 게임)
```

## 배포

Vercel 정적 호스팅. `frontend/` 폴더가 루트.

```bash
cd frontend && npx vercel --prod
```

## 핵심 구조

### 팀 / 선수
- 8개 팀 (MC, ARS, LIV, CHE, TOT, MUN, NEW, AVL)
- 팀당 11명, 능력치: pa(pace) sh(shooting) ps(passing) dr(dribbling) de(defending)
- `TEAMS` 객체에 모든 선수 데이터 정의

### 전술
- `TACTICS`: 팀별 포메이션 + 스타일 정의
- 포메이션: 4-3-3 / 4-4-2 / 4-2-3-1 / 3-5-2
- 스타일: possession / highpress / counter
- 고압박 팀은 수비 시 presser 2명, 나머지는 1명

### 시뮬레이션
- `FPM=80`: 1배속 기준 프레임/분 (90분 = 약 2분 실시간)
- `frm += speed`: 게임 클럭만 배속, 선수 물리는 항상 1x
- 이벤트(패스/슛/턴오버)는 `eventTimer -= speed`로 게임 시간 기준 발동
- 자동 시뮬: `simResult()` Poisson 분포로 득점 계산

### 공 / 선수 물리
- `Ball`: flying 상태(물리) vs attached 상태(carrier에 lerp 추종)
- `pendingCarIdx`: 공이 날아가는 중 다음 수신자 예약, 도착 시 carIdx 전환
- Player role: `carrier` / `support` / `presser` / `shape-att` / `shape`
- 22명 상호 separation force로 뭉침 방지

### 시즌
- 8팀 라운드로빈 14라운드 (홈/원정 각 1회)
- `ST` (localStorage 'matchday1'): `{tbl, res, wr, wm, live}`
- 경기 종료 → 같은 라운드 나머지 경기 자동 시뮬 → 다음 경기 자동 시작
- `live` 필드로 진행 중 경기 상태 저장 (다른 경기 보다 돌아와도 복원)

### 날씨
- 라운드별 월 매핑 (8월~4월), 확률 테이블로 맑음/비/눈 결정
- 비: 패스 정확도 -12%, 선수 속도 -10%, 빗방울 파티클
- 눈: 패스 정확도 -22%, 선수 속도 -22%, 눈송이 파티클 + 피치 하얀 틴트

### 진영 교체
- 하프타임 후 `[...HP,...AP].forEach(p => { p.isH=!p.isH; p.hrx=1-p.hrx; ... })`
- `updateScorebar()`: 진영에 따라 스코어바 좌우 팀 위치도 스왑
- `homeRight = HP[0].isH` 기준으로 슛 방향, 골 판정, 패스 방향 모두 결정

### 렌더
- `requestAnimationFrame` 루프, 매 프레임 1회 tick()
- Canvas 860×500, 상대좌표(0~1)를 `rp(rx,ry)`로 픽셀 변환
- GK: 사각형 + 연두색, 필드 플레이어: 원형 + 팀 색상
- 골 플래시(금색), 진영 교체 플래시(파란색), 날씨 파티클 레이어 순서로 렌더

## 주요 상수

| 상수 | 값 | 설명 |
|---|---|---|
| FPM | 80 | 프레임/분 (1배속) |
| SEP_RADIUS | 0.10 | 선수 간 분리 반경 |
| SEP_FORCE | 0.018 | 분리 힘 |
| maxSpd | 0.0012 + pace*0.0008 | 선수 최대 속도/프레임 |
