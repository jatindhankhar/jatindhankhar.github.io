---
title: 'Using Google App Script to Automate Treat Counter '
categories: blog
excerpt: How I used google sheets with google app script to make a simple web app
  to automate treat counter
tags:
- programming
- js
comments: true
description: How I used google sheets with google app script to make a simple web
  app to automate treat counter

---
## Problem

At work, my team follows  [daily standup meeting](https://www.scrum-institute.org/Daily_Scrum_Meeting.php "daily standup meetings")  (tries to  :sweat_smile: ). To ensure that people don't miss standup, we maintain a ice cream counter (which is now a generic treat counter)

**_Rule_**

So if someone misses 4 standups, their counter increases by 1 and they need to treat the entire team to decrement the counter.

Treat counter is maintained as a google sheet and contain stats of every team member including their name, counter and other data.

Sheet is updated manually by treat committee (this can be automated as well )

![](/images/Screenshot 2019-07-27 at 9.56.27 PM.png)

**_Issue_**

It's possible that people lose track of counter and someone needs to remind the person with pending treats.

Even though everyone secretly wants the treat, no one wants to be the one to remind the person with pending treats about the same :laughing: .

**_Plan_**

Secretly remind everyone about the treat and maintain anonymity, automate the process.

## Solutions

There are many ways to automate this problem

1. **Daily Cron:** We can use a daily cron to pull the data from sheet using [Google Sheets API](https://developers.google.com/sheets/api/) , parse the data then post it on the group using DingTalk's Robot API.

   Since sheet is updated manually, we don't need to post the result everyday but only when someone's treat crosses a certain threshold.
2. **Reactive Response (kind of):** Since sheet is not updated daily and we need results to be posted only when someone updates the treats pending column, a reactive approach in response to update seems like a step in the right direction

   We could have used Google app scripts update trigger but this was also a lot of work and involved handling sudden bursts and rollback of  accidental counter upgrades.
3. **Using Google Sheet as a web app:**

   I decided to go with this approach and tried to use sheet as a quick web app by using utilizing google sheets custom triggers.

Now there is a big button inside the sheet itself, treat committee can update the counters and click on the button to announce the treat counter leader board.

**_Google App Script_**

[Google app script  is a  super set of javascript](https://developers.google.com/apps-script/), (ES2015 to be precise ?), so I couldn't use ES6 goodies and syntax sugar.

So when green button is clicked. Script does the following

1. Fetches sheet specified by the `SHEET_ID` (usually part of the url itself)
2. Then converts each row and converts it to a object.
3. Filters people having more than 4 missed counts
4. Sorts them based on their treat pending counter
5. Posts the result by making a POST call to DingTalk's robot api.

![](/images/Screenshot 2019-07-27 at 9.50.48 PM.png)

So this is what I came up with, (code is not organized, this is not how I usually write code :see_no_evil:)

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

Since DingTalk supports only a subset of markdown, I combined several string in markdown to form one big markdown document. If google app script supported ES6, I could have used string interpolation to make it more elegant.

On the ending note, after writing this one, I am missing more standups than I was attending early :p
