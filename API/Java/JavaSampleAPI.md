# JavaでiTunesSearchAPIを使ってみる。



#### 環境

- ------

- eclipse4.7


- java８




#### 使用ライブラリ

- ------

- gson-2.8.5

##### Gson

googleから公開されているjavaライブラリ

Javaオブジェクト⇒JSON	`gson.toJson()`

JSON⇒Javaオブジェクト	`gson.fromJson()`

相互変換ができる。

Github - google/gson https://github.com/google/gson





#### 使用API

- ------

  iTunes Search API

  https://affiliate.itunes.apple.com/resources/documentation/itunes-store-web-service-search-api/


リクエストURL

http://itunes.apple.com/search?　

？の後に検索パラメータを記述、　

例

```
http://itunes.apple.com/search?term="+key+"&country=JP&media=music&entity=album&lang=ja_jp&limit=5"  
```

termはキーワード、mediaは音楽や映画などのジャンル、entityでさらに絞り込みができる。





##### アーティスト名とアルバム名を検索

------



```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Scanner;

import com.google.gson.Gson;

public class Sample {

	public static void main(String[]args){
		Scanner sc = new Scanner(System.in);
		System.out.println("キーワードを入力");
		String key = sc.nextLine();
		StringBuilder sb = new StringBuilder();
			try{
				URL url = new URL("https://itunes.apple.com/search?"
						+ "term="+key+""  //キーワード
						+ "&country=JP"   //国コード
						+ "&media=music"  //種類
						+ "&entity=album" //種類をさらに絞る
						+ "&lang=ja_jp"   //言語
						+ "&limit=5");    //出力数
		HttpURLConnection con = (HttpURLConnection) url.openConnection(); //コネクション取得
		con.connect(); 	//接続
		BufferedReader ｒ = new BufferedReader(new InputStreamReader(
		con.getInputStream(),"UTF-8")); //読み込み
		String line = "";
		while ((line = ｒ.readLine()) != null) {   //一行づつ読み込んでsbに書き込んでいく
		  sb.append(line);
		}
		ｒ.close();
		con.disconnect();//切断
		}catch(Exception e){
		e.printStackTrace();
		}

		// 読み込んだjsonをパース
		Gson gson = new Gson();		//GSON生成
		SampleList it = gson.fromJson(sb.toString(),SampleList.class);   //JSONデータをJavaオブジェクトに変換
		System.out.println("検索結果"+it.resultCount+"件");

		for(int n =0;n < it.results.size();n++){
		System.out.println("アーティスト名："+it.results.get(n).artistName+"\t\tアルバム名："+it.results.get(n).collectionName);


		}
	}
}
```



取得した値を格納しておくクラス

```
import java.util.List;

public class SampleList {
	//リスト生成
	List<SampleList>results;
	int resultCount;		//取得件数
	String artistName;		//アーティスト名
	String collectionName;  //アルバム名
}
```

  

実行結果

------

谷で検索

```
キーワードを入力
谷
検索結果5件
アーティスト名：久石 譲		アルバム名：風の谷のナウシカ ~はるかな地へ~
アーティスト名：横山だいすけ・三谷たくみ(NHKおかあさんといっしょ)		アルバム名：NHKおかあさんといっしょ 最新ベスト「みんなのリズム」
アーティスト名：島谷ひとみ		アルバム名：15th Anniversary SUPER BEST
アーティスト名：横山だいすけ・三谷たくみ(NHKおかあさんといっしょ)		アルバム名：「おかあさんといっしょ」メモリアルアルバム~キミといっしょに~
アーティスト名：横山だいすけ・三谷たくみ(NHKおかあさんといっしょ)		アルバム名：NHKおかあさんといっしょ 最新ベスト それがともだち

```

