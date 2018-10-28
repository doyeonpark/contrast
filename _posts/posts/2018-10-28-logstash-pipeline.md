---
title:  "로그스태시 파이프라인 만들기"
date:   2018-10-28 22:00:00
tags: [logstash, filebeat, elk, logger]
categories: posts
---

ElasticSearch와 Filebeat와의 호환성, 다른 어플리케이션보단 조금 더 프로그래밍적인 파이프라인 설정방식을 사용한다는 이유로 logstash를 쓰기로 했다. 우여곡절끝에 파이프라인 설정을 마치나니 세상에 공짜는 없다는 걸 느꼈다. 어떤 기술을 제대로 이해하고 올바른 방식으로 활용하려면 시행착오를 겪어야한다는걸 배웠다. 까먹기 전에 디테일하게 기록해놓기로 했다.

### 디버깅 모드 세팅하기

파일비트-카프카-로그스태시-엘라스틱서치 구간을 만든다음에 최초의 설정을 테스트하다가, 쉽게 끝나지 않으리라는걸 깨닫고 디버깅 모드를 세팅했다. 이 판단이 더 빠르게 이뤄졌음 좋았을텐데. 디버깅 세팅을 할떈 [이 포스트](https://deviantony.wordpress.com/2014/06/04/logstash-debug-configuration/)를 참조했다.

{% highlight json %}
input {
  stdin { } # 커맨드라인에서 받은 로그를 input으로 받는다
}

filter {
  # 필터 세팅
}

output {
  stdout { codec => "json" }
  file { codec => "json" path => "tmp/output.json" }
  # 로그스태시 home 디렉토리에서 디버깅 결과 로그를 파일로 남긴다
  # 이때 권한문제가 발생하면 도커 attach된 상태에서 권한 주는 것을 잊지말자
}
{% endhighlight %}

### 파일비트에서 전달된 raw data의 형태

파일비트 -> 카프카 -> 로그스태시로 들어올때 로그라인은 아래와 같다. json 형식에 맞춰서 줄바꿈을 해줬지만 실제로는 한줄짜리 라인으로 온다. 

{% highlight json %}
{"@timestamp":"2018-10-25T07:53:39.249Z","@metadata":{"beat":"filebeat","type":"doc","version":"6.4.2","topic":"kr-r2"},"message":"2018-10-11 05:40:30 172.0.0.1 GET /news/updates/detail bbmNo=89 443 - 10.1.30.99 Mozilla/5.0+(Windows+NT+6.1;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/69.0.3497.100+Safari/537.36 https://test.webpage.com/main 200 0 0 288","prospector":{"type":"log"},"input":{"type":"log"},"fields":{"level":"none","logtype":"iis"},"beat":{"name":"WEB-TEST-01","hostname":"WEB-TEST-01","version":"6.4.2"},"host":{"name":"WEB-TEST-01"},"source":"C:\\\\inetpub\\\\logs\\\\LogFiles\\\\W3SVC2\\\\u_ex181011.log","offset":"697632"}
{% endhighlight %}

이걸 정리하면 아래와 같다. 중개자인 카프카는 로그라인만 전달하고, 파일비트가 로그라인을 json형식으로 만들어서 전달한다. `message` 필드가 실제 로그파일에서 갖고오는 데이터이고, 나머지 필드는 파일비트가 생성해준다. 특히 field에는 `filebeat.yml`파일에 개발자가 경로별로 세팅해준 데이터가 전달된다.

{% highlight json %}
{
    "@timestamp":"2018-10-25T07:53:39.249Z",
    "@metadata":{"beat":"filebeat","type":"doc","version":"6.4.2","topic":"kr-r2"},
    "message":"2018-10-11 05:40:30 172.0.0.1 GET /news/updates/detail bbmNo=89 443 - 10.1.30.99 Mozilla/5.0+(Windows+NT+6.1;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/69.0.3497.100+Safari/537.36 https://test.webpage.com/main 200 0 0 288",
    "prospector":{"type":"log"},
    "input":{"type":"log"},
    "fields":{"level":"none","logtype":"iis"},
    "beat":{"name":"WEB-TEST-01","hostname":"WEB-TEST-01","version":"6.4.2"},
    "host":{"name":"WEB-TEST-01"},
    "source":"C:\\\\inetpub\\\\logs\\\\LogFiles\\\\W3SVC2\\\\u_ex181011.log",
    "offset":"697632"
}
{% endhighlight %}

### filter 세팅

내가 설정을 만들면서 가장 고생을 한 부분은, 로그스태시가 파이프라인의 input 영역을 지나면서 **위의 데이터를 message영역으로 한번 더 감싼다는 것**이다. 그리고 파일비트에서 제이슨 형식으로 만들어서 전달한 데이터를 deserialize한다. 이렇게 하면 filter에 들어왔을 때 root 영역에 있는 field만 인식이 되고, nested field영역은 접근이 되지 않는다. 그렇기 때문에 먼저 json serialize를 해줘야했다. 이렇게 하면 감싸진 `message` 필드는 삭제되고, 안에있는 데이터가 `source`가 되어 필드가 구성된다.

{% highlight conf %}
filter {
  json { source => 'message' }
}
{% endhighlight %}

json serialize 필터를 지나면 다음과 같은 형태가 된다. 그리고 다음 필터에서 에러를 내면 디버거에서 다음과 같은 상태의 데이터를 출력해주며 에러 메세지를 전달한다. '=>' 표기를 통해 로그스태시가 nested field까지 제이슨 필드로 인식하는지 알 수 있다. 언뜻보면 없던 event 영역이 감싸진 것 처럼 보이는데, event는 로그스태시 데이터의 root 필드 이름이라고 보면된다. json필터의 `target`이 지정되어있지않으면 자동으로 루트 필더로 데이터가 전달된다.

{% highlight json %}
{
    "event"=> {
        "source"=>"C:\\\\inetpub\\\\logs\\\\LogFiles\\\\W3SVC2\\\\u_ex181011.log",
        "beat"=> { "name"=>"WEB-TEST-01", "hostname"=>"WEB-TEST-01", "version"=>"6.4.2"},
        "@timestamp"=>2018-10-25T07:53:39.249Z,
        "fields"=>{"logtype"=>"iis", "level"=>"none"},
        "offset"=>"697632",
        "@version"=>"1",
        "message"=>"2018-10-11 05:40:30 172.0.0.1 GET /news/updates/detail bbmNo=89 443 - 10.1.30.99 Mozilla/5.0+(Windows+NT+6.1;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/69.0.3497.100+Safari/537.36 https://test.webpage.com/main 200 0 0 288",
        "host"=> {"name"=>"WEB-TEST-01"},
        "prospector"=> {"type"=>"log"},
        "input"=> {"type"=>"log"}
    }
}
{% endhighlight %}

### nested field 조건문 세팅하기

nested 필드에 접근하려면 괄호를 통해서 접근하면 된다. 로그타입이 iis인경우와 nlog인경우로 나뉘기때문에, 해당 필드에 접근해서 타입을 확인하고 적절한 형식으로 grok 파싱을 해주고, 필요없는 필드는 `mutate` 영역에서 삭제했다.

{% highlight conf %}
filter {
  json { source => 'message' }
  if [fields][logtype] == "nlog" {
    grok {
      match => [ 'message', '%{DATESTAMP:date},%{DATA:url},%{NUMBER:status},%{IP:serverip},%{IP:clientip},{"PostData" : %{DATA:postdata},"GetData" : %{DATA:getdata}},%{DATA:stopwatch},{"message":"%{DATA:message}"},%{DATA:exception}$' ]
    }
  } else if [fields][logtype] == "iis" {
    grok {
      match => [ "message", "%{DATESTAMP:date} %{IP:server} %{WORD:method} %{DATA:uri_stem} %{DATA:uri_query} %{NUMBER:port} %{DATA:user_name} %{IP:client} %{DATA:user_agent} %{NUMBER:status} %{NUMBER:sub_status} %{NUMBER:win32_status} %{NUMBER:time_taken}" ]
    }
  }
  mutate {
    remove_field => ['beat', 'input', 'prospector']
  }
}
{% endhighlight %}

최종적으로 es에 전달되는 데이터 형태는 다음과 같다.

{% highlight json %}
{
    "message":"2018-10-11 05:40:30 172.0.0.1 GET /news/updates/detail bbmNo=89 443 - 10.1.30.99 Mozilla/5.0+(Windows+NT+6.1;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/69.0.3497.100+Safari/537.36 https://test.webpage.com/main 200 0 0 288",
    "fields":{"level":"none","logtype":"iis"},
    "uri_stem":"/news/updates/detail",
    "sub_status":"0",
    "time_taken":"288",
    "client":"10.1.30.99",
    "user_agent":"Mozilla/5.0+(Windows+NT+6.1;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/69.0.3497.100+Safari/537.36 https://test.webpage.com/main",
    "@timestamp":"2018-10-25T07:53:39.249Z",
    "date":"18-10-11 05:40:30",
    "server":"172.0.0.1",
    "win32_status":"0",
    "host":{"name":"WEB-TEST-01"},
    "user_name":"-",
    "offset":"697632",
    "status":"200",
    "uri_query":"bbmNo=89",
    "method":"GET",
    "port":"443",
    "source":"C:\\\\inetpub\\\\logs\\\\LogFiles\\\\W3SVC2\\\\u_ex181011.log",
    "@version":"1"
}
{% endhighlight %}