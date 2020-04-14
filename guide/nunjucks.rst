.. _nunjucks:

Appendix B: Nunjucks
***********************

`Nunjucks <https://mozilla.github.io/nunjucks/>`_ 에서 자주 사용되는 기능에 대해 설명한다. 

-  `Nunjucks <https://mozilla.github.io/nunjucks/>`_
-  `Jinja2 - Template Designer Documentation <https://jinja.palletsprojects.com/en/2.11.x/templates/>`_

간단한 테스트는 `Jinaj2 live parser <http://jinja.quantprogramming.com/>`_ 가 유용하다.


문자열
====================================

`자르기 <https://mozilla.github.io/nunjucks/templating.html#truncate>`_
------------------------------------

문자열을 원하는 길이만큼 자른다. ::

   truncate(s, length=255, killwords=False, end='...', leeway=None)

-  ``s`` - 입력 문자열
-  ``length (기본: 255)`` - 자를 문자열 길이
-  ``killwords (기본: false)`` - ``true`` 라면 단어가 잘리더라도 길이에 맞추어 정확히 자른다. ``false`` 라면 잘리는 단어를 버린다.
-  ``end (기본: ...)`` - 단어가 잘렸다면 이 값을 문자열에 붙인다.
-  ``leeway`` 

::

  {{ "foo bar baz qux"|truncate(9) }}
      -> "foo..."
  {{ "foo bar baz qux"|truncate(9, True) }}
      -> "foo ba..."
  {{ "foo bar baz qux"|truncate(11) }}
      -> "foo bar baz qux"
  {{ "foo bar baz qux"|truncate(11, False, '...', 0) }}
      -> "foo bar..."