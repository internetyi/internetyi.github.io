---
layout: post
title: 自编用PHP来获取GOOGLE PR值代码
excerpt: 谷歌改变了他们PR检查API网址后，网上有些提供PR检测的工具都不能正常工作。下面的代码已经修改使用了谷歌PR查询 的最新接口，直接从google查询。
categories: [后端开发]
tags: [PHP]
---

谷歌改变了他们PR检查API网址后，网上有些提供PR检测的工具都不能正常工作。下面的代码已经修改使用了谷歌PR查询 的最新接口，直接从google查询。

如果你需要批量检查网页排名，使用诸如站长之家的pr一个个查效率会慢很多所以我花了一点时间写了已段代码用php来替代网页排名检查工具。

只要将下面代码拷贝下来建一个`pr.class.php`文件再在需要用到的地方 `include` 一下就好了，很方便。

代码如下：

**pr.class.php**

``` php
<?php
class PR { 
		 public function get_google_pagerank($url) { 
			 $query="http://toolbarqueries.google.com/tbr?client=navclient-auto&ch=".$this->CheckHash($this->HashURL($url)). "&features=Rank&q=info:".$url."&num=100&filter=0";
			 // $data=file_get_contents($query);
			 $data = self::curl_get_contents($query);
			 $pos = strpos($data, "Rank_");
			 if($pos === false){} else{
			 $pagerank = substr($data, $pos + 9);
			 return $pagerank;
		 	}
		 }

		 public function curl_get_contents($url, $timeout = 5){
		    $cookiepath=dirname(__FILE__)."/cookie.txt";
		    $userAgent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36';
		    $referer = $url;
		    $ch = curl_init();
		    curl_setopt($ch, CURLOPT_URL, $url);
		    curl_setopt($ch, CURLOPT_TIMEOUT, $timeout);
		    curl_setopt($ch, CURLOPT_USERAGENT, $userAgent);
		    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
		    curl_setopt($ch, CURLOPT_COOKIEJAR, $cookiepath);	//COOKIE的存储路径,返回时保存COOKIE的路径
		    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);        //跟踪301		
		    curl_setopt($ch, CURLOPT_HEADER, false);
		    @curl_setopt($ch, CURLOPT_ANS_USE_GLOBAL_CACHE, true);
		    curl_setopt($ch, CURLOPT_ANS_CACHE_TIMEOUT, 86400); // 缓存一天
		    $content = curl_exec($ch);
		    curl_close($ch);
		    return $content;
		}

		public function HashURL($String){
			$Check1 = $this->StrToNum($String, 0x1505, 0x21);
			$Check2 = $this->StrToNum($String, 0, 0x1003F);

			$Check1 >>= 2;
			$Check1 = (($Check1 >> 4) & 0x3FFFFC0 ) | ($Check1 & 0x3F);
			$Check1 = (($Check1 >> 4) & 0x3FFC00 ) | ($Check1 & 0x3FF);
			$Check1 = (($Check1 >> 4) & 0x3C000 ) | ($Check1 & 0x3FFF);

			$T1 = (((($Check1 & 0x3C0) < < 4) | ($Check1 & 0x3C)) <<2 ) | ($Check2 & 0xF0F );
			$T2 = (((($Check1 & 0xFFFFC000) << 4) | ($Check1 & 0x3C00)) << 0xA) | ($Check2 & 0xF0F0000 ); 			  			return ($T1 | $T2); 		}                                  public function CheckHash($Hashnum){ 			$CheckByte = 0; 			$Flag = 0; 			  			$HashStr = sprintf('%u', $Hashnum) ; 			$length = strlen($HashStr); 			  			for ($i = $length - 1; $i >= 0; $i --) {
				$Re = $HashStr{$i};
				if (1 === ($Flag % 2)) {
					$Re += $Re;
					$Re = (int)($Re / 10) + ($Re % 10);
				}
			$CheckByte += $Re;
			$Flag ++;
			}

			$CheckByte %= 10;
			if (0 !== $CheckByte) {
				$CheckByte = 10 - $CheckByte;
				if (1 === ($Flag % 2) ) {
					if (1 === ($CheckByte % 2)) {
						$CheckByte += 9;
					}
					$CheckByte >>= 1;
				}
			}

			return '7'.$CheckByte.$HashStr;
		}
}
```

下面是使用代码示例：

``` php
    include("pr.class.php");
    $url='http://blog.zhny.qzz.io/';
    $pr = new PR();
    echo $pr->get_google_pagerank($url);
```

