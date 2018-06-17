---
title:  "iOS App In House 배포하는 두 가지 방법"
date:   2018-06-17 01:11:00
categories: text
---

# Methodologies

iOS 앱을 private하게 배포하는 방법은 흔히 두가지 방법이 있다. 전통적인 방식으로 URL 다운로드링크 배포하거나, Fastlane beta를 통해 배포하는 법.

구글에서 제공하는 Fastlane은 빌드부터 아카이빙, ipa추출, 업데이트까지 원큐로 해주고 거기에 인증서를 git으로 관리하는 'match'까지 제공한다. 그런데,
1. 배포 대상이 수시로 변동될 경우 계속해서 배포 이메일을 추가해야하는 문제가 있으며
2. 보통 apple 개발자 계정을 공유하는 회사에서는 누가 어떤 인증서를 쓰는지도 모르는 상태이기 마련인데, 처음 repo 생성시 인증서 삭제를 권유한다. 인증서가 이미 배포된 서비스는 중지되지 않는다고 안심을 시키지만.
3. 나의 경우 개발자계정관리를 다른팀에서 하고 있는 상태였기 때문에, 잘못된 과정을 밟게되면 또 다른 인증서 지옥을 겪을 가능성이 농후해서 manual하게 직접 추가해주는게 수월할 것이라는 판단을 했다.

지금부터의 가이드는 아카이빙을 통한 ipa파일 생성이 완료된 상태에서 할 일이다.


# How-To

## 배포를 위한 설정파일 만들기

1. 프로젝트에 ipa파일 추가
2. .plist, .ipa MIME type 추가하기
	- IIS의 경우 IIS관리자에 서비스 인스턴스>MIME 타입변경을 통해 할 수 있다
	- IIS의 경우 .ipa application/octet-stream 으로, .plist text/xml으로 하면된다.
	- Mac에서 배포하는 경우에도 추가해줘야하며, 같은 확장자라도 타입이 다를수도 있다.
3. app.plist 파일을 만든다
	- archiving할때 설정 잘 해주면 xcode에서 생성해주기도 하지만 굳이 빌드하지 않고 직접만들어도 된다.
	- 어차피 html파일에 href로 링크 붙일거라 파일명은 상관없고 plist확장자만 갖고있으면 된다.
4. 아래의 코드는 plist파일 최소의 스펙이다.
	- 참고로 다운로드중이리 때 백색 이미지 말고 별개의 이미지를 설정할수도 있다. 이건 in house 배포 검색하면 나올것이다.
	- 배포와 상관없는 key값이 더 추가되면 문제가 생긴다는 글을 보기도 했다. 애플은 버전별 변동이 잦아서 상이하므로 아무튼 주의.
	- 특히 Metadata가 빠지거나 잘못되어있으면 별다른 피드백 없이 그냥 다운로드가 되지 않으므로 반드시 xcode 프로젝트의 info.plist와 일치시켜 넣도록 하자.
    {% highlight c %}
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
        <plist version="1.0">
        <dict>
            <key>items</key>
            <array>
                <dict>
                    <key>assets</key>
                    <array>
                        <dict>
                            <key>kind</key>
                            <string>software-package</string>
                            <key>url</key>
                            <string>https://monitor.webzen.com/build/monitor.ipa/monitor.ipa</string>
                        </dict>
                    </array>
                    <key>metadata</key>
                    <dict>
                        <key>bundle-identifier</key>
                        <string>com.webzen.app.haribo.monitor</string>
                        <key>bundle-version</key>
                        <string>1.0</string>
                        <key>kind</key>
                        <string>software</string>
                        <key>title</key>
                        <string>Monitor</string>
                    </dict>
                </dict>
            </array>
        </dict>
    </plist>
    {% endhighlight %}

## HTML 링크 붙이기
1. 배포를 위한 설정파일이 준비되면, Url은 ipa 파일이 있는 경로를 추가해주면 된다.
	- "https://localhost/directorypath/ipa"와 같은 형식이면 된다.
	- 애플 정책상 ipa파일과 plist파일 모두 https를 반드시 사용해야한다.
	- 없다면 그냥 드롭박스나 구글드라이브로 배포해야한다.
2. Html파일에 아래와같이 해당 링크를 추가하면 끝~
    - FYI: 다운로드가 제대로 안되는 상태에서 아래 링크를 아이폰 내 사파리앱에서 들어가면 itunes랑 연결하겠냐는 질문이 나오는 것으로 보아 아이튠즈랑 통신하는 과정이 있는 듯 하다.

{% highlight c %}
<a href="itms-services://?action=download-manifest&url=https://monitor.webzen.com/build/monitor.ipa/monitor.plist">
    <button type="button" class="btn btn-default" id="iosDownload">
</a>
{% endhighlight %}