---
layout: post
title: Running Tests with AngularJS, Karma and Jasmine
description: Example on how you can run specific Jasmine tests and how you can debug them in Chrome Dev tools.
categories: angularjs angualar karma jasmine debugging
published: True
---

When running the JavaScript tests on our current codebase, it takes too long to initialize and run tests.
This is big slowdown, especially for someone who doesn't have much experience with Jasmine or Angular.

Ideally for me would be if I could:

- run specific tests
- being able to breakpoint and debug test in Chrome Dev tools
- auto update, so I could just add line of code to test and refresh the page


_The following changes might be specicific for our codebase setup._

# How To Do It

1. When running tests, use `fdiscribe` and `fit` which will focus on specific spec or sepcific test. ([Jasmin focused_spec.js][focused-spec])

2. Install `karma-cli` ([StackOverflow #1][sr1])

   `npm install karma-cli -g`

3. In static/karma.conf.js set `autoWatch` to be True, so the new changes in files will be automatically picked up. ([Newtriks LTD blog][autowatch])

   `autoWatch: true`

4. Run tests using `karma-cli` ([StackOverflow #2][sr2])

    `$> cd project_folder`

    `$> cd static`

    `$> karma start karma.conf.js --browser=Chrome --single-run=false --reports=html`

5. After the `karma` test starts, the new Chrome window will open up. Click on Debug in new window.

    ![Chrome window when starting test](/assets/js-test/chrome.png)

6. In new tab, open developers tools and you have access to all running tests.

    ![Chrome debugging using Dev Tools](/assets/js-test/chrome-debug.png)

When you edit your tests or specs, just refresh the tab with developer console open and the changes should be immediately visible.

[focused-spec]: http://jasmine.github.io/2.1/focused_specs.html
[sr1]: http://stackoverflow.com/questions/17704106/karma-command-not-found-when-karma-already-installed/23767936#23767936
[autowatch]: http://newtriks.com/2013/04/26/how-to-test-an-angularjs-directive/
[sr2]: http://stackoverflow.com/questions/14412437/how-do-i-debug-a-jasmine-spec-run-with-testacular-karma/33302720#33302720
