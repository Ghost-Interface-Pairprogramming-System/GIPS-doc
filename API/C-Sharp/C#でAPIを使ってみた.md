# C#でAPIを使ってみた

## 使用api

- [WeatherHacks](http://weather.livedoor.com/weather_hacks/qa)

<!-- Weather Hacksが何なのか、何が取得できるかの概要が欲しい -->

Livedoorが提供する天気情報のWebAPI。
天気・気温・天気の概要文の他その天気のアイコンまで取得できる。

ただし商用利用は不可。このWebAPIに限らず利用規約はきっちり読んで使おう。

## 使用Package 

- DynamicJson

<!-- 突然DynamicJsonを出してくるより、それは何ができて、何のために追加するのかの説明がほしい -->

WeatherHacksでは値をjsonで取得できるため、jsonをC#で扱いやすくするためのライブラリを追加した。

DynamixJsonはその名の通り、jsonをC#のDynamic型に変換し、いちいちjsonの受け皿になるクラス作らなくてもjsonが扱える便利なやつである。

## パッケージのDL方法

#### 	ツール　⇒　Nugetパッケージマネージャー　⇒　

#### ソリューションのNugetパッケージマネージャーの管理　⇒　参照　⇒

#### 「 DynamicJson」を検索しインストール


## 実際にコードを書いてみる

<!--
いきなり個々の機能について解説し始めると何をやっているのかイメージしにくい。
まずは何を作るのか、どんなものが完成するのかのイメージを読者に持たせた方が良さそう。
-->

今回は、ユーザが任意の地域の天気情報を選択して表示できるコンソールアプリケーションを作成してみた。
WeatherHacksでは地域にIDが割り振られており、そのIDで地域を指定する形式になっている。

なので、
- そのIDはXMLの形で取得できるので、そのXMLからIDを全件取得、一覧を表示。
- その表示されたIDからユーザがIDを選択し、入力
- そのIDの天気情報を取得、表示する

という流れで作成した。

- ### Main

  ##### ユーザがIDを入力し天気概要を表示させる場所

  ```c#
  
  static void Main(string[] args)
   {	
      //GetCityList()から府県名とIDを取得
       var citys = GetCityList();
  
       // 1.city(府県区分とID)一覧表示
       citys.ForEach((city) =>
       {
           Console.WriteLine("{0}: ID: {1}", city.Name, city.Id);
       });
  
       // 2.ユーザがID入力
       Console.WriteLine("----------------------------------------------------");
       Console.WriteLine("↑から検索したいIDを入力してください");
       var cityId = Console.ReadLine(); // 260010など
       Console.WriteLine("----------------------------------------------------");
  	
       // 3.入力された天気概要を表示させる
       // GetWeatherTexから検索したいIDの天気概要を取得
       Console.WriteLine(GetWeatherText(cityId));
       Console.ReadKey();
   }
  ```


- ####  GetCityList()

##### 	     http://weather.livedoor.com/forecast/rss/primary_area.xmlから

##### 　　　全ての府県名とIDをセットで取得し一覧表示出来るようにする

```c#
 //名前とIDだけを全件抽出
        private static List<City> GetCityList()
        {
            var citys = new List<City>();
            var url = "http://weather.livedoor.com/forecast/rss/primary_area.xml";
            var req = WebRequest.Create(url);

            using (var res = req.GetResponse())
            using (var s = res.GetResponseStream())
            {
                var xml = XDocument.Load(s);

                //各ノードのNameを名前空間なしの要素名にする
                foreach (var e in xml.Descendants()) e.Name = e.Name.LocalName;
               
                var prefs = xml.XPathSelectElements("rss/channel/source/pref");

                foreach (var pref in prefs)
                {
                   
                    var tmpCitys = pref.XPathSelectElements("city").Select((city) => {
                        var name = city.Attribute("title").Value;
                        var id = city.Attribute("id").Value;

                        return new City(id, name);
                    });

                    citys = citys.Concat(tmpCitys).ToList();
                }
            }
            //Mainに府県全てのIDと名前が入ったデータを返す
            return citys;
        }
```







- ###  GetWeatherText()

#####   	    入力されたID（府県の区分）の天気概要を表示させる

   

```c#
		const string NO_VALUE = "---";   
	　　//天気概要
	   // string cityId : Main側で入力したID
        private static string GetWeatherText(string cityId)
        {
            
            var url = "http://weather.livedoor.com/forecast/webservice/json/v1?city=" + 					   cityId;
            var req = WebRequest.Create(url);

            using (var res = req.GetResponse())
                using (var s = res.GetResponseStream())
            {
                dynamic json = DynamicJson.Parse(s);

                //今日の天気情報
                dynamic today = json.forecasts[0];

                //日付
                string date = today.date;
                //天気(晴れ、曇り、雨など)
                string telop = today.telop;

                //最高気温
                var sbTempMax = new StringBuilder();
                dynamic todayTemperatureMax = today.temperature.max;
                if (todayTemperatureMax != null)
                {
                    sbTempMax.AppendFormat("{0}℃", todayTemperatureMax.celsius);
                }
                else
                {
                    sbTempMax.Append(NO_VALUE);
                }

                //最低気温
                var sbTempMin = new StringBuilder();
                dynamic todayTemperatureMin = today.temperature.min;
                if (todayTemperatureMin != null)
                {
                    sbTempMin.AppendFormat("{0}℃", todayTemperatureMin.celsius);
                }
                else
                {
                    //もしも情報がなければ ---------が表示される
                    sbTempMin.Append(NO_VALUE);
                }

                //天気概況文
                var situation = json.description.text;

                //府県表示
                var pr = json.title;


             return string.Format("{0}\n{1}\n天気 {2}\n最高気温 {3}\n最低気温 {4}\n\n{5}",
                                     pr,
                                     date,
                                     telop,
                                     sbTempMax.ToString(),
                                     sbTempMin.ToString(),
                                     situation
                                    );
            }

        }
```



- ### Cityクラス

##### 　名前とIDを管理するクラス

```c#
class City
    {
        public string Id { get; set; }
        public string Name { get; set; }

        public City(string id, string name)
        {
            Id = id;
            Name = name;
        }
    }
```





## これらを組み合わせて表示させてみた

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.Xml;
using System.Xml.Linq;
using System.Xml.XPath;
using Codeplex.Data;

namespace api_1
{
    class Program
    {
        const string NO_VALUE = "---";

        static void Main(string[] args)
        {
            var citys = GetCityList();

            // 1.city(府県区分とID)一覧表示
            citys.ForEach((city) =>
                          {
                              Console.WriteLine("{0}: ID: {1}", city.Name, city.Id);
                          });

            // 2.ユーザがID入力
            Console.WriteLine("----------------------------------------------------");
            Console.WriteLine("↑から検索したいIDを入力してください");
            var cityId = Console.ReadLine(); // 260010など
            Console.WriteLine("----------------------------------------------------");

            // 3.入力された天気概要を表示させる
            Console.WriteLine(GetWeatherText(cityId));
            Console.ReadKey();
        }	

        //名前とIDだけを全件抽出
        private static List<City> GetCityList()
        {
            var citys = new List<City>();
            var url = "http://weather.livedoor.com/forecast/rss/primary_area.xml";
            var req = WebRequest.Create(url);

            using (var res = req.GetResponse())
                using (var s = res.GetResponseStream())
            {
                var xml = XDocument.Load(s);

                //各ノードのNameを名前空間なしの要素名にする
                foreach (var e in xml.Descendants()) e.Name = e.Name.LocalName;

                var prefs = xml.XPathSelectElements("rss/channel/source/pref");

                foreach (var pref in prefs)
                {

                    var tmpCitys = pref.XPathSelectElements("city").Select((city) => {
                        var name = city.Attribute("title").Value;
                        var id = city.Attribute("id").Value;

                        return new City(id, name);
                    });

                    citys = citys.Concat(tmpCitys).ToList();
                }
            }
            //Mainに府県全てのIDと名前が入ったデータを返す
            return citys;
        }

        //天気概要
        private static string GetWeatherText(string cityId)
        {

            var url = "http://weather.livedoor.com/forecast/webservice/json/v1?city=" + 					   cityId;
            var req = WebRequest.Create(url);

            using (var res = req.GetResponse())
                using (var s = res.GetResponseStream())
            {
                dynamic json = DynamicJson.Parse(s);

                //今日の天気情報
                dynamic today = json.forecasts[0];

                //日付
                string date = today.date;
                //天気(晴れ、曇り、雨など)
                string telop = today.telop;

                //最高気温
                var sbTempMax = new StringBuilder();
                dynamic todayTemperatureMax = today.temperature.max;
                if (todayTemperatureMax != null)
                {
                    sbTempMax.AppendFormat("{0}℃", todayTemperatureMax.celsius);
                }
                else
                {
                    sbTempMax.Append(NO_VALUE);
                }

                //最低気温
                var sbTempMin = new StringBuilder();
                dynamic todayTemperatureMin = today.temperature.min;
                if (todayTemperatureMin != null)
                {
                    sbTempMin.AppendFormat("{0}℃", todayTemperatureMin.celsius);
                }
                else
                {
                    //もしも情報がなければ ---------が表示される
                    sbTempMin.Append(NO_VALUE);
                }

                //天気概況文
                var situation = json.description.text;

                //title
                var pr = json.title;


                return string.Format("{0}\n{1}\n天気 {2}\n最高気温 {3}\n最低気温 			　　　　　　　　　　　　　　　　　　　　　　　{4}\n\n{5}",
                                     pr,
                                     date,
                                     telop,
                                     sbTempMax.ToString(),
                                     sbTempMin.ToString(),
                                     situation
                                    );
            }

        }
    }

    class City
    {
        public string Id { get; set; }
        public string Name { get; set; }

        public City(string id, string name)
        {
            Id = id;
            Name = name;
        }
    }

}

```





## 実行結果

####   今回は「大分 : 440010」で検索してみた

![キャプチャ](https://user-images.githubusercontent.com/40161221/48674369-a1f6f580-eb8e-11e8-976e-a42dc83e8b9c.PNG)



### 

