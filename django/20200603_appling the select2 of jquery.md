# admin에서 select2 태그 적용하기
## admin에서 사용하는 정적파일 경로 확인
* settings.py에서 installed app의 admin을 따라 가보면 다음과 같이 경로를 확인할 수 있음
  * css path: admin/static/admin/css/vendor/select2/select2.css
  * jquery path: admin/static/js/vendor/jquery/jquery.j
  * select2 path: admin/static/js/vendor/select2/select2.full.js가 있음
## 적용
다음과 같이 모든 admin 수정 폼에 select2를 기본 적용한다.
* change_form.html을 상속받아서, 다음 코드를 추가함.

```
{% extends "admin/change_form.html" %}
{% load static %}
{% block extrahead %}{{ block.super }}
<link rel="stylesheet" type="text/css" href="{% static 'admin/css/vendor/select2/select2.min.css' %}"/>
{% endblock %}

{% block admin_change_form_document_ready %}
    {{ block.super }}
    <script src="{% static 'admin/js/vendor/jquery/jquery.min.js' %}"></script>
    <script src="{% static 'admin/js/vendor/select2/select2.full.min.js' %}"></script>
    <script>
        $('select').elect2();
    </script>
{% endblock %}
```
