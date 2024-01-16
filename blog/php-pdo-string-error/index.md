---
layout: post
id: 1
title:  "PHP PDO json_encode strings as numbers 반환 오류"
subtitle: ""
description: "PHP PDO json_encode strings as numbers 반환 오류에 대한 정보"
type: "PHP"
created_at: "2024-01-14"
updated_at: "2024-01-14"
blog: true
text: true
author: "Neptune"
post-header: true
header-img: ""
order: 1
tags: ['php']
comments: true
---

평소 때와 다름 없는 업무 시간, API 작업을 하는 도중 프론트에서 긴급한 이슈사항이 넘어왔다.

```php
{
	"title": "1111111111",
        "content": "222222222222222222222"
}
```

등록 API에서 제목과 내용을 프론트에서 유닛 테스트를 진행하던 도중 숫자로만 구성된 문자열을 보냈는데

오류가 발생했다는 것이다. 보통 게시글에서 제목이라 하면 `🚀 공지사항 테스트` 문자가 들어간 문자열로 들어갈텐데

무슨 큰 일이 있겠어라는 생각으로 DB에서 결과값을 확인해보았는데 경악을 금치 못했다.

```php
{
	"title": 12341265+7ef,
	"content": 123452+2cf
}
```

상세 API에서 호출된 결과값은 문자열이 아닌 숫자로 반환해 주는게 확인되었다.

그러다 보니 프론트에서는 숫자로 된 문자열이지만 `string`으로 보냈는데

결과값은 왜 `number`이냐라는 결론에 도달했다.
<br/>
어디서부터 문제인건지 급한 마음에 디버깅을 시작했지만 DB에 데이터가 들어가기 직전에도 올바르게

문자열로 인식이 되고 있었다.



그러던 중, 문득 DB에 데이터가 들어가기 전에도 잘 인식하고 있는데.....

혹시 PDO에서 문제를 발생시키는 건 아닐까?라는 궁금증이 생겼다.

```php
<?php

namespace model;

use PDO;

class Model 
{
    private $connect;

    public function __construct() 
    {
        try {
            $connect = new PDO('DB_CONNECTION:host=DB_HOST;dbname=DB_DATABASE', 'DB_USERNAME', 'DB_PASSWORD',
            [PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8', PDO::MYSQL_ATTR_FOUND_ROWS => true]);    

            $connect->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $connect->exec('set names utf8mb4');
        } catch (\PDOException $ex) {
            echo "PDO CONNECTION Error {$ex->getMessage()}"; exit;
        }

        $this->connect = $connect;
    }   
}
```

PDO 인스턴스를 생성하여 setAttribute로 옵션을 설정할 수 있는데 공식 문서를 확인해보자.

https://www.php.net/manual/en/pdo.setattribute.php

POD::ATTR_STRINGIFY_FETCHES 숫자 값을 가져올때 string 형태로 변환 / 옵션으로 true false를 줘야함
PDO 공식 문서에서 확인을 해보면 숫자 값을 가져올 때 string 형태로 변환하는 옵션에 대한 값이 있다.

```php
$connect->setAttribute(PDO::ATTR_EMULATE_PREPARES, false); 
$connect->setAttribute(PDO::ATTR_STRINGIFY_FETCHES, false);
```

기존 PDO 인스턴스를 연결하는 부분에서 해당 두 구문을 추가하여 다시 상세 API를 호출해보았다.

```php
{
	"title": "1111111111",
        "content": "222222222222222222222"
}
```

결과는 다행히 숫자로만 이루어진 문자열임에도 불구하고 `number`가 아닌 문자열 그대로

출력되는 부분이 확인이 되었다.

당연히 제목이나 내용이면 숫자가 아닌 문자도 들어갈거라는 내 착오에서 발생된 오류라

이번 계기로 많은 깨달음을 느낀 것 같았다.