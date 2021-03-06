---
title: PHP와 UTF-8
isChild: true
anchor: php_and_utf8
---

## PHP와 UTF-8 {#php_and_utf8_title}

_이 섹션은 [Alex Cabal](https://alexcabal.com/)이 작성한
[PHP Best Practices](https://phpbestpractices.org/#utf-8)에 포함되어 있는 내용입니다.
이제 이곳에서도 볼 수 있게 되었습니다._

### 한 줄로 끝나는 팁은 없습니다. 주의깊고 일관성있게 작업해야 합니다.

PHP는 현재 유니코드 지원을 언어 내부적으로 해주고 있지 않습니다. UTF-8 문자열을 문제없이 
처리할 수 있는 방법은 있지만, HTML 코드, SQL 쿼리, PHP 코드 등 웹 어플리케이션의 모든 
레벨을 다뤄야 하는 수준의 이야기입니다. 여기서는 활용할 수 있는 간략한 설명을 제공하는데 초점을 
두겠습니다.

### PHP 코드 수준에서의 UTF-8

문자열을 변수에 할당하거나 두 문자열을 이어붙이는 등의 기본적인 문자열 연산은 UTF-8 문자열이라고
해도 특별할 것이 없습니다. 하지만 `strpos()`나 `strlen()`과 같은 대부분의 문자열
함수들은 신경을 써야 합니다. 이런 함수들은 `mb_strpos()`나 `mb_strlen()` 같이
원래 이름 앞에 `mb_`가 붙은 함수들이 별도로 존재합니다. 그런 함수들을 멀티바이트 문자열 함수라고
부릅니다. 멀티바이트 문자열 함수들은 유니코드 문자열을 다루기 위해서 특별히 제공되는 함수들입니다.

유니코드 문자열을 다룰 때에는 항상 `mb_`로 시작하는 함수들을 사용해야 합니다. 예를 들어
`substr()` 함수를 UTF-8 문자열에 대해서 사용해보면, 결과물에는 이상하게 깨진 글자가
나온다는 사실을 알게 될 좋은 기회가 될 것입니다. UTF-8 문자열을 다룰 때에는 
`mb_substr()` 함수를 사용해야 합니다.

유니코드 문자열을 다룰 때에는 항상 `mb_`로 시작하는 함수를 사용해야 한다는 걸 기억하는 게 
어려운 일입니다. 한 군데라도 까먹고 일반 문자열 함수를 사용하면 깨진 유니코드 문자열을 보게 될 수가
있습니다.

모든 문자열 함수가 그에 해당되는 멀티바이트 버전의 `mb_` 로 시작되는 함수를 가지고 있는 것은 
아닙니다. 여러분이 사용하려고 했던 문자열 함수 그런 경우라면, 운이 없었다고 할 수 밖에요.

추가로, 모든 PHP 스크립트 파일(또는 모든 PHP 스크립트에 include 되는 공용 스크립트)의 
가장 윗부분에 `mb_internal_encoding()` 함수를 사용해서 문자열 인코딩을 지정하고, 그 바로 
다음에 `mb_http_output()` 함수로 브라우저에 출력될 문자열 인코딩도 지정해야 합니다. 이런 
식으로 모든 스크립트에 문자열의 인코딩을 명시적으로 지정하는 것이, 나중에 고생할 일을 아주
많이 줄여줍니다.

마지막으로, 많은 문자열 함수들이 함수 파라미터의 문자열 인코딩을 지정할 수 있는 옵셔널 파라미터를
제공한다는 사실을 이야기해야겠습니다. 그런 경우마다 항상 UTF-8 문자열 인코딩을 명시적으로
지정해야 합니다. 예를 들어, `htmlentities()` 함수의 경우가 그렇습니다. UTF-8 문자열을 전달하는 
경우 항상 UTF-8로 지정하십시오.

PHP 5.4.0 부터는  `htmlentities()` 함수와 `htmlspecialchars()` 함수의 경우에 
UTF-8이 기본 인코딩으로 변경되었음을 알아두십시오.


### 데이터베이스 수준에서의 UTF-8

MySQL을 사용하는 PHP 스크립트가 있다면, 앞에서 이야기한 내용을 충실히 따르더라도 
데이터베이스에 저장되는 문자열은 UTF-8이 아닌 다른 인코딩으로 저장될 가능성이 있습니다. 

PHP에서 MySQL로 전달되는 문자열이 UTF-8로 확실히 전달되게 하려면, 사용하는 데이터베이스와
테이블이 모두 `utf8mb4` 캐릭터 셋과 콜레이션(collation)을 사용하게 해야합니다.
그리고 PDO 연결문자열에도 `utf8mb4` 캐릭터 셋을 사용한다고 명시해야 합니다. 아래의 
예제를 보세요. _정말 중요합니다_.

UTF-8 문자열을 제대로 사용하려면 반드시 `utf8mb4` 캐릭터 셋을 사용해야 합니다.
`utf8` 캐릭터 셋이 아닙니다! (놀라워라...) 그 이유는 '더 읽어보기'를 참고하세요.

### 브라우저 수준에서의 UTF-8

PHP 스크립트가 UTF-8 문자열을 브라우저에 제대로 전송하게 하려면 `mb_http_output()`
함수를 사용하세요. HTML의 `<head>` 태그에는 [charset `<meta>` 태그](http://htmlpurifier.org/docs/enduser-utf8.html)를 넣습니다.

{% highlight php %}
<?php
// 이 스크립트 파일의 끝까지 UTF-8 문자열을 사용할 것임을 PHP에게 알려줍니다.
mb_internal_encoding('UTF-8');

// UTF-8 문자열을 브라우저에 전송하려고 한다고 PHP에게 알려줍니다.
mb_http_output('UTF-8');
 
// UTF-8 테스트용 문자열
$string = 'Êl síla erin lû e-govaned vîn.';
 
// 멀티바이트 문자열 함수를 사용해서 문자열 자르기를 합니다.
$string = mb_substr($string, 0, 15);
 
// 자르기 해서 새로 만들어진 문자열을 데이터베이스에 저장하기 위해서 일단 접속을 합니다.
// 더 많은 정보를 얻으려면 이 문서의 PDO 관련 내용을 참고하세요.
// `charset=utf8mb4` 로 지정하고 있다는 점을 유의하세요!
$link = new \PDO(   
                    'mysql:host=your-hostname;dbname=your-db;charset=utf8mb4',
                    'your-username',
                    'your-password',
                    array(
                        \PDO::ATTR_ERRMODE => \PDO::ERRMODE_EXCEPTION,
                        \PDO::ATTR_PERSISTENT => false
                    )
                );
 
// 데이터베이스에 문자열을 저장합니다.
// DB와 테이블을 utf8mb4 캐릭터 셋과 콜레이션을 사용하도록 만들어 둔 상태인지 다시 한 번 확인하세요.
$handle = $link->prepare('insert into ElvishSentences (Id, Body) values (?, ?)');
$handle->bindValue(1, 1, PDO::PARAM_INT);
$handle->bindValue(2, $string);
$handle->execute();
 
// 제대로 저장되었는지 검증하기 위해서 방금 저장한 문자열을 다시 읽어옵니다.
$handle = $link->prepare('select * from ElvishSentences where Id = ?');
$handle->bindValue(1, 1, PDO::PARAM_INT);
$handle->execute();
 
// HTML에 출력하기 위해서 변수에 결과값을 저장해둡니다.
$result = $handle->fetchAll(\PDO::FETCH_OBJ);
?><!doctype html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>UTF-8 test page</title>
    </head>
    <body>
        <?php
        foreach($result as $row){
            print($row->Body);  // 자르기 한 문자열을 정확하게 표시해주기를 기대해 봅니다.
        }
        ?>
    </body>
</html>
{% endhighlight %}

### 더 읽어보기

* [PHP Manual: String Operations](http://php.net/manual/en/language.operators.string.php)
* [PHP Manual: String Functions](http://php.net/manual/en/ref.strings.php)
    * [`strpos()`](http://php.net/manual/en/function.strpos.php)
    * [`strlen()`](http://php.net/manual/en/function.strlen.php)
    * [`substr()`](http://php.net/manual/en/function.substr.php)
* [PHP Manual: Multibyte String Functions](http://php.net/manual/en/ref.mbstring.php)
    * [`mb_strpos()`](http://php.net/manual/en/function.mb-strpos.php)
    * [`mb_strlen()`](http://php.net/manual/en/function.mb-strlen.php)
    * [`mb_substr()`](http://php.net/manual/en/function.mb-substr.php)
    * [`mb_internal_encoding()`](http://php.net/manual/en/function.mb-internal-encoding.php)
    * [`mb_http_output()`](http://php.net/manual/en/function.mb-http-output.php)
    * [`htmlentities()`](http://php.net/manual/en/function.htmlentities.php)
    * [`htmlspecialchars()`](http://www.php.net/manual/en/function.htmlspecialchars.php)
* [PHP UTF-8 Cheatsheet](http://blog.loftdigital.com/blog/php-utf-8-cheatsheet)
* [Stack Overflow: What factors make PHP Unicode-incompatible?](http://stackoverflow.com/questions/571694/what-factors-make-php-unicode-incompatible)
* [Stack Overflow: Best practices in PHP and MySQL with international strings](http://stackoverflow.com/questions/140728/best-practices-in-php-and-mysql-with-international-strings)
* [How to support full Unicode in MySQL databases](http://mathiasbynens.be/notes/mysql-utf8mb4)
* [Brining Unicode to PHP with Portable UTF-8](http://www.sitepoint.com/bringing-unicode-to-php-with-portable-utf8/)
