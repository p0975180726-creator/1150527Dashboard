/**
 * 護腎紅綠燈：高血鉀問卷雲端後台接收腳本
 *
 * 功能：接收前端問卷的 POST 請求，解析資料並寫入指定的 Google 試算表（分頁：課後問卷）
 * 注意：本檔案為純 JavaScript，請勿加入任何 HTML 標籤（例如 <!DOCTYPE html> 或 <script> 等）
 */

function doPost(e) {
  // 建立回傳物件
  var res = ContentService.createTextOutput();
  res.setMimeType(ContentService.MimeType.JSON);
  
  try {
    // 自動定位此程式綁定的試算表 (Container-bound)
    var ss;
    try {
      ss = SpreadsheetApp.getActiveSpreadsheet();
    } catch (err) {
      // 備用方案：若為獨立部署，則直接開啟您的試算表 ID
      var sheetId = "1xlaVO1Eny4qUnk2BE0b2XTDKp2MTg0ATJLVcEtUM6lI";
      ss = SpreadsheetApp.openById(sheetId);
    }
    
    var sheetName = "課後問卷";
    var sheet = ss.getSheetByName(sheetName);
    
    // 如果找不到該名稱的分頁，則自動建立一個並寫入表頭
    if (!sheet) {
      sheet = ss.insertSheet(sheetName);
      sheet.appendRow(["時間戳記", "性別", "年紀", "衛教內容清楚明瞭", "本次影片觀看滿意度", "Q1回答", "Q2回答", "Q3回答", "Q4回答", "總分數"]);
    }
    
    // 安全解析 JSON 傳入內容
    var params;
    if (e && e.postData && e.postData.contents) {
      try {
        params = JSON.parse(e.postData.contents);
      } catch (jsonErr) {
        // 如果傳送格式為 URL 編碼，則改由 parameter 讀取
        params = e.parameter;
      }
    } else if (e && e.parameter) {
      params = e.parameter;
    } else {
      throw new Error("發送請求內容為空，請確認傳輸資料格式。");
    }
    
    // 準備要寫入試算表的欄位值 (防空機制)
    var timestamp = new Date(); // 系統當前時間
    var gender = params.gender || "";
    var age = params.age || "";
    var sat1 = params.satisfaction1 || "";
    var sat2 = params.satisfaction2 || "";
    var q1 = params.q1 || "";
    var q2 = params.q2 || "";
    var q3 = params.q3 || "";
    var q4 = params.q4 || "";
    var score = params.score !== undefined ? params.score : 0;
    
    // 將資料新增至試算表最後一列
    sheet.appendRow([
      timestamp, 
      gender, 
      age, 
      sat1, 
      sat2, 
      q1, 
      q2, 
      q3, 
      q4, 
      score
    ]);
    
    // 回傳 JSON 成功狀態給網頁
    res.setContent(JSON.stringify({ "result": "success" }));
    
  } catch (error) {
    // 發生錯誤時回傳錯誤訊息
    res.setContent(JSON.stringify({ "result": "error", "error": error.toString() }));
  }
  
  return res;
}

// 提供測試 API 是否正常的 GET 方法
function doGet(e) {
  return ContentService.createTextOutput("問卷 API 連線正常！請使用 POST 方法進行問卷資料傳送。");
}
