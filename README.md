# Monty Run

MONSTER(AI Travel Partner) 마스코트 **Monty**의 도시 러너 미니게임. 서버·빌드 없이 `index.html` 한 파일로 동작합니다.

## 배포

| 서비스 | 방법 |
|---|---|
| GitHub Pages | 이 폴더를 저장소 루트로 푸시 → Settings → Pages → Branch: `main` / `(root)` |
| Cloudflare Pages | 대시보드 → Create → Upload assets → 이 폴더 업로드 |
| Netlify Drop | https://app.netlify.com/drop 에 이 폴더 드래그 |
| Vercel | `vercel .` 또는 대시보드에서 폴더 업로드 |

## 조작

- 탭 / 스페이스 / ↑ : 점프 (공중에서 한 번 더 가능)
- 28초마다 TOKYO → BANGKOK → PARIS → NEW YORK → SEOUL 순으로 도시가 바뀝니다
- 스마트폰(eSIM) 아이템: +100 MB 데이터
- 장애물: No Signal 표지판, 콘

## 앱 연동 (WebView 브릿지)

게임은 아래 이벤트를 호스트 앱으로 보냅니다.

```json
{ "source": "monty-run", "type": "gameover", "score": 1240, "best": 1500, "cities": 3, "dataMB": 300, "haul": { "coffee": 4, "phone": 3 } }
{ "source": "monty-run", "type": "reward",   "dataMB": 300 }
```

### iOS (WKWebView)

```swift
let config = WKWebViewConfiguration()
config.userContentController.add(self, name: "montyRun")
let webView = WKWebView(frame: .zero, configuration: config)

// WKScriptMessageHandler
func userContentController(_ c: WKUserContentController, didReceive message: WKScriptMessage) {
    guard message.name == "montyRun",
          let body = message.body as? [String: Any],
          let type = body["type"] as? String else { return }
    if type == "reward", let mb = body["dataMB"] as? Int {
        // 쿠폰 지급 API 호출
    }
}
```

### Android (WebView)

```kotlin
class MontyRunBridge { @JavascriptInterface fun postMessage(json: String) { /* parse & grant */ } }
webView.settings.javaScriptEnabled = true
webView.addJavascriptInterface(MontyRunBridge(), "MontyRunAndroid")
```

### 웹 (iframe)

```js
window.addEventListener('message', e => { if (e.data?.source === 'monty-run') console.log(e.data); });
```

## 커스터마이즈

- `ITEMS` 배열: 아이템별 점수·표정·등장 확률·데이터 보너스
- `CITIES` 배열: 도시 순서, 하늘/스카이라인 색, 게임오버 인용구
- `S.cityT > 28`: 도시 전환 주기(초)
- 스프라이트는 캐릭터 시트에서 추출한 PNG를 data URI로 내장
