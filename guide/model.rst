.. _mvc:

5장. Model
******************

이 장에서는 엔드포인트가 참조하는 데이터인 모델에 대해 설명한다.
일반적으로 RESTful API로 제공되는 JSON이나, HTML처럼 사람이 읽을 수 있는 포맷을 의미한다. 

다음과 같이 엔드포인트가 설정되어 있다고 가정한다. ::

   # vhosts.xml - <Vhosts><Vhost><M2><Endpoints>

   <Endpoint Alias="inven" Post="OFF" Get="ON">
      <Control ViewParam="view" ModelParam="model">/search</Control>
      <Model>https://foo.com/#model</Model>
      <View>https://bar.com/#view</View>
   </Endpoint>


M2-JSON 구조
====================================

엔드포인트를 ``/search`` 로 설정했으므로 클라이언트는 다음과 같이 호출한다. ::

   http://www.example.com/search?model=100&view=list


M2는 ``<Model>`` 설정에 따라 다음 주소를 호출한다. ::

   https://foo.com/100

::

   {
      "firstName": "John",
      "lastName": "Smith",
      "age": 25,
      "address": {
         "streetAddress": "21 2nd Street",
         "city": "New York",
         "state": "NY",
      "postalCode": "10021"
         },
      "phoneNumber": [
         { "type": "home", "number": "212 555-1234" },
         { "type": "fax", "number": "646 555-4567" }
      ]
   }


응답을 포함한 모은 컨텍스트 정보의 집합을 M2-JSON이라고 부른다. M2-JSON은 크게 ``model`` 과 ``req`` 나뉘며 이 경우 다음과 같이 구성된다. ::

   {
      "model": {
         {
            "firstName": "John",
            "lastName": "Smith",
            "age": 25,
            "address": {
               "streetAddress": "21 2nd Street",
               "city": "New York",
               "state": "NY",
               "postalCode": "10021"
            },
            "phoneNumber": [
               { "type": "home", "number": "212 555-1234" },
               { "type": "fax", "number": "646 555-4567" }
            ]
         }
      },
      "req": { ... }
   }

.. note::

   M2-JSON의 ``"model"`` 은 고정이 아니다. 만약 ``<Control ModelParam="myData">`` 라고 설정했다면 다음과 같이 구성된다. ::

      {
         "myData": { ... },         
         "req": { ... }
      }



GET Method
------------------------------------



Model 배열
====================================

단일모델 사용시 접두어는 ``model.`` 이며 모델배열의 경우 ``model[0].`` 처럼 배열 인덱스를 사용한다.
하나의 뷰에 동일한 형태의 여러 모델이 필요한 경우 배열을 사용한다. ::

   /users/platinum?mym=[apple,banana,cherry]&view=catalog

위와 같이 ``#model`` 에 대응하는 값을 ``[ ... ]`` 형식으로 입력한다. M2는 ``<Model>`` 에 설정된 주소에 각각의 값을 바인딩하여 결과를 배열로 취합한다. 이렇게 생성된 배열의 키는 쿼리스트링 키로 맵핑된다. ::

   {
      "mym" : [
         { "name": "apple", ... },
         { "name": "banana", ... },
         { "name": "cherry", ... }
      ]
   }

위와 같은 모델 배열을 생성하기 위해 아래의 API 호출이 발생한다. ::

   https://alice.com/bob/apple.json
   https://alice.com/bob/banana.json
   https://alice.com/bob/cherry.json

모든 API 호출이 성공하면 다행이겠지만 일부만 성공할 가능성이 있다. 이런 일부 모델의 실패 상황을 ``Sparse`` 속성으로 대처할 수 있다. ::

   # vhosts.xml - <Vhosts><Vhost><M2><Endpoints><Endpoint>

   <Model Sparse="Off">https://alice.com/bob/#model.json</Model>

-  ``Sparse (기본: OFF)`` 모델 참조가 하나라도 실패하면 실패처리한다. ``ON`` 설정이라면 모든 모델 참조가 실패할 경우에만 실패처리 된다.

예를 들어 ``Sparse="On"`` 인 상황에서 apple과 cherry의 모델 참조가 실패하면 모델 배열은 다음과 같이 구성된다. ::

   {
      "mym" : [
         { },
         { "name": "banana", ... },
         { }
      ]
   }


쿼리스트링 모델변수
====================================

``<Model>`` 설정 외에 클라이언트로부터 직접 모델을 입력받을 수 있다. ::

   /users/platinum?mym=[apple,banana,cherry]&view=catalog&mynumber=123456&myage=24


위 주소의 쿼리 스트링 중 ``ModelParam`` 과 ``ViewParam`` 를 제외하면 약속된 쿼리스트링이 아니다. 이러한 키/값 쌍은 ``__query`` 의 자식 노드로 접근 가능하다. ::

   {
      "__query" : {
         "mynumber": "123456",
         "myage": "24"
      }
   }


모델 결합
====================================

``<Endpoint>`` 는 독립적으로 서로 영향을 받지 않는다. ::

   # vhosts.xml - <Vhosts><Vhost>

   <M2 Status="Active">
      <Endpoints>
         <Endpoint Alias="inven"> ... </Endpoint>
         <Endpoint Alias="golduser"> ... </Endpoint>
      </Endpoints>
   </M2>


.. figure:: img/m2_userguide_05.png
    :align: center


두 모델의 값을 비교,연산해야하는 경우가 있을 수 있다. 이런 경우 모델들을 결합하는 별도의 ``<Endpoint>`` 를 만들면 가능하다. ::

   # vhosts.xml - <Vhosts><Vhost>
   
   <M2 Status="Active">
      <Endpoints>
         <Endpoint Alias="inven"> ... </Endpoint>
         <Endpoint Alias="golduser"> ... </Endpoint>
         <Endpoint Alias="golditem">
            <Control ViewParam="view" ModelParam="model">/items/gold</Control>
            <Mapper>https://foo.com/mapper.json</Mapper>
            <View>https://bar.com/#view</View>
         </Endpoint>
      </Endpoints>
   </M2>

-  ``<Model>`` 태그가 없다면 모델 결합을 위한 ``<Endpoint>`` 로 인식한다.
-  ``@Alias`` 를 통해 다른 M2-JSON을 참조한다. (예. ``@inven`` , ``@golduser`` )

결합 맵퍼는 다음과 같이 작성한다. ::

   {
      "item" : {
         "inventory" : "@inven",
         "user" : "@golduser"
      },
      "description" : "this is a compound model"
   }

.. figure:: img/m2_userguide_06.png
    :align: center

``@Alias`` 뒤에 뷰를 명시하면 M2-JSON을 가공한 뷰를 참조할 수 있다. 단, 해당 뷰의 형식은 반드시 JSON이어야 한다.

.. figure:: img/m2_userguide_07.png
    :align: center

예제의 ``golditem`` 는 ``@inven`` 과 ``@golduser`` 의 엔드포인트를 참조한다. 따라서 각각의 모델 값을 ``키:값`` 을 콤마로 구분한다. ::

   /items/gold?mode=inven:1000,golduser:javalive&view=img



내장변수
====================================

내장변수는 __XXX 형식으로 표기되며 주로 M2-JSON의 메타 속성을 다루는 역할을 한다. ::

   {
      "firstName": "...",
      "address": {
         "streetAddress": "...",
         "city": "..."
      },
      "phoneNumber": ["..."],
      "__model_url" : "http://www.foo.com/goods?no=12345",
      "__model_raw" : "<html> ...(생략)... </html>"
   }

-  ``__model_url`` 모델이 참조된 URL
-  ``__model_raw`` 모델의 원시(RAW) 데이터 문자열



Mapper
====================================

맵퍼(Mapper)를 작성해 다양한 소스를 M2-JSON으로 맵핑(Mapping)한다.

.. figure:: img/m2_userguide_04.png
    :align: center


M2-JSON은 정보를 다루기 위한 JSON형식일 뿐 그 자체가 특별한 의미를 가지지 않는다. ::

   {
      "firstName": "...",
      "address": {
         "streetAddress": "...",
         "city": "..."
      },
      "phoneNumber": ["..."]
}

규칙은 간단하다.

-  값 참조 구분자는 ``space`` 이다. 예로 웹 페이지의 타이틀은 ``"html head title"`` 으로 표현한다.
-  맵핑하고 싶은 대상이 복수인 경우 값을 배열 ``["..."]`` 로 한다.



JSON
====================================

-  JSON은 별도의 맵핑 없이 M2-JSON으로 사용 가능하다.



HTML/XML
====================================

-  HTML과 XML 맵핑 규칙은 동일하며 추가적인 표현을 제공한다.
-  class 는 접두어 # 으로 참조한다.
-  id 는 접두어 . 으로 참조한다.
-  <Element>의 속성은 Element.속성키 으로 참조한다.

::

   <!DOCTYPE html>
   <html>
      <style type="text/css">
      <!--
         .foo {color:red};
         #bar {color:yellow};
         .foobar {color:cyan};
      //-->
      </style>
      <head>
         <title>Amazon.com: Online Shopping</title>
      </head>
      <body>        
         <h1>Amazon.com, Inc.</h1>
         <img id="foobar" src="https://amazon.com/logo.jpg" />
         <p class="foo">is an American multinational technology company </p>
         <p class="foo">based in Seattle that focuses on e-commerce,</p>
         <p class="foo">cloud computing, digital streaming, and artificial intelligence.</p>
      </body>
   </html>

예제 HTML은 다음과 같이 맵핑 가능하다. ::

   {
      "myTitle" : "html head title",
      "meta" : {
         "logo" : "#foobar img.src",
         "name" : "html body h1",
      },
      "descriptions" : [ ".foo"],
   }

위 맵핑은 아래와 같은 M2-JSON으로 변환된다. ::

   {
      "myTitle" : "Amazon.com: Online Shopping",
      "meta" : {
         "logo" : "https://amazon.com/logo.jpg",
         "name" : "Amazon.com, Inc.",
      },
      "descriptions" : [ 
         "is an American multinational technology company",
         "based in Seattle that focuses on e-commerce,",
         "cloud computing, digital streaming, and artificial intelligence."
      ]
   }