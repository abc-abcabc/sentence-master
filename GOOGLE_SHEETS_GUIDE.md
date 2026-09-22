# 📊 구글 스프레드시트 실시간 연동 완벽 가이드

학생들이 문장 학습을 완료할 때마다 **선생님의 구글 스프레드시트에 학생 이름, 정답률, 틀린 문장 등의 결과가 실시간으로 자동 기록**되도록 설정하는 방법입니다.

---

## ⚡ 3분 만에 끝나는 설정 단계

### 1단계: 구글 스프레드시트 생성 및 스크립트 열기
1. [구글 스프레드시트(Google Sheets)](https://sheets.new)에 접속하여 새 시트를 만듭니다. (시트 이름 예: `영어 문장 마스터 학습 기록`)
2. 상단 메뉴에서 **[확장 프로그램]** ➔ **[Apps Script]**를 클릭합니다.

---

### 2단계: 연동 코드 복사 & 붙여넣기
새로 열린 창의 기존 내용을 모두 지우고, 아래 코드를 그대로 복사해서 붙여넣습니다.

```javascript
/**
 * 📖 문장 마스터 PRO - 구글 스프레드시트 실시간 연동 스크립트
 */
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // 시트가 비어있으면 헤더(제목 행) 자동 생성 및 디자인 적용
    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        "학습 일시",
        "학생 이름",
        "학습 문장 수",
        "정답률",
        "최대 콤보",
        "총 시도 횟수",
        "오답 횟수",
        "틀린 문장 목록",
        "학습 단계",
        "학습 모드"
      ]);
      
      var headerRange = sheet.getRange(1, 1, 1, 10);
      headerRange.setBackground("#4f46e5");
      headerRange.setFontColor("#ffffff");
      headerRange.setFontWeight("bold");
      headerRange.setHorizontalAlignment("center");
      sheet.setFrozenRows(1);
    }
    
    var data = {};
    if (e && e.postData && e.postData.contents) {
      data = JSON.parse(e.postData.contents);
    } else if (e && e.parameter) {
      data = e.parameter;
    }
    
    var timestamp = data.timestamp || Utilities.formatDate(new Date(), "Asia/Seoul", "yyyy-MM-dd HH:mm:ss");
    var studentName = data.studentName || "익명";
    var totalSentences = data.totalSentences || 0;
    var accuracy = data.accuracy || "0%";
    var maxStreak = data.maxStreak || 0;
    var totalAttempts = data.totalAttempts || 0;
    var wrongAttempts = data.wrongAttempts || 0;
    var wrongSentences = data.wrongSentences || "없음 (100% 완벽 통과)";
    var activeSteps = data.activeSteps || "1~5단계 전체";
    var mode = data.mode || "일반 학습";
    
    // 새 행 추가
    sheet.appendRow([
      timestamp,
      studentName,
      totalSentences,
      accuracy,
      maxStreak,
      totalAttempts,
      wrongAttempts,
      wrongSentences,
      activeSteps,
      mode
    ]);
    
    // 행 스타일 및 정렬
    var lastRow = sheet.getLastRow();
    sheet.getRange(lastRow, 1, 1, 10).setVerticalAlignment("middle");
    sheet.getRange(lastRow, 1, 1, 7).setHorizontalAlignment("center");
    sheet.getRange(lastRow, 8).setHorizontalAlignment("left"); // 오답 목록은 좌측 정렬
    sheet.getRange(lastRow, 9, 1, 2).setHorizontalAlignment("center");
    
    return ContentService.createTextOutput(JSON.stringify({ status: "success", message: "기록 완료" }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ status: "error", message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({ status: "ok", message: "문장 마스터 Google Sheets Webhook이 정상 작동 중입니다." }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

붙여넣은 후 상단의 **💾 저장 아이콘(또는 Ctrl+S)**을 누릅니다.

---

### 3단계: 웹 앱으로 배포 (URL 생성)
1. 우측 상단의 **[배포]** ➔ **[새 배포]**를 클릭합니다.
2. 왼쪽 톱니바퀴 ⚙️ 아이콘을 누르고 **[웹 앱]**을 선택합니다.
3. 아래와 같이 설정합니다:
   - **설명**: `문장 마스터 연동`
   - **다음 사용자로 실행**: `나 (내 이메일)`
   - **액세스 권한이 있는 사용자**: **`모든 사용자 (Anyone)`** *(중요! 학생들이 로그인 없이 결과를 보낼 수 있게 해줍니다)*
4. **[배포]** 버튼을 누르고, 처음 1회 나오는 **[액세스 승인]**을 진행합니다.
   *(구글 보안 경고 화면이 나오면 `고급` ➔ `...로 이동(안전하지 않음)` ➔ `허용` 클릭)*
5. 배포 완료 후 나타나는 **`웹 앱 URL` (https://script.google.com/macros/s/.../exec)**을 복사합니다.

---

### 4단계: 문장 마스터 앱에 연동하기
1. 문장 마스터 웹사이트를 열고, 상단 툴바의 **[⚙️ 구글시트 연동]** 버튼을 누릅니다.
2. 복사한 웹 앱 URL을 붙여넣고 **[저장 및 테스트 전송]**을 누릅니다.
3. 구글 스프레드시트에 테스트 행이 추가되면 성공입니다! 🎉

---

## 🎁 학생들에게 배포하는 가장 쉬운 방법 (학생 설정 불필요!)

선생님이 학생들에게 링크를 보낼 때 뒤에 `?sheet=선생님웹앱URL`을 붙여서 보내면, **학생들은 아무런 설정을 하지 않아도 자동으로 선생님의 구글 시트에 기록**됩니다!

> **예시 링크:**  
> `https://your-domain.vercel.app/?sheet=https://script.google.com/macros/s/AKfycbx.../exec`

*앱 내의 [⚙️ 구글시트 연동] 창에서 `학생 공유용 자동 연동 링크 복사` 버튼을 누르면 이 링크가 1초 만에 자동 생성됩니다!*
