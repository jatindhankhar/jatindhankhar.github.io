---
title: 'Using Google App Script to Automate Treat Counter '
categories: programming
excerpt: How I used google sheets with google app script to make a simple web app
  to automate treat counter
tags:
- programming
- js
comments: true
description: How I used google sheets with google app script to make a simple web
  app to automate treat counter

---
```js
function postTreatData() {
    var sheet = SpreadsheetApp.openById("SHEET_ID");

    var data = sheet.getDataRange().getValues();
    // Start from second entry, since first entry is the columns itself
    var memberStats = []
    for (var i = 1; i < data.length; i++) {
        memberStats.push(getMemberStats(data[i]));
    }

    const pending = memberStats.filter(function(stats) {
        return stats['missed_count'] >= 4;
    });
    pending.sort(function(a, b) {
        return a['treats_pending'] > b['treats_pending'];
    });
    pending_messages = pending.map(messageToMarkDownEntry);
    postOnGroup(pending_messages.join("\n"));
}

function getMemberStats(rowData) {
    return { name: rowData[0], missed_count: parseInt(rowData[1], 10), given_count: parseInt(rowData[2], 10), treats_pending: parseInt(rowData[3], 10) };
}

function messageToMarkDownEntry(data) {
    return '#### ' + data['name'] + " -- Treats Pending  " + data['treats_pending'];
}

function postOnGroup(message) {

    var heading = '# 🍕🍔🍟  Treat LeaderBoard  🍧🍨🍰 \n # 🥄🍴 Be Ready ️🥣🥡🥢\n\n\n\n';
    var footer = '> ## [Treat Counter Sheet](https://docs.google.com/spreadsheets/d/SHEET_ID/) \n ';



    var formData = {
        "msgtype": "markdown",
        "markdown": {
            "title": "Ice Cream LeaderBoard",
            "text": [heading, message, footer].join("\n"),

        }
    }

    var headers = {
        "Accept": "application/json",
        "Content-Type": "application/json",
    };

    var options = {
        "method": "POST",
        "headers": headers,
        "payload": JSON.stringify(formData)
    };
    url = 'https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN';
    UrlFetchApp.fetch(url, options);

}
```