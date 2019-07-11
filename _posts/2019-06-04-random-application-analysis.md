---
layout: post
title:  "랜덤한 앱 분석시간"
date:   2019-06-04 15:53:39 +0900
categories: life
tags: android reversing
image: /assets/2019-06-04-random-application-analysis/1.png
---

<style>
img + em {
    display: block;
    text-align: center;
}
</style>

## 개요

플레이 스토어에는 별의별 앱이 존재한다. 게임부터 시작해서 뭐 메모장 앱 전광판 앱 뭐 다양하게 있는데. 솔직히 존재 이유를 알수 없거나, 이런게 올라와도 되는건가 싶은 앱도 많이 봤다. 그래서 한번 앱 다운받아서 분석해보려고 한다. 혹시라도 분석하다가 악성코드 같은거라도 발견하면 제보할수도 있으니 히히

미리 써놓는데 필자가 겁나 초짜여서 잘하는 사람이 보기엔 분석은 개뿔 그냥 겉만 훝고 지나가는 수준일수 있으니 글은 그냥 재미로 읽어주기 바란다.. 바랍니다

## 물론 대상은 랜덤하다

![1](/assets/2019-06-04-random-application-analysis/1.png){: .center-image }
*세상은 넓고 취향은 다양하다*

그래서 뭘 한번 분석해볼까 물색하다가 [`헤어 클리퍼 - 장난이다`](https://play.google.com/store/apps/details?id=com.cuneytayyildiz.sackesmemakinesi) 라는 앱을 발견했다. 보니까 면도기 진동이랑 소리를 따라한 앱 같은데 다운로드 수가 10,000,000회가 넘는다고 한다, 세상에. 물론 출시 날짜가 2013년으로 모바일 어플리케이션 시장이 슬슬 가동하기 시작한 시기이니 사람들이 뭐 궁금해서 많이 깔아본걸 수도 있을것 같다. 그래도 이 잉여 앱을 천만명이 깔았다는 건 좀 신기하다...?

## APK 추출

```
$ adb devices
List of devices attached
52003aad460a8585        device
$ adb shell "pm list packages -f | grep 'com.cuneytayyildiz.sackesmemakinesi'"
package:/data/app/com.cuneytayyildiz.sackesmemakinesi-WGf935H7FTl35ZyZA5PBcw==/base.apk=com.cuneytayyildiz.sackesmemakinesi
$ adb pull /data/app/com.cuneytayyildiz.sackesmemakinesi-WGf935H7FTl35ZyZA5PBcw==/base.apk .
/data/app/com.cuneytayyildiz.sackesmemakinesi-WGf935H7...pk: 1 file pulled. 22.8 MB/s (5946896 bytes in 0.248s)
```

뭐 웹에도 APK 파일을 다운로드 받을수 있는 여러 웹페이지가 존재한다. 나는 그냥 확실하게 스마트폰에서 다운받고 ADB로 추출했다. grep으로 가져올때 사용한 패키지 이름 같은건 플레이 스토어 링크에서 찾을 수 있다. PC로 아무 이름 없이 pull 받으면 아마 `base.apk`란 이름으로 추출될 것이다.

> https://play.google.com/store/apps/details?id=com.cuneytayyildiz.sackesmemakinesi

## 분석...? 

분석할 때 [`jadx`](https://github.com/skylot/jadx)란 툴을 사용했다. `luyten`, `jd-gui`, `apktool` 같은 다른 도구 또한 있으나 얘는 apk 파일 하나만 던져줘도 다 한큐에 해줘서 편해서 쓴다..ㅎㅎ 

![2](/assets/2019-06-04-random-application-analysis/2.png){: .center-image }
*kotlin 패키지가 포함된걸로 보아하니 아마...*

여러 패키지가 포함되어 있는데 우리가 볼건 당연히 `com.cunetayyildiz.sackesmemakinesi` 패키지 안 내용이다. 프로가드가 적용된건지 클래스가 다 a, b, c로 되어 있다. 음...

특이한건 a, b, c 클래스가 문자열 2개를 표현하는 클래스란 것이다.

```java
package com.cuneytayyildiz.sackesmemakinesi;

/* compiled from: BuildConfig */
class b {
    int a;

    b() {
    }

    public String toString() {
        r0 = new byte[20];
        this.a = -300800051;
        r0[0] = (byte) (this.a >>> 21);
        this.a = 651195876;
        r0[1] = (byte) (this.a >>> 8);
        this.a = -1990036119;
        r0[2] = (byte) (this.a >>> 16);
        this.a = 945433598;
        r0[3] = (byte) (this.a >>> 17);
        this.a = -997359303;
        r0[4] = (byte) (this.a >>> 14);
        this.a = -2046429074;
        r0[5] = (byte) (this.a >>> 1);
        this.a = 587916865;
        r0[6] = (byte) (this.a >>> 5);
        this.a = 139889898;
        r0[7] = (byte) (this.a >>> 6);
        this.a = 1617250420;
        r0[8] = (byte) (this.a >>> 17);
        this.a = 474146563;
        r0[9] = (byte) (this.a >>> 23);
        this.a = 1502919226;
        r0[10] = (byte) (this.a >>> 19);
        this.a = -1271708966;
        r0[11] = (byte) (this.a >>> 12);
        this.a = 830758066;
        r0[12] = (byte) (this.a >>> 19);
        this.a = -744084521;
        r0[13] = (byte) (this.a >>> 13);
        this.a = -1189636007;
        r0[14] = (byte) (this.a >>> 7);
        this.a = -768324197;
        r0[15] = (byte) (this.a >>> 16);
        this.a = 1315775148;
        r0[16] = (byte) (this.a >>> 22);
        this.a = 1383228255;
        r0[17] = (byte) (this.a >>> 9);
        this.a = -1009457364;
        r0[18] = (byte) (this.a >>> 5);
        this.a = 1626026589;
        r0[19] = (byte) (this.a >>> 8);
        return new String(r0);
    }
}
```

이런 형식으로 문자열이 안박혀있고 실행 할때마다 저렇게 연산되도록 만들어져 있어서 코드 짜서 풀어봤는데 `ca-app-pub-6723282401049192~7306289350`, `pub-6723282401049192` 두 문자열이 나왔다. 생성된 문자열 패턴을 찾아봤는데 안드로이드 광고 ID인것으로 추정된다. 아마 쌩으로 박아놓긴 좀 그래서 컴파일러가 빌드할때 알아서 최소한의 난독화를 시킨건가 추측해 본다. 신기

궁금해서 `io.fabric.sdk.android`는 무슨 라이브러리인가 하고 쳐봤는데 Fabric 자체가 앱 유지보수 할때 쓰는 플랫폼인걸로 확인했다. 저기 위에 있는 `crashlytics.android` 패키지도 firebase랑 연결되는 것 같다. [여길](https://raon-studio.tistory.com/3) 보니 앱 터질때 상태를 Fabric으로 보내서 어디가 터졌는지 개발자에게 알려줘 좀 더 쉽게 개발할 수 있게 도와주는 역할을 하는 것 같다. 와우

```java
protected void onCreate(Bundle bundle) {
    super.onCreate(bundle);
    setContentView((int) R.layout.activity_main);
    q();
    l();
    o();
    p();
    f.a(new f(this), false, 1, null);
}
```

MainActivity onCreate를 보면 q, l, o, p, f.a를 호출한다.

```java
private final void q() {
    Object findViewById = findViewById(R.id.container);
    g.a(findViewById, "findViewById(R.id.container)");
    this.p = (LinearLayout) findViewById;
    findViewById = findViewById(R.id.adView);
    g.a(findViewById, "findViewById(R.id.adView)");
    this.q = (AdView) findViewById;
    findViewById = findViewById(R.id.imgClipper);
    g.a(findViewById, "findViewById(R.id.imgClipper)");
    this.r = (ImageView) findViewById;
    findViewById = findViewById(R.id.imgSettings);
    g.a(findViewById, "findViewById(R.id.imgSettings)");
    this.s = (ImageView) findViewById;
    ImageView imageView = this.r;
    if (imageView != null) {
        imageView.setOnClickListener(new e(this));
        imageView = this.s;
        if (imageView != null) {
            imageView.setOnClickListener(new f(this));
            return;
        } else {
            g.b("imgSettings");
            throw null;
        }
    }
    g.b("imgClipper");
    throw null;
```

q는 그냥 UI 그리는 함수인것 같다. onCreate에 한번에 넣기는 좀 그래서 따로 컴파일러가 분리시킨건가

```java
private final void l() {
    Object obj = c.b;
    g.a(obj, "BuildConfig.ADMOB_PUBLISHER_ID");
    Object string = getString(R.string.privacy_policy_url);
    g.a(string, "getString(R.string.privacy_policy_url)");
    new a(this, obj, string, this.t, false, true).a(null);
}
```

보면 볼수록 g가 많이 보이는데 저건 `kotlin.c.b.g` 패키지의 g 클래스이다. 시도때도 없이 나오고 호출할 때 문자열을 매개변수로 넣는것으로 보아하니 안드로이드의 `Log` 함수같은게 아닐까 조심스럽게 추측해본다. 아니면 아까 나온 Fabric으로 로그 데이터를 뿌려주는걸까. 뭐 나중에 logcat으로 확인하면 되겠지

```java
private final void o() {
        AdView adView = this.q;
        String str = "adView";
        if (adView != null) {
            adView.loadAd(n());
            adView = this.q;
            if (adView != null) {
                adView.setAdListener(new b(this));
                this.u = new InterstitialAd(this);
                InterstitialAd interstitialAd = this.u;
                str = "interstitialAd";
                if (interstitialAd != null) {
                    interstitialAd.setAdUnitId(getString(R.string.admob_interstitial_ad_id));
                    interstitialAd = this.u;
                    if (interstitialAd != null) {
                        interstitialAd.loadAd(n());
                        interstitialAd = this.u;
                        if (interstitialAd != null) {
                            interstitialAd.setAdListener(new c(this));
                            return;
                        } else {
                            g.b(str);
                            throw null;
                        }
                    }
                    g.b(str);
                    throw null;
                }
                g.b(str);
                throw null;
            }
            g.b(str);
            throw null;
        }
        g.b(str);
        throw null;
    }
```

o는 딱봐도 광고 가져오는 메소드인것 같다. 참고로 `n()`은 광고 ID 문자열 가져오는 함수이다.

```java
private final void p() {
    AnimationDrawable m = m();
    Object a = B.a((FragmentActivity) this).a(g.class);
    g.a(a, "ViewModelProviders.of(th…ainViewModel::class.java)");
    this.v = (g) a;
    g gVar = this.v;
    if (gVar != null) {
        gVar.c().a(this, new d(this, m));
    } else {
        g.b("mainViewModel");
        throw null;
    }
}
```

애니메이션이 나오는데 이건 버튼 눌럿을때 면도기가 동작하는 것처럼 표현할때 이미지 움직이는걸 구현한 것 같다. 계속 보고있는데 자꾸 g나오고 뜬금없이 m나오고 어휴 복잡해

## 참고

- https://brompwnie.github.io/reversing/2018/02/12/Kotlin-and-Java-How-Hackers-See-Your-Code.html