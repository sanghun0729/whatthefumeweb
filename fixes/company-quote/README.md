# company.html 큰 따옴표 모양 수정

## 문제
회사소개 페이지(`/shopinfo/company.html`)의 큰 따옴표가 브랜드 그래픽(PNG)의
둥근 헤비 따옴표와 다른 **각진 직선 바** 모양으로 나옴.

## 원인
`.wtf-card .quote` 규칙이 따옴표를 **DIN Condensed**로 렌더링:

```css
.wtf-card .quote { font-family: "DIN Condensed", var(--font-body) !important; ... }
```

- 문자(U+201C `“` / U+201D `”`)는 동일하지만 **적용 폰트가 달라서** 글리프 모양이 다름.
- 검증 결과: DIN Condensed·Pretendard·Noto Sans KR 모두 이 문자를 "직선 바"로 그림.
  브랜드 PNG의 따옴표는 **Arial/Helvetica 계열의 둥근 곡선 따옴표**이며,
  사이트에 로드된 어떤 폰트로도 정확히 재현되지 않음(디자인 툴에서 만든 그래픽).

## 해결 (권장): PNG 모양을 그대로 벡터화한 SVG로 교체
폰트에 의존하지 않으므로 모든 브라우저/기기에서 PNG와 **동일한 모양** 보장.

- `quote-open.svg`  — 여는 따옴표 (viewBox 142×150)
- `quote-close.svg` — 닫는 따옴표 (viewBox 142×150)
- 색은 `fill="currentColor"` → CSS `color` 를 따라감.

### HTML 교체
```html
<!-- 변경 전 -->
<span class="quote quote--open"  aria-hidden="true">“</span>
<span class="quote quote--close" aria-hidden="true">”</span>

<!-- 변경 후: 문자 대신 SVG를 span 안에 넣기 -->
<span class="quote quote--open"  aria-hidden="true"><!-- quote-open.svg 내용 붙여넣기 --></span>
<span class="quote quote--close" aria-hidden="true"><!-- quote-close.svg 내용 붙여넣기 --></span>
```

### CSS 교체 (`.wtf-card .quote`)
```css
.wtf-card .quote{
  position:absolute; z-index:0;
  width:clamp(90px,15vw,190px);   /* 크기: SVG가 이 폭에 맞춰 스케일 */
  color:var(--blush);             /* SVG 색(currentColor) */
  opacity:.22;                    /* 기존 워터마크 농도 유지 */
  pointer-events:none; user-select:none; line-height:0;
  /* font-family / font-size / font-weight 는 더 이상 불필요 → 삭제 */
}
.wtf-card .quote svg{ display:block; width:100%; height:auto; }
```

## 대안 (간단하지만 근사치)
SVG 없이 폰트만 바꾸려면 `.quote` 폰트를 시스템 산세리프로:
```css
font-family: Arial, Helvetica, "Liberation Sans", sans-serif !important;
```
→ 둥근 곡선 따옴표가 나오지만 PNG와 100% 동일하진 않음(더 둥근 66/99 형태).

## 참고
- PNG처럼 **진한 솔리드**로 원하면 `opacity:1` + `color:var(--wine)`(#7c1836) 로.
- 위치가 어긋나면 `.quote--open`(top/left), `.quote--close`(bottom/right) 오프셋만 조정.
