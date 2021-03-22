# Python 爬蟲蟲

哈囉~各位帥哥美女，是不是有一些困擾，例如 : 用手動的方式去[公開資訊戰](https://mops.twse.com.tw/mops/web/index)找[年報](https://doc.twse.com.tw/server-java/t57sb01)，然後在輸入你要分析的那家公司和年份，再下載pdf檔；之後在從pdf檔案中找尋你要的資訊，一個一個的把值打進你的Excel 或 Word 報告裡面。

身為資工的我看在眼裡，心理非常的想寫一套程式幫各位一手包辦這些事情，且在短短幾秒鐘內完成。

於是.....ㄐㄧㄤ ㄐㄧㄤ ~~~!!!!，就是底下我將介紹的內容。

p.s.等等看到程式碼會心悸、噁心、癲癇和孕吐的人，拜託你先看醫生，然後再回來努力看到第二段就好，謝謝(´・ω・｀)

code本體 -> [傳送門](https://github.com/yoyoisaboy/CompayYearReport_Python/blob/main/%E8%82%A1%E6%9D%B1%E6%9C%83%E5%B9%B4%E5%A0%B1.ipynb)


## 第一段 : 創一個虛擬網路，模擬我們手動執行
```
import time
from selenium.webdriver import Chrome #需安裝 : https://pypi.org/project/selenium/
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager #需安裝 :  https://pypi.org/project/webdriver-manager/

driver = webdriver.Chrome(ChromeDriverManager().install())
```

:::success
基本上想做到按鍵觸發、鍵盤輸入都需要用到[selenium](https://pypi.org/project/selenium/)這套件，那我模擬的環境是用chrome，所以執行這段會跑出一個乾淨的網頁。
* 執行結果:
![](https://i.imgur.com/eGDVZ3k.png)
:::


## 第二段 : 創一個虛擬網路，模擬我們手動執行
```
year = 108 #想找的年度
number = 2330 #想找的公司代碼，此例為台機電
url = f"https://doc.twse.com.tw/server-java/t57sb01?step=1&colorchg=1&co_id={number}&year={year}&mtype=F&"
driver.get(url)
print(url)
```
:::success
這裡就是使用者輸入想要找的年度和公司代碼，所以看不懂程式碼的你只要改這裡的值就好，其他程式碼你一律直接執行到最後就好了。

url字串可以發現?後面是分別加入我們輸入的值，利用這些參數就可以到查詢的結果，無須手動方式輸入，這叫[url request](https://icodding.blogspot.com/2015/08/blog-post_16.html)的方式呼叫。

參數分別有 
* step = 1
以市場別/產業別查詢代號
* colorchg=1
背景變白色
* co_id={number} -> 這寫法是python獨特寫法的
公司代號
* year={year}
年度
* mtype=F
資料類型
Q : 我怎知道參數的名稱?
A : 你可以到網站按F12，看看你手動輸入的格子他的name是什麼。例如mtype
![](https://i.imgur.com/pdpn8y1.png)
:::
## 第三段 : 取得"股東會年報"，點pdf
```
link = driver.find_elements_by_xpath('//a[contains(., "F04") and contains(@href, ".pdf")]')[0].get_attribute('href') #取得股東會年報
pdf_name = link.split('\"')[-2] #pdf名稱
driver.find_element_by_xpath(f"//a[contains(@href,'{link}')]").click() #pdf按下去
print("link = ",link)
print("pdf_name = ",pdf_name)
```
* 執行結果:
link =  javascript:readfile2("F","2330","2018_2330_20190605F04.pdf");
pdf_name =  2018_2330_20190605F04.pdf
:::success
link 是 從這網站找到"F04" 且 ".pdf" 的<a href的文字 。(xpath:[詳細內容](https://selenium-python-zh.readthedocs.io/en/latest/locating-elements.html))

```
<a href="javascript:readfile2(&quot;F&quot;,&quot;2330&quot;,&quot;2018_2330_20190605F04.pdf&quot;);">2018_2330_20190605F04.pdf</a>
```
之後找到link取得pdf_name，找到點擊下去，如下。
![](https://i.imgur.com/en7vSzH.png)
你會發現它新增了一個分頁~~
:::
## 第四段 : 切換瀏覽器分頁

```
windows=driver.window_handles  #獲得當前瀏覽器所有視窗
driver.switch_to.window(windows[-1]) #換到最新的視窗
time.sleep(3) #休息3s
url = driver.current_url #看看目前的url
url 
```
* 執行結果 :
`'https://doc.twse.com.tw/server-java/t57sb01'`
:::success
可以發現目前的模擬瀏覽器停留在新的分頁。
之所以要time.sleep是要評估開啟瀏覽器需要幾秒，沒設sleep的話可能會跟不上程式的速度而產生錯誤，這點需要特別注意。
:::
## 第五段 : 切換瀏覽器分頁
```
from selenium.webdriver.common.action_chains import ActionChains #模擬滑鼠&鍵盤觸發事件，參數:https://www.selenium.dev/selenium/docs/api/py/webdriver/selenium.webdriver.common.action_chains.html
from selenium.webdriver.common.keys import Keys

left_click = driver.find_element_by_link_text(pdf_name)
ActionChains(driver).double_click(left_click).perform() #左擊

windows=driver.window_handles  #獲得當前瀏覽器所有視窗
driver.switch_to.window(windows[-1]) #換到最新的視窗
time.sleep(3) #休息3s
url = driver.current_url #看看目前的url
driver.quit()
print(url) #確實進入pdf頁面
```
* 執行結果:
`'https://doc.twse.com.tw/pdf/2018_2330_20190605F04_20201228_142438.pdf'`
:::success
ActionChains 是模擬滑鼠點擊，點擊.pdf的連結，切到最新頁面。
由結果知道原來這個pdf檔案是在這個url，也可以發現說他最後一串url的數字是隨機的，並且還具有時效性。
![](https://i.imgur.com/eej9xCf.png)

然後取完url後，再把虛擬網頁關掉。
:::
## 第六段 : 自動下載
```
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_experimental_option(
    'prefs', {
     "download.default_directiory": './',
     "download.prompt_for_download": False,
     "download.directory_upgrade": True,
     "plugins.always_open_pdf_externally":True
    }
) 
browser = webdriver.Chrome(ChromeDriverManager().install(),options=chrome_options)
browser.get(url)
time.sleep(15)
browser.quit()
```
:::success
根據上一段可以知道我們還需要做點擊才能夠下載，那我這方法是希望能夠不用點擊即可下載，原因是下載圖示是抓不到的資訊，所以需要用其他方法完成他。
![](https://i.imgur.com/Z6YR1zv.png)
Options來設定:
1. default_directiory: './' 預設下載地址
2. prompt_for_download: False 定義下載不彈窗
3. directory_upgrade": True 得到default_directiory位置
4. always_open_pdf_externally :True 打開pdf

之後.get(url)就可以根據你設定的參數達成無點擊下載了。
:::

就這樣可以把熱騰騰的資料在短短幾秒抓下來咯~~

ㄙㄨㄚˋ 