.. _engine-prditem:

13장. 상품기술서 엔진
******************

상품기술서 엔진은 도큐먼트 엔진에서 파생되어 E-Commerce에 특화된 엔진이다. 
상품기술서 엔진을 통해 Mixed Contents 이슈 등 셀 수 없이 많은 상품기술서를 즉시 개선할 수 있다.

설정 구조는 다음과 같다. ::

   # /usr/local/m2/config-production.json

   {
      "m2": {
         "mixed" : {
            ...
         }
      }
   }


.. toctree::
   :maxdepth: 2


.. _engine-prditem-mixed-traffic:

Mixed Contents 트래픽 상세
====================================

처리의 종류 ::

   // -> https://
   http:// -> https://
   onloading
   https:// 그대로


/m2x/main
---------------------

/m2x/rebound
---------------------

/m2x/resource
---------------------



.. _engine-prditem-mixed-contents:

Mixed Contents 처리
====================================

Mixed Contents 엔진의 목적은 최소한의 ``URL`` 에 대해 SSL Onloading 을 적용하는 것이다.
아래와 같이 5단계의 정책 우선순위로 SSL Onloading 이 결정된다.

.. figure:: img/prditem01.png
   :align: center


*  ``IP`` IP 주소는 인증서를 탑재할 수 없다. SSL Onloading 한다.
*  ``Remain List`` 등록된 도메인에 대해서는 처리하지 않는다.
*  ``Black/White List`` 등록된 도메인에 강제로 SSL Onloading 여부를 결정한다.
*  ``SVL (SSL/TLS Validation List)`` m2live 서비스 데이터베이스를 참고하여 결정한다.
*  ``Syntax`` HTML 문법만으로 판단한다. ``http://`` 프로토콜 Scheme이 명시된 경우에만 SSL Onloading 한다.



.. _engine-prditem-mixed-contents-ip:

IP 정책
---------------------

상품기술서 내에 IP를 포함하는 ``URL`` 을 처리한다. ::

   <img src="http://182.162.143.217/cob/20FW/20FW_main2_kid780.jpg">
   <img src="http://182.162.143.217:8080/cob/20FW/20FW_main2_kid781.jpg">

위 태그는 다음과 같이 변환된다. ::

   <img src="https://foo.com/.../m2x/mixed/resource/http://182.162.143.217/cob/20FW/20FW_main2_kid780.jpg">
   <img src="https://foo.com/.../m2x/mixed/resource/"http://182.162.143.217:8080/cob/20FW/20FW_main2_kid781.jpg">">


SSL Onloading 여부를 설정할 수 있다. ::

   # m2.mixed

   "httpsUrls" : {
      "ip" : {
         "upgrade" : true
      }
   }

-  ``upgrade``

   -  ``true (기본)`` IP를 SSL Onloading한다.

   -  ``false`` IP를 처리하지 않는다.



Remain List
---------------------


Black/White List
---------------------


SVL
---------------------



Syntax
---------------------