# API自動化測試規範

[Here is English version](https://github.com/hayasilin/api-automation-tests-guide)

這是總結個人在開發可靠及穩定的API自動化測試所得之規範及最佳實踐，歡迎同好的[反饋](https://github.com/hayasilin/api-automation-tests-guide/issues) 和 [Pull requests](https://github.com/hayasilin/api-automation-tests-guide/pulls)。

如果你是Mobile app的開發者，想了解關於Mobile app的UI自動化測試，可以參考我另外一篇[iOS 及 Android UI自動化測試規範](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/blob/master/README_zh-hant.md)

如果你是Web的開發者，想了解關於Web的UI自動化測試，可以參考我另外一篇[Web UI自動化測試規範](https://github.com/hayasilin/web-ui-automation-tests-guide/blob/master/README_zh-hant.md)

## 介紹

本規範適合在API及各種自動化測試領域已經有些許經驗的工程師，且將API自動化測試整合至團隊的CI/CD流程中。

關於測試框架，如果這裡沒提到，那就在以下的文件裡：
- Postman
  - [Postman](https://www.postman.com/)
  - [Newman](https://support.getpostman.com/hc/en-us/articles/115003710329-What-is-Newman-)

## 目錄
- [開始開發API自動化測試之前](#開始開發API自動化測試之前)
  - [API自動化測試不可或缺](#API自動化測試不可或缺)
  - [什麼是Flaky tests](#什麼是Flaky-tests)
  - [為何API自動化測試會有Flaky tests](#為何API自動化測試會有Flaky-tests)
- [讓API自動化測試可靠及穩定的最佳實踐](#讓API自動化測試可靠及穩定的最佳實踐)
  - [遵從測試三角形](#遵從測試三角形)
  - [開發前的準備](#開發前的準備)
  - [什麼樣的測試適合API自動化測試](#什麼樣的測試適合API自動化測試)
  - [避免Flaky tests的5大實用方法](#避免Flaky-tests的5大實用方法)
- [API測試的程式碼範例](#API測試的程式碼範例)
  - [Status code](#status-code)
  - [String handling](#string-handling)
  - [Array handling](#array-handling)
  - [Random handling](#random-handling)
- [常見問答](#常見問答)

## 開始開發API自動化測試之前

### API自動化測試不可或缺
- 在App或是Web服務開發中，透過API與伺服器端溝通及取得資料已是不可或缺的開發方式，包含Restful API或是GraphQL都是常見的API工具。
- 如果App或Web發生問題，除了客戶端本身的問題外，另一個大多是是API出現問題。
- 因此，有好的API自動化測試，不僅可以及早發現問題，也可避免App或Web端出現問題。
- 因為API有可能因為商業需要而不斷變動，所以測試碼也需要跟著變動，別忘了這些維護成本。
- 對QA團隊來說，如果能有API自動化測試，則可以在App出現問題時，透過API自動化測試首先確認是否Server端的問題，如果API沒問題，則可以縮減問題範圍到客戶端，因此對於開發及測試來說，API自動化測試皆是不可或缺的工具。
- 如同UI自動化測試，如果API自動化測試程式寫不好的話，你首先會遇到惡名昭彰的**Flaky tests**。
- Flaky tests會導致不可靠也不穩定的API自動化測試，不過API測試與UI測試相較之下相對單純，發生Flaky tests的機會也較低。

### 什麼是Flaky tests
- 當執行自動化測試時，有些測試項目這次會成功但下次會失敗，成功及失敗會反覆出現，即便你並沒有更動程式碼，而用人力親自去確認API其實沒問題。我們無法預測該測項下次會成功還是失敗。
- 下面是Flaky tests造成的例子。測項A跟測項C是Flaky tests因為它們成功及失敗的結果輪流出現，相對的測項B一直成功，是穩定的測項。
- 一般而言我們會把API自動化測試放入CI/CD流程之中，並根據所設定的情況去自動啟動它。最常見的方式是與Jenkins整合。如果我們測項中有Flaky tests，那麼以下例子顯示每次Jenkins測試完畢後結果都很有可能是紅燈，因為哪怕只要是1個測項失敗，整個Jenkins Job結果就會顯示錯誤，結果是團隊必須要派人力去檢查為何測試失敗。
- 以下例子只有3個測項，如果你的團隊已經埋頭苦幹先寫好了100多個測項，想像一下如果測試沒寫好裡面有Flaky tests，你的團隊人員要花多少時間去確認是否真的是API的問題，還是Flaky tests造成的。

| Test Runs   | 1st     | 2nd      | 3rd      | 4th      | 5th     | 6th      | 7th      | 8th      | 9th      | 10th     | Next run?              |
| ----------- | ------- |--------- | ---------| -------- | ------- | -------- | -------- | -------- | -------- | -------- | ---------------------- |
| Test Case A | Success | **Fail** | Success  | Success  | Success | **Fail** | Success  | Success  | **Fail** | **Fail** | **?**                  | 
| Test Case B | Success | Success  | Success  | Success  | Success | Success  | Success  | Success  | Success  | Success  | **?**                  | 
| Test Case C | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | Success  | **Fail** | **?**                  | 
| ...                                                                                                                                              |
| Jenkins     | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | **Fail** | **Fail** | **Highly likely Fail** |

- 為何Flaky tests很糟糕:
  - 團隊會對API自動化測試失去信心。
  - 如果你的CI/CD設計當API自動化測試成功後將繼續其他動作，Flaky tests將停滯住CI/CD流程。
  - 無法達成產品快速上線，更別提快速提供反饋讓團隊維持或提升品質。

- Google處理Flaky tests的經驗
  - 2015 [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
  - 2016 [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
  - 2017 [Where do our flaky tests come from](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
  
### 為何API自動化測試會有Flaky tests

**團隊內部溝通造成的Flaky因子:**
- API內容在每次改版都可能會改變。
- 不穩定的測試環境及伺服器。
- 沒有API文件造成測試程式沒有設計依據。
- 寫得不好的測試程式碼。

## 讓API自動化測試可靠及穩定的最佳實踐

### 遵從測試三角形

根據Google的[Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)這篇文章，測試三角形是想像有個三角形，底部面積最大的部分是你的單元測試，接著往上你的測試會變得越來越大，但同時測試的數量會變得越來越少。

Google建議70/20/10的比例：70%單元測試，20%整合測試，以及10%的UI測試。實際的比例會根據每個團隊而有所不同，但整體來說，比例應該如同三角形的形狀一樣。另外請避免反模式，比如團隊主要依賴UI測試，並很少使用整合測試，且幾乎沒有單元測試。

如同正常的三角形在現實世界是最穩定的結構，測試三角形也是最穩定的測試策略。

API測試位於整合測試的範疇，數量設計上應介於單元測試與UI測試之間。

### 開發前的準備
- **重要!** 負責開發API的伺服器端工程師應該提供寫好的API文件，該文件裡應該包含API的規格，Endpoints，Payload等內容，各回傳參數是否可為null，各回傳參數是否可為空值等重要資訊，由此作為依據才能開發API自動化測試程式碼。

### 什麼樣的測試適合API自動化測試

**推薦**
- 驗證API回傳的status code。
- 驗證API文件定義為絕對不為Null的值。
- 驗證API文件定義為絕對不為Null的值，且定義不會為空值。

**不推薦**
- 驗證API回傳物件會等於某個特定的數量。
- 結構改變（ex: 確認Array是否變成Object）。
- 型別改變（ex: 確認String是否變成Number）。

**API回傳內容可能數量龐大，驗證全部回傳內容會造成維護成本過高，因此挑選最重要欄位驗證即可。**

### 避免Flaky tests的5大實用方法
1. 請負責開發API的伺服器端工程師一定要提供API文件，該文件需清楚寫下各欄位的規則。
2. 確認所有API可能回傳的內容，包含錯誤情況。寫測項時當下所看到的API回傳內容並不代表所有狀態下都會回傳相同的內容，將所有可能的API回傳內容都思考過並讓測試能充分對應各種情境才能避免Flaky。
3. 每次完成測試程式碼後，請先將所有測項一起執行，並重複約10次的測試，確保新加入的程式碼不會造成Flaky tests。
4. 選擇最成熟的API開始寫測試程式，也就是該API不會因為短期商業計劃而不斷改變。
5. 不要過度設計你的測試程式，讓每個API測試程式既簡單又很容易維護。

## API測試的程式碼範例

**以下使用javascript及Postman作為範例**

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

### Set environment variables depends on the response of previous API
```javascript
pm.environment.set("token", data.token);
```

## 常見問答
1. 在CI/CD流程中，多久需執行一次API自動化測試確認API正常運作？
  - **建議1天3次左右即可**。
    - 因為團隊裡不論是客戶端還是伺服器端的工程師，往往需在工作時間時更新程式碼或更新環境，而API測試可能會測試在不同的環境上（Beta / Release），過度頻繁測試API只是讓Flaky tests發生的機會大增。除非你的團隊能確保一個超級穩定的測試環境及測試資料（但又是另1種成本），不然其實只需要在1天中選3個時間執行API自動化測試確認API沒問題即可，而結果也足以提供團隊即時的反饋。
    - 因為API測試相較於UI測試來的單純，且執行快速，如可能建議每當伺服器端工程師更新API的程式碼時，除了執行伺服器端自己的單元測試外，也建議能一併執行API自動化測試，將可及早發現問題。

2. 使用哪種測試框架及工具比較好？
  - 因為API測試相對於UI測試來說相對單純，各種工具應該都可以達到不錯的結果。我本身使用Postman，Postman是一個優秀的API工具，同時也有提供自動化測試工具名為Newman。