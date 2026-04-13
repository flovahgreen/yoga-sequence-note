# Yoga Flow PWA 모니터링 리포트

**점검 일시:** 2026-04-10
**대상 URL:** https://flovahgreen.github.io/yoga-sequence-note/

---

## 결과 요약: ⚠️ 이슈 2건 발견

배포 상태는 대체로 양호하나, 서비스 워커 미등록 및 버전 불일치 이슈가 있습니다.

---

## Part 1: 배포 점검

| 항목 | 상태 | 비고 |
|------|------|------|
| 사이트 접근 (HTTP 200) | ✅ 정상 | |
| index.html | ✅ 정상 | |
| manifest.json | ✅ 정상 | name, short_name, start_url, display, icons 모두 포함 |
| sw.js | ✅ 정상 | 캐시 전략 정상 구현 |
| icon-192x192.png | ✅ 정상 | |
| icon-512x512.png | ✅ 정상 | |
| apple-touch-icon.png | ✅ 정상 | |

## Part 2: 코드 품질 점검

### ✅ 정상 항목

**[데이터안전] initSample() 보호 로직**: 기존 데이터 덮어쓰기 방지 가드 정상 작동
```javascript
if(LS.get(KEYS.SEQ)||LS.get(KEYS.CLS))return;
```

**[데이터안전] localStorage 키 일관성**: KEYS 객체(yoga_sequences, yoga_classes, yoga_settings, yoga_api_key)가 일관되게 사용됨. deleteData()는 confirm() 대화상자 후에만 실행됨.

**[코드품질] 이벤트 핸들러**: 인라인 onclick 방식 사용으로 중복 바인딩 위험 없음. 포즈 삭제 버튼에 event.stopPropagation() 적용됨.

**[PWA설정] Apple 메타 태그**: 모두 정상
- `apple-mobile-web-app-capable`: ✅ yes
- `apple-mobile-web-app-status-bar-style`: ✅ black-translucent
- `apple-touch-icon`: ✅ 존재함
- `link rel="manifest"`: ✅ 존재함
- Safe area CSS (standalone 모드): ✅ env(safe-area-inset-top), env(safe-area-inset-bottom) 적용됨

**[코드품질] JS 구문 오류**: 발견되지 않음

---

### ⚠️ 발견된 이슈

#### 이슈 1: [배포] 서비스 워커 미등록 — 심각도: 높음

**문제:** index.html에 `navigator.serviceWorker.register('./sw.js')` 코드가 없습니다. sw.js 파일은 서버에 존재하지만, HTML에서 등록하지 않아 실제로 작동하지 않습니다.

**영향:** PWA 오프라인 기능이 작동하지 않으며, 앱 설치 프롬프트가 정상적으로 표시되지 않을 수 있습니다.

**수정 제안:** index.html의 `</body>` 직전 또는 스크립트 끝에 다음 코드를 추가:
```javascript
if('serviceWorker' in navigator){
  navigator.serviceWorker.register('./sw.js');
}
```

#### 이슈 2: [배포] 버전 불일치 — 심각도: 중간

**문제:** sw.js의 캐시 버전은 `yoga-flow-v1.6.0`이지만, index.html 설정 화면에 표시되는 버전은 `1.5.0`입니다.

**영향:** 사용자에게 표시되는 버전이 실제 배포 버전과 다릅니다. 디버깅 시 혼동을 줄 수 있습니다.

**수정 제안:** index.html의 버전 표시를 `1.6.0`으로 업데이트하거나, APP_VER 상수를 도입하여 한 곳에서 관리하세요.

---

## 다음 점검 시 확인할 사항
- 서비스 워커 등록 코드 추가 여부
- 버전 번호 동기화 여부
