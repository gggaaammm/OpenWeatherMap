# Openweathermap API 操作教學
![](https://i.imgur.com/FTmBDep.png)

此篇章只針對iottalk上Sensor的部分替換成OpenWeatherMap API操作，其餘教學請參照原Dashboard篇章。

Dashboard教學：[請點這裡](https://hackmd.io/5LqVk4MBSCinRXQderD_Jw?both)

## 事前準備
* 一隻申請API KEY的E-mail帳號
* 安裝python packages: requesets(必須), json(必須), pprint(選填)
```
pip3 install requesets
pip3 install json
pip3 install pprint
```
* 做好[Dashboard教學](https://hackmd.io/5LqVk4MBSCinRXQderD_Jw?both)的安裝


## 1. 申請OpenWeatherMap的帳號
申請網址：https://home.openweathermap.org/users/sign_in
點開連結，出現以下畫面
![](https://i.imgur.com/Zpw7bNQ.png)
接著點選**Create an Account**
![](https://i.imgur.com/WSONiDe.png)
輸入資料後，到自己的信箱開通帳號，第一步驟完成。

## 2. 取得API Key
點選API 找到適合的API function
免費版帳號有兩個選擇(此篇教學以One Call API當作範本)
1. Current Weather Data 
2. One Call API

點選**Subscribe**
![](https://i.imgur.com/Lumwtge.png)
(註:API Key是可以共用的。也就是說一組API Key可以同時用在這兩個API call上面。)
接著點選API Keys
![](https://i.imgur.com/HtNLAJJ.png)
帳號通常會預設一組API Key,為求保險,這邊再申請另一組API Key名為"iottalk"
![](https://i.imgur.com/OCJXpKW.png)

## 3. Coding：取得資料

完整範例code參考：[點這裡](https://github.com/gggaaammm/OpenWeatherMap)
在這個步驟，僅顯示由API call所獲得的資料
```
# import required modules 
import requests, json 
# Enter your API key here (Please replace API Key with yours!)
api_key = "2f634d46796850dcc4c3a54a254759d9" 
latitude = "24.7857152"
longitude = "120.995840"(the coordinates of Hsinchu)
# complete_url variable to store 
complete_url = base_url +"lat="+latitude+"&lon="+longitude +"&exclude=hourly,daily,minutely&appid=" + api_key +"&lang=zh_tw&units=metric" #ver. 2.0
# get method of requests module return response object 
response = requests.get(complete_url) 
# json method of response object 
# convert json format data into python format data 
x = response.json() 
pprint(x)
```
會顯示出下列資訊(介紹如下)：
![](https://i.imgur.com/9ZKHg0J.png)

## 4. Coding: 將取得的資料串接至IoTtalk
基本上，與DummyDevice相同。註冊該Device model後，定時送資料至iottalk server。
以下是關於如何註冊Device、連線至iottalk的部分
```

#  注意你用的 IoTtalk 伺服器網址或 IP  
ServerURL = 'https://demo.iottalk.tw' # with SSL secure connection
Reg_addr = None #if None, Reg_addr = MAC address #本來在 DAN.py 要這樣做 
# Note that Reg_addr 在以下三句會被換掉! # the mac_addr in DAN.py is NOT used
mac_addr = 'CD8600D38' + str( 517 )  # put here for easy to modify :-)
# 若希望每次執行這程式都被認為同一個 Dummy_Device, 要把上列 mac_addr 寫死, 不要用亂數。
Reg_addr = mac_addr   # Note that the mac_addr generated in DAN.py always be the same cause using UUID !
#以下三列請根據自己的需求作修改
DAN.profile['dm_name']='OpenWeatherMap'   # you can change this but should also add the DM in server
DAN.profile['df_list']=['hscli_Pressure_I', 'hscli_Humidity_I', 'hscli_Temperature_I', 'hscli_WindSpeed_I', 'hscli_WeatherDes_I', 'hscli_UV_I', 'hscli_FeelsLike_I']   # Check IoTtalk to see what IDF/ODF the DM has
DAN.profile['d_name']= "HSCLi_."+ "Hsinchu" +"_"+ DAN.profile['dm_name'] # None

DAN.device_registration_with_retry(ServerURL, Reg_addr) 
print("dm_name is ", DAN.profile['dm_name']) ; print("Server is ", ServerURL)
```

## 5. IoTtalk設定
打開[iottalk](https://demo.iottalk.tw),在Device Feature Management中新增自己的Device Feature, 並在Device Model Management新增自己的Device Model，這兩者資訊必須跟上個步驟相同。
>如何新增Device Feature:[教學影片](https://youtu.be/3d3xeCUGYxs)
>如何新增Device Model:[教學影片](https://youtu.be/HTR-QkqKUOM)

![](https://i.imgur.com/UonFItC.png)
![](https://i.imgur.com/GHxxtyQ.png)


## 6. IDF執行、監看
```
python3 OpenWeatherMapdemo.py
```
執行後，即可監看Device Model是否成功註冊、API是否正常運作。
在此步驟iottalk**可能出現的錯誤**有以下幾項：
1. **Mac Address設定錯誤**(解決: 在Demo.py確認自己的mac_addr是"獨一無二的，重複的address會造成不可預期的錯誤")
2. **某些Device Features不存在**(解決： 代表Demo.py的Device Feature與iottalk上有落差，請確認是否完全一致)


## 7. ODF執行、監看
```
./startup.sh
```
此部分與[Dashboard教學](https://hackmd.io/5LqVk4MBSCinRXQderD_Jw?both)相同
這邊我做了些許的修改：
>在DAI.py修改Device Name以及Device model name成自己專屬的Device model(#Line 62)