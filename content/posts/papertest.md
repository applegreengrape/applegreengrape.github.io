+++
title = "Google Spreadsheets Paper Test"
date = "2019-08-13"
aliases = ["Post","post","blog-page"]
tags = ["Google Apps Script", "Google Spreadsheets", "Paper Test"]
disqusShortname = "http-applegreengrape-com"
+++

I was aspired by the alpaca's idea of using google spreadsheet as a test environment for auto algo trading. So I decided to make a version for bitcoin paper test trading. For alpaca's stock version, you can find the details [here](https://medium.com/automation-generation/manage-your-stocks-from-google-spreadsheet-using-api-43026db44289). The google spreadsheet will look like this: 

{{< image src="/static/img/btc_spreadsheet.png" alt="btc_spreadsheet" position="left" width=60% height="80%">}} 

I am using [coinbase api](https://api.coinbase.com/v2/) to get the lastest Bitcoin price. And there is already one google app script class that you can use - [`Class UrlFetchApp`](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app). Then you need to call this class in your function.

```go
function myFunction() {
 // Make a GET request and log the returned content.
  var response = UrlFetchApp.fetch('https://api.coinbase.com/v2/prices/BTC-USD/spot');
  var fact = response.getContentText();
  var json = JSON.parse(fact)
 
  Logger.log(json.data.amount)
  var sheet = SpreadsheetApp.getActiveSheet();
  var lastRow = sheet.getLastRow();
  
  
  for (var i = 2; i <= lastRow; i++){
// update each row to set the latest bitcoin price
  sheet.getRange(i,1).setValue([json.data.amount]);
  
  }
  ...
}
```

Now we need to do some small calculation to update the position status (open, close and closed profit/loss).  We can use the [class sheet](https://developers.google.com/apps-script/reference/spreadsheet/sheet). Then the script will be some thing like this:
```go
function myFunction() {
    ...
    for (var i = 2; i <= lastRow; i++ ){
        
        var status = sheet.getRange(i,9).getValues();
        var price = sheet.getRange(i,1).getValues();
        var direction = sheet.getRange(i,2).getValues();
        var buy = sheet.getRange(i,3).getValues();
        var sell = sheet.getRange(i,4).getValues();
        var limit = sheet.getRange(i,5).getValues();
        var stop = sheet.getRange(i,6).getValues();
        var size = sheet.getRange(i,7).getValues();
    
    
    if ( status[0][0] == "closed"){}
    else { 
      
      if ( direction[0][0] =="long"){ if(price-buy<limit && buy-price<stop){
        //  Logger.log("still open")
        sheet.getRange(i,9).setValue("open");
        var pl = (price-buy)*size
        sheet.getRange(i,8).setValue(pl);
      
      }else{
        // Logger.log("closed")
        sheet.getRange(i,9).setValue("closed");
        var pl = (price-buy)*size
        sheet.getRange(i,8).setValue(pl);
        
      }}
      
      else{
        if( direction[0][0] =="short" ){
        if(sell-price<limit && price-sell<stop){
        //  Logger.log("still open")
        sheet.getRange(i,9).setValue("open");
        var pl = (sell-price)*size
        sheet.getRange(i,8).setValue(pl); 
       
        }else{
        // Logger.log("closed")
        sheet.getRange(i,9).setValue("closed");
        var pl = (sell-price)*size
        sheet.getRange(i,8).setValue(pl); 
        }
       }
      }
    }
  }
}
```
You can find [the source code here](https://github.com/applegreengrape/btc-google-spreadsheet-app-script/blob/master/btcFunction). Contact me 👉📱message (+447479275693) if you want some customized google apps scripts.
