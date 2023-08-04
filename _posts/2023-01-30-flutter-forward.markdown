---
layout: post
title:  "[Flutter Forward] 레코드에 대해 알아보자"
date:   2023-01-30 19:39:00 +0
categories: Flutter Forward
published: true
tags: flutter
author: subincdev
---


최근 구글에서 Flutter Forward를 열어 플러터와 다트의 신기술에 관한 내용을 발표하였다.

(한국 시간으론 새벽에 열어서 기다리다 지쳐 잠들었지만..)

다음날 일어나서 발표한 내용들을 쭉 훑어보니 꽤 흥미로운 내용들이 많았는데,

그 중 눈에 띄는 다트 3.0에 반영될 예정인 '레코드'의 좋은 점에 대해 쉽고 간단히 이야기해보려한다.

(자바의 레코드, 파이썬의 튜플의 개념을 알고계신 분들은 유사한 개념이라고 생각하면 된다! )

하위 코드들은 모두 Flutter Forward에서 발표한 예시 코드입니다. 
아래 내용 중 틀린 내용이 있으면 피드백 언제든 환영입니다.
1. record를 활용해 함수 여러개 뱉기 가능! 


간단하게 사용자의 이름과 나이 정보를 받아 처리해주는 함수를 짠다고 가정해보자.
현재는 이름과 나이를 리스트에서 뽑은 다음 타입을 캐스팅해주어야 했다.

리스트는 value들의 타입들을 모두 같다고 가정하기 때문에

또 다시 타입을 명시해주어야 했다.


{% highlight dart %}
List<Object> userInfo(Map<String, dynamic> json) {
	return [json['name'] as String, json['age'] as int];
}


var json = {'name': 'Lily', 'age' : 13};

main() {
    var info = userInfo(json);
    var name = info[0] as String; // 스트링이라고!
    var age = info[1] as int; // 인트라고!
}
{% endhighlight %}

하지만 우리 이렇게 하기 싫어서 아래와 같이 클래스 만들어서 사용해줬더랬다.

물론 좀 더 깔끔하고 타입을 중복으로 명시해 줄 필요가 없어졌지만,


{% highlight dart %}
UserInfo userInfo(Map<String, dynamic> json) {
	return UserInfo(json['name'] as String, json['age'] as int);
}

var json = {'name': 'Lily', 'age' : 13};

main() {
    var info = userInfo(json);
    var name = info.name;
    var age = info.age;
}
{% endhighlight %}

위와 같은 코드를 짜기 위해선 아래와 같은 UserInfo라는 클래스를 만들고 또 만들고...


{% highlight dart %}
class UserInfo {
    final String name;
    final int age;
    
    UserInfo(this.name, this.age);
}
{% endhighlight %}

이 때 유용하게 사용할 수 있는 것이 record!


레코드는 다트 3.0에 반영될 built-in collection type(다수의 데이터를 처리할 수 있는 자료구조)으로
이제 더이상 리스트로 반환해서 타입 캐스팅을 해 줄 필요도 없고,

클래스를 만들어줄 필요도 없다.



레코드로 이제 타입이 다른 데이터들을 그냥 뱉고 $로 접근해 사용하면 된다!
$는 그저 $로 시작하는 getter이다.


{% highlight dart %}
(String, int) userInfo(Map<String, dynamic> json) {
	return (json['name'] as String, json['age'] as int);
}

var json = {'name': 'Lily', 'age' : 13};

main() {
    var info = userInfo(json);
    var name = info.$1;
    var age = info.$2;
}
{% endhighlight %}

함수에서 여러개를 리턴할 수 있는건 레코드의 그냥 일반적인 용례이고,

여기저기 자유롭게 사용할 수 있다.



특히 실무에서 json을 받아와 처리하는데 코드가 많이 간편해질 것 같다.

특히나 도입될 패턴이라는 친구와 합쳐진다면 증말..굿 (이건 다음에 이야기해보겠다)

2. 위젯 조건문 깔끔하게 만들 수 있음 
추가적으로 (왕중요) 위젯에서 조건문 처리를 할 때도 유용하다.

우리는 위젯에서 조건에 따라 어떤 처리를 해줄 때,

리스트 안에서 if문을 사용해주거나,

argument에 조건을 달아줄 땐 아래 코드와 같이

지옥의 삼항 연산자를 사용하여 구현했다.


{% highlight dart %}
ListTile(
   leading: const Icon(Icons.weekend),
   title: const Text("Welcome"),
   enabled: hasNextStep,
   subtitle: hasNextStep ? const Text("Tap to advance.") : null,
   onTap: hasNextStep ? () { advance(); } : null,
   trailing: hasNextStep ? null : const Icon(Icons.stop)
)
{% endhighlight %}

이제 지옥에서 벗어날 수 있게됐다.

이건 레코드로 깔쌈하게 해결 가능하기 때문이다.

왜냐면 레코드는 positional(위치 매개변수)와 named parameters가 모두 사용가능하기 때문에 (입틀막)!!!!!!



`var record = (1,2,x:3,y:4);`

if문 뒤에

named parameter로 만든 레코드를 spread 해준다면?..


{% highlight dart %}
ListTile(
   leading: const Icon(Icons.weekend),
   title: const Text("Welcome"),
   enabled: hasNextStep,
   if(hasNextStep) ...( // spread record
       subtitle: const Text("Tap to advance."),
       onTap: () { advance(); },
   ) else ...(// spread record
       trailing: const Icon(Icons.stop),
   )
)
{% endhighlight %}
그저 극락..

하지만 argument spread는 dart3에는 안들어간다고 한다. (예?)

빨리 내주세요..일해라 구글..



참고 자료

- [Bringing pattern matching to Dart]
- [Flutter-Dart-Flutter-forward-2023]
- [https://www.itworld.co.kr/news/274993]

[Bringing pattern matching to Dart]: https://www.youtube.com/watch?v=KhYTFglbF2k
[Flutter-Dart-Flutter-forward-2023]: https://velog.io/@long9725/Flutter-Dart-Flutter-forward-2023
[https://www.itworld.co.kr/news/274993]: https://www.itworld.co.kr/news/274993

<!-- You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/ -->
