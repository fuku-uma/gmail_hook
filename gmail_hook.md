function hook() {
  // 未読かつ一日以内かつ大学からのメールを検索
  // hogehogeを各大学ドメインに変更
  var query = 'is:unread newer_than:1d from:@hogehoge.ac.jp subject:[CS2'; 
  const threads = GmailApp.search('label:unread');  // 未読のスレッドを取得

  if (threads.length == 0) {
    Logger.log('新規メッセージなし');
    return
  }

  threads.forEach(function (thread) {
    const messages = thread.getMessages();

    const payloads = messages.map(function (message) {
      const from = message.getFrom();
      const subject = message.getSubject();
      const plainBody = message.getPlainBody();

      const webhook = getWebhookUrl();

      Logger.log(subject);
      if (subject.includes('[CS2')){
        message.markRead();  // メールを既読に設定する
        const payload = {
          content: subject,
          embeds: [{
            title: subject,
            author: {
              name: from,
            },
          description: plainBody.substr(0, 2048),
          }],
        }
        return {
          url: webhook,
          contentType: 'application/json',
          payload: JSON.stringify(payload),
        }
      }
      return null;
   })

    Logger.log(payloads);
    const filteredPayloads = payloads.filter(function (payload) {
      return payload !== null; // nullでないものだけフィルタリング
    });

    UrlFetchApp.fetchAll(filteredPayloads);
  })
}

function getWebhookUrl() {
 const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
 const sheet = spreadsheet.getActiveSheet();

 return sheet.getRange(1, 1).getValue();  // セルA1を取得
}
