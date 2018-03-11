---

title: "Parsing PDF files with ruby"
excerpt: "Parsing PDF containing tables using ruby"
tags: [programming,ruby]
description: How to parse PDF containing tables using ruby
---

I was working on the backend for a cool (college) project that would send transactional notifications via SMS and e-mail and decided to incorporate the notification for attendance as well, but there was one problem, I had no proper data for parsing. So I did what every other person would do first hand, ask authorities for data and I was directed to administrative department only to be told that I can have raw data file with no attendance and it was kind of funny and stupid at the same moment as most important thing was the attendance % itself.

# Plan
Since attendance was embedded in a table without any standard format inside PDF file, at first I tried various online, PDF to excel converters only to find all of them failed. Another solution was to manually copy the data,parse it and the process it but one key thing was missing automation and to conquer humanity automation should be priority :stuck_out_tongue: . Then I looked for gems that would help in parsing the data from PDF and I found [pdf-reader](https://github.com/yob/pdf-reader).
