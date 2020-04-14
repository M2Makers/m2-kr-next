.. _mvc:

6장. View
******************

뷰(View)는 ``M2-JSON`` 을 가공하여 사용자가 원하는 출력을 생성하는 템플릿을 의미한다. 
`Nunjucks <https://mozilla.github.io/nunjucks/>`_ 언어를 통해 ``M2-JSON`` 을 다룬다.

.. figure:: img/m2_userguide_08.png
    :align: center

.. note::
   
   View가 입력되지 않으면 ``M2-JSON`` 을 응답한다.


`Nunjucks <https://mozilla.github.io/nunjucks/>`_ 는 `Jinja2 <https://jinja.palletsprojects.com/>`_ 에 영감을 받은 언어이다. 
따라서 기본적인 `Jinja2 <https://jinja.palletsprojects.com/>`_ 의 문법이나 필터를 그대로 사용한다. ::

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

`Nunjucks <https://mozilla.github.io/nunjucks/>`_ 형식으로 다음과 같이 참조 가능하다. ::

   {{ model.firstname }}
   {{ model.address.state }}
   {{ model.phoneNumber.0.number }}


조건문, 반복문을 지원한다. ::

   {% if hungry %}
     I am hungry
   {% elif tired %}
     I am tired
   {% else %}
     I am good!
   {% endif %}


::

   <h1>Posts</h1>
   <ul>
   {% for item in items %}
      <li>{{ item.title }}</li>
   {% else %}
      <li>This would display if the 'item' collection were empty</li>
   {% endfor %}
   </ul>



포맷
====================================

HTML, XML
------------------------------------

HTML, XML 템플릿을 만든다. ::

   <html>
   <body>
      <H1>{{ model.firstname }} {{ model.lastName }}</H1>
      <p>{{ model.address.city }}</p>
   </body>
   </html>


별도의 ``<meta>`` 태그가 필요없다.


JPG, PNG, WEBP, BMP, PDF
------------------------------------

이미지 출력은 HTML 템플릿을 기반으로 렌더링한다. 
``<meta>`` 태그를 통해 출력 포맷을 지정한다. 
다음은 PNG 이미지를 가로 400, 세로 300으로 출력하는 예제이다. ::

   <!DOCTYPE html>
   <html>
      <head>
         <meta name="m2-render-png" content="width=400;height=300;" />
         <style>
            p { display: block; margin-top: 1em; margin-bottom: 1em; }
         </style>
      </head>
      <body>
         <H1>{{ model.firstname }} {{ model.lastName }}</H1>
         <p>{{ model.address.city }}</p>
      </body>
   </html>

이하 이미지 포맷에 따라 ``name`` 값과 지원 옵션이 다르다. 입력되지 않은 기본 값은 다음과 같다.

============== ================= ========================
속성            설명               기본값
============== ================= ========================
``width``       가로 픽셀         400
``height``      세로 픽셀         300
``quality``     JPEG 품질(%)      100
============== ================= ========================


이미지 포맷별 ``<meta>`` 태그 예제는 다음과 같다.

-  JPG ::
      
      <meta name="m2-render-jpg" content="width=400;height=300;quality=85" />

-  PNG ::
      
      <meta name="m2-render-png" content="width=400;height=300;" />

-  WEBP ::
      
      <meta name="m2-render-webp" content="width=400;height=300;quality=85" />

-  BMP ::
      
      <meta name="m2-render-bmp" content="width=400;height=300;" />

-  PDF ::
      
      <meta name="m2-render-pdf" content="width=400;height=300;scale=1;margin-top: 10px;margin-bottom:10px;margin-right:10px;margin-left:10px;" />


MP4, GIF
------------------------------------

비디오, Animated GIF 등 시간흐름이 필요한 포맷은 연속된 장면( ``<Scene>``)을 연결하여 만든다.

.. figure:: img/m2_userguide_09.png
    :align: center


다음과 같이 ``<Scene>`` 태그를 통해 각 화면을 구성한다. ::

   <!DOCTYPE html>
   <html>
      <head>
         <meta name="m2-render-gif" content="width=400;height=300;delay=1000;" />
         <style>
            p { display: block; margin-top: 1em; margin-bottom: 1em; }
         </style>
      </head>
      <body>
         <Scene>
            <Div style="background-color: blue;">
               <H1>{{ model.firstname }} {{ model.lastName }}</H1>
               <p>{{address.city}}</p>
            </Div>
         </Scene>
         <Scene>
            <Div style="background-color: blue;">
               <H1>{{ model.lastName }} {{ model.firstname }} </H1>
               <p>{{ model.address.city }}</p>
            </Div>
         </Scene>
         <Scene>
            <Div style="background-color: green;">
               <H1>{{ model.lastName }} {{ model.firstname }} ({{ model.age }})</H1>
               <p>{{ model.address.city }}</p>
            </Div>
         </Scene>
      </body>
   </html>

``<Scene>`` 태그는 의미가 없다. 따라서 ``<Div>`` 를 넣어 영역을 구분하면 개발 단계에서 쉽게 확인이 가능하다.

-  MP4 ::
      
      <meta name="m2-render-mp4" content="width=400;height=300;interval=1000;" />


-  GIF ::
      
      <meta name="m2-render-gif" content="width=400;height=300;delay=1000;" />

   -  장면 시간( ``delay (단위: ms)`` ) = 1000


JSON
------------------------------------

JSON 템플릿을 만든다. ::

   {
      "myName" : "{{firstname}} {{lastName}}",
      "myCity" : "{{address.city}}"
   }



필터
====================================

M2는 View에서 활용할 수 있는 필터를 제공한다. 
필터는 ``<meta>`` 태그로 명시하며 다음 형식을 가진다. ::

<meta name="m2-filter-XXX" content="키1=값1;키2=값2;...">


m2-filter-image
------------------------------------

페이지에서 참조하는 이미지를 일괄처리한다. 
View에 
상품기술서처럼 크기와 용량이 큰 이미지를 효율적으로 전송하기에 적합니다. ::

   <html>
      <head>
         <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
         <meta name="m2-filter-image" content="imagetool-url=https://img.example.com/m2/images;splitheight=500;function=optimize,sref,fitimage;addclass=m2div;">
         <style>
            .m2div {
               display: inline-block;
               width: 100%
            }
         </style>
      </head>
      <body>
         {{ model.__raw }}
      </body>
   </center>


===================== ===================================================================================
키                     값
===================== ===================================================================================
imagetool-url          프로토콜`이미지 툴 <https://ston.readthedocs.io/ko/latest/admin/image.html>`_ 을 설정한 URL
split-height           이미지가 설정된 값(px)을 초과할 경우 여러 이미지로 분할한다.
imagetool-functions    이미지 가공함수
sref                   원본이미지 주소를 넘길 파라미터 키
fitimage               AAAAAAAAA
addclass               BBBBBBBBBBB
===================== =====================================