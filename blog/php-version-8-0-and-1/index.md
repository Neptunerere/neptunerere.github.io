---
layout: post
id: 1
title:  "PHP 8.0 | 8.1 변경점 알아보기"
subtitle: ""
description: "PHP 8.0 | 8.1 변경점에 대한 정보"
type: "PHP"
created_at: "2024-01-10"
updated_at: "2024-01-10"
blog: true
text: true
author: "Neptune"
post-header: true
header-img: "img/php-version-8-0-and-1.png"
order: 1
tags: ['php']
comments: true
---

실무에서 많이 사용하는 위주로 콕 집어서 정리해보았다.git 

# 어트리뷰트 (Attribute)

코드를 선언할 때 메타 정보를 추가할 수 있도록 지원을 한다.

이 때 어트리뷰트의 정보는 `리플렉션 API` 를 통해서 접근할 수 있다.

```php
#[Attribute]
class Bar {}

class Foo 
{
    #[Bar]
    public function barTest()
    {
        // ...
    }
}
```

# 열거형 (Enum)

PHP에는 그동안 열거형이 없어서 클래스와 상수를 통해 사용을 해왔는데 드디어

용도에 맞게 사용할 수 있는 타입이 생겼다.

## 기초

기본적으로 열거형은 다음과 같이 선언이 가능하다.

```php
enum HttpMethod
{
    case GET;
    case POST;
    case PULL;
    case PATCH;
    case DELETE;
}
```

열거형 타입으로 `HttpMethod` 를 선언하고 http method로 사용이 가능한

5가지 허용된 값으로 `HttpMethod::GET`, `HttpMethod::POST` ... 그 외

직접 사용하거나 변수에 할당해서 사용하는 것이 가능하다.

## 지원 열거형

위에서 본 열거형과 달리 직렬화 해야 하는 경우에 열거형에 기본값을 넣어서 더 유용하게 사용할 수 있다.

```php
enum ApiVersion: string
{
    case VERSION_1 = 'v1';
    case VERSION_2 = 'v2';
    case VERSION_3 = 'v3';
}
```

이 지원 열거형은 `int`나 `string`과 함께 사용할 수 있다. 

다만 `int|string`과 같이 동시에 둘을 지원할 수는 없다.

# `?->` null 안전 연산자

PHP 8.0에 들어서면서 `널 병합 연산자 '??'`를 많이 사용하였는데 거기에 더해

null에 안전하게 프로퍼티와 메소드에 접근할 수 있도록 연산자가 추가되었다.

Javascript 기준으로 치자면 `옵셔널 체이닝 'Optinal Chaining'`과 유사하다.

```php
$userStreet = $user?->getAddress(8)?->street;

// 풀어서 보기
if (is_null($user)) {
    $userStreet = null;
} else {
    $user = $user->getAddress(8);
    
    if (is_null($user)) {
        $userStreet = null;
    } else {
        $userStreet = $user->street;
    }
}
```

# `throw` 표현식

드디어 `throw`를 표현식에도 사용할 수 있게 됐다.

```php
$test = fn() => throw new Exception('Exception in test');
$userData = $session->user ?? throw new Exception('user Not Found');
```

# `readonly` 프로퍼티 추가

개체 초기화 시, 작성할 수 있는 `readonly` 프로퍼티가 추가 되었다.

타입이 지정된 프로퍼티에서만 사용이 가능하다.

단, 정적 클래스에서 지원하지 않고 어떤 값이든 다시 배정할 수가 없다.

```php
class Foo 
{
    public readonly string $bar;
    
    public function __construct(string $bar)
    {
        $this->bar = $bar;
    }
}


$fooBar = new Foo('foobar');
var_dump($fooBar->bar); // string(6) "foobar"
```

