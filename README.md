# API Automation Tests Guide

[Here is Traditional Chinese version 繁體中文版本](https://github.com/hayasilin/api-automation-tests-guide/blob/master/README_zh-hant.md).

This is a development guide of how to develop reliable and stable API automation tests that I conclude from my experience. I welcome your feedback in [issues](https://github.com/hayasilin/api-automation-tests-guide/issues) and [pull requests](https://github.com/hayasilin/api-automation-tests-guide/pulls).

If you are a mobile app developer and want to know about mobile app UI automation tests, you can see my [iOS and Android UI Automation Tests Guide](https://github.com/hayasilin/ios-android-ui-automation-tests-guide)

If you are a web developer and want to know about web UI automation tests, you can see my [Web UI Automation Tests Guide](https://github.com/hayasilin/web-ui-automation-tests-guide/blob/master/README_zh-hant.md)

## Introduction

Before you start reading this guide. I assume you have basic knowledge of automation testing like API automation tests. In addition, you have experience in implementing automation tests into your CI/CD automation pipeline.

Here are some of the documents of the test frameworks. If something isn't mentioned here, it's probably covered in one of these:
- Postman
  - [Postman](https://www.postman.com/)
  - [Newman](https://support.getpostman.com/hc/en-us/articles/115003710329-What-is-Newman-)

## Table of Contents
- [Before developing API automation tests](#before-developing-api-automation-tests)
  - [API automation tests is indispensible](#api-automation-tests-is-easy-to-develop-but-expensive-to-maintain)
  - [What are flaky tests](#what-are-flaky-tests)
  - [Why API automation tests make flaky tests](#why-api-automation-tests-make-flaky-tests)
- [Best practice to make API automation tests reliable and stable](#best-practice-to-make-api-automation-tests-reliable-and-stable)
  - [Follow testing pyramid](#follow-testing-pyramid)
  - [The mind set of developing API automation tests](#the-mind-set-of-developing-api-automation-tests)
  - [Choose tests cases for API automation tests](#choose-tests-cases-for-api-automation-tests)
  - [Top 5 practical ways to avoid flaky tests](#top-5-practical-ways-to-avoid-flaky-tests)
- [API tests code convention](#api-tests-code-convention)
- [Common questions](#common-questions)

## Before developing API automation tests

### API automation tests is indispensible
- In app and web development, getting data from server side through API is a common practice nowadays. Restful API and GraphQL are common API tools.
- If app or web have issues, apart from client side issues, another major root cause if because API has issues.
- Hence, by having a good API automation tests, not only we can find issues early, we can avoid our app or web have major issues.
- API may need to be changed frequently due to business plan, so test code need to change as well, don't forget the maintenance cost.
- For QA team, if they have API automation tests, they can check API first to check whether it's a server side issue. If API is working properly, then they can narrow down the question to client side. Hence for development and testing, API automaiton test is a indispensible tool.
- Like UI automation tests, if you have poorly written API test code, the first and foremost thing you are going to deal with is the notorious **flaky tests**.
- Flaky tests will make unreliable and unstable API automation tests. However, API tests is more simple compared to UI tests, so it has lower chance to have flaky tests.

### What are flaky tests
- Flaky tests are when you run your automation tests, some tests passes this time but fail next time even you didn't change any code. What's worse when you check the web application by yourself, it works fine. It's unpredictable to know if the tests will pass or not.
- Flaky tests example is below, the Test A and Test C are flaky tests because pass and fail results take turns to show. On the contrary, Test B is not a flaky test because it keep passing. 
- In general, we will put UI automation tests into our CI/CD pipeline and trigger it automatically according to our setting. One of the most common tool we use is Jenkins. If your team have flaky tests, you can see the example below that your Jenkins result would be highly likely failed because Jenkins will mark the job failed even only 1 test case failed. Then, your team need to send one of the team member to check why it failed.
- Below example only has 3 tests cases. If your team has 100 more test cases and there are flaky tests. Imagine how many times your team need to check if it's really API issue or flaky tests.

| Test Runs   | 1st     | 2nd      | 3rd      | 4th      | 5th     | 6th      | 7th      | 8th      | 9th      | 10th     | Next run?              |
| ----------- | ------- |--------- | ---------| -------- | ------- | -------- | -------- | -------- | -------- | -------- | ---------------------- |
| Test Case A | Success | **Fail** | Success  | Success  | Success | **Fail** | Success  | Success  | **Fail** | **Fail** | **?**                  | 
| Test Case B | Success | Success  | Success  | Success  | Success | Success  | Success  | Success  | Success  | Success  | **?**                  | 
| Test Case C | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | Success  | **Fail** | **?**                  | 
| ...                                                                                                                                              |
| Jenkins     | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | **Fail** | **Fail** | **Highly likely Fail** |


- Why flaky tests are bad:
  - Make the team has no confidence on API automation tests.
  - It's a blocker for making CI/CD pipeline if you design a series of actions depend on the result of API auotmation tests.
  - Can't achieve the goal of fast product delivery, let alone maintain or improve web application's quality by providing quick feedback during development

- Google's experience on handling flaky tests
  - 2015 [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
  - 2016 [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
  - 2017 [Where do our flaky tests come from](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
  
### Why API automation tests make flaky tests

**Internal flaky factors:**
- API may change in every release.
- Team don't have a stable test environment or server.
- No API document to follow for developing test code.
- Poorly written test code.

## Best practice to make API automation tests reliable and stable

### Follow testing pyramid

According to Google's [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html), testing pyramid is try to imagine a pyramid, the bulk of your tests are unit tests at the bottom of the pyramid. As you move up the pyramid, your tests gets larger, but at the same time the number of tests (the width of your pyramid) gets smaller.

Google often suggests a 70/20/10 split: 70% unit tests, 20% integration tests, and 10% end-to-end tests. The exact mix will be different for each team, but in general, it should retain that pyramid shape. Try to avoid anti-pattern like the team relies primarily on end-to-end tests, using few integration tests and even fewer unit tests. 

Just like a regular pyramid tends to be the most stable structure in real life, the testing pyramid also tends to be the most stable testing strategy.

API testing is in the category of integration tests, the proportion should in between unit testing and UI testing.

### The mind set of developing API automation tests
- **Important!** Server side engineer who develops API should provide API document, in which should includes API spec, endpoints, payload, return value can be null or not, return value will be empty or not. By having these information in API document, we can follow the document to develop API automation tests.

## Choose tests cases for API automation tests

**Pros**
- Verify API response status code.
- Verify a return value that will not be null according to API document.
- Verify a return value that will not be empty according to API document.
- Verfiy a return collection value has certain items if it's specified in API document.

**Cons**
- Structure changing, such as check whether array change to object.
- Type changing, such as check whether string change to number.

- API response may be huge, if we try to verify all the value it would make a higher maintenance cost. So only choose the most important values to verify.
  
### Top 5 practical ways to avoid flaky tests
1. Server side engineer who develop API must provide API document and should include necessary specs.
2. The API response you see now may not to be the whole cases. API response might be different according to different situation, including error cases. Consider every factor that could cause flaky and make your tests robust to deal with all known API response changes.
3. Need to run all the tests around 10 times after complete new test code to see it won't become flaky tests.
6. Choose the mature API to develop test code, which means the API will not be changed in short term business plan.
7. Don't over design your test code or make it too complex, keep it simple and make it easy to maintain.

## API tests code convention

**Below uses javascript and Postman as example**

### Status code

```javascript
// first check status code
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

### String handling

```javascript
var jsonData = pm.response.json();

// If titleLabel is required in API document, it shouldn't be null.
pm.test("titleLabel exists", function () {
    pm.expect(jsonData.titleLabel.text).not.to.eql(null); 
});

// If titleLabel is required and will not be empty value according to API document, it shouldn't be empty.
pm.test("titleLabel exists", function () {
    pm.expect(jsonData.titleLabel.text).is.not.empty;
});

// If titleLabel is required in API document, and result is fixed, check it.
pm.test("titleLabel exists", function () {
    pm.expect(jsonData.titleLabel.text).eql("your title"); 
});
```

### Array handling

```javascript
var jsonData = pm.response.json();

// If array is requried and has certain fix length according to API document, check it.
pm.test("array data has correct length", function () {
    pm.expect(jsonData.data.results.length).eql(2);
});
```

### Random handling
```javascript
var jsonData = pm.response.json();

// Sometimes the API response data is too large, so need to use some random way to test API for higher confidence, but use it carefully otherwise it would cause flaky tests.
pm.test("non-nullable data is not null", function () {
    var randomNumber = Math.floor(Math.random() * 1);
    pm.expect(jsonData.titleLabel).not.to.eql(null);
    pm.expect(jsonData.sections[randomNumber].items[randomNumber].url).not.to.eql(null);
    pm.expect(jsonData.sections[randomNumber].items[randomNumber].imageUrl).not.to.eql(null);
    pm.expect(jsonData.sections[randomNumber].items[randomNumber].titleLabbel).not.to.eql(null);
});
```

## Common questions
1. What frequency should we trigger our API automation tests in CI/CD pipeline to test our API is functioning properly?
  - My recommendation is **thrice a day**.
    - Because usually your team, including client side developers or server side developers, are modifying code or environemnt's config for new features during working hours. API tests would be running in different environments (Beta / Release). So it's better to avoid running API tests to frequency to avoid flaky factors. Run API tests thrice a day can provide us enough confidence to know our API is working properly.
    - Because API tests is more simple compared to UI tests and it runs fast. If server side engineerupdate API code, not only running server side's own unit tests, also recommend to run API automation tests as well, it could help to find issues in early stage.

2. What test frameworks or tools should we use?
  - API tests is simple comapred to UI tests, all kinds of tool should be fine to use. I use Postman, which is a great API tool and it provides automation testing tool called Newman.