# django-tagging 라이브러리를 이용한 태그 기능 추가하기
## 설치
```
# repo: https://github.com/Fantomas42/django-tagging
pip install django-tagging
```

## 설정하기

```
# settings.py
INSTALLED_APPS = [
    'tagging',  # 추가
]
```
* django-tagging 관련 테이블 마이그레이션
```
python managy.py makemigrations
python managy.py migrate
```

## 블로그 앱에 태그 기능 추가하기
### 추가할 기능
* 태깅 기능 (어드민에서 태그를 추가할 수 있음)
* 태그 클라우드 페이지
  * 모든 태그를 확인할 수 있다.
  * 태그의 사용빈도를 시각화 한다.
* 태깅된 리스트 뷰 (태그를 클릭하면, 그룹핑된 포스트만 리스팅된다.)


### url 작성
```
# urls.py
urlpatterns = [
    # ...
    path('tag/', views.tag_view, name='tag_cloud'),  # 태그 클라우드 url
    path('tag/<tag>/', views.PostTaggedObjectList.as_view(), name='tagged_object_list'),  # 태깅된 리스트 뷰 url
]
```


### view 작성하기
```
from django.shortcuts import render
from tagging.views import TaggedObjectList
from .models import Post

def tag_view(request):
    '''태그 클라우드, 뷰에서는 단순한 템플릿 뷰로 구현하고 템플릿에서 태그 리스트를 참조하면 된다.'''
    return render(request, 'tagging_cloud.html')


class PostTaggedObjectList(TaggedObjectList):
    '''
    * 태깅된 리스트 뷰
    * TaggedObjectList를 상속받은 클래스기반 뷰. TaggedObjectList는 django-tagging라이브러리를 추가하고 settings.py에 앱을 추가하면 참조 가능하다.
    * 태깅할 모델(Post)과 렌더링 할 페이지(tagging_post_list.html)를 설정한다.
    '''
    model = Post
    template_name = 'tagging_post_list.html'
```

### template 작성하기
* tagging_cloud.html 작성하기: 모든 태그 및 사용 빈도를 시각화 하는 화면 
```
{# tagging_cloud.html #}

{% extends 'base.html' %}
{% load static %}
{% block extracss %}
    <style>
        .tag-cloud {
            width: 30%;
            margin-left: 30px;
            text-align: center;
            padding: 5px;
            border: 1px solid orange;
            background: #ffc;
        }

        .tag-1 {
            font-size: 12px;
        }

        .tag-2 {
            font-size: 14px;
        }

        .tag-3 {
            font-size: 16px;
        }

        .tag-4 {
            font-size: 18px;
        }

        .tag-5 {
            font-size: 20px;
        }

        .tag-6 {
            font-size: 24px;
        }
    </style>
{% endblock %}
{% block header %}
    <!-- Page Header -->
    <header class="masthead" style="background-image: url('{% static 'img/home-bg.jpg' %}')">
        <div class="overlay"></div>
        <div class="container">
            <div class="row">
                <div class="col-lg-8 col-md-10 mx-auto">
                    <div class="site-heading">
                        <h1>Blog Tag Cloud</h1>
                        <span class="subheading">A Blog Theme by Start Bootstrap</span>
                    </div>
                </div>
            </div>
        </div>
    </header>
{% endblock %}
{% block content %}
    {{ block.super }}
    <!-- Post Content -->
    <div class="row">
        <div class="col-lg-8 col-md-10 mx-auto tag-cloud">
            {% load tagging_tags %}
            {% tag_cloud_for_model posts.Post as tags with steps=6 min_count=1 distribution=log %}
            {# posts.Post: 태그를 추출할 대상은 posts앱의 post모델임, as tags: 태그 리스트를 tags라는 변수에 담음 #}
            {# with steps=6 min_count=1: 태그 폰트 범위를 1~6, 출력용 최소 사용 횟수를 1로 정함 #}
            {# distribution=log:태그 폰트의 크기를 할당할 때 수학 Logarithmic 알고리즘을 사용. #}
            {% for tag in tags %}
                <span class="tag-{{ tag.font_size }}">
                        <a href="{% url 'posts:tagged_object_list' tag.name %}">{{ tag.name }} ({{ tag.font_size }})</a>
                        </span>
            {% endfor %}
        </div>
    </div>
    <hr>
{% endblock %}
```
ps.
* {% load tagging_tags %}를 통해 tags객체를 불러올 수 있다.
* tag객체에서 {{ tag.font_size }}을 통해 태그 사용 빈도를 반환받는다.
---

* tagging_post_list.html 작성하기
  * 태그별로 그룹핑된 리스트를 출력하는 화면 구현
  * 뷰에서 이미 그룹핑된 객체가 넘어오기 때문에 리스트 뷰와 동일하게 작성하면 된다. 
```
{% extends 'base.html' %}
{% load static %}
{% block header %}
    <!-- Page Header -->
    <header class="masthead" style="background-image: url('{% static 'img/home-bg.jpg' %}')">
        <div class="overlay"></div>
        <div class="container">
            <div class="row">
                <div class="col-lg-8 col-md-10 mx-auto">
                    <div class="site-heading">
                        <h1>Posts for tag - {{ tag.name }}</h1>
                        <span class="subheading">A Blog Theme by Start Bootstrap</span>
                    </div>
                </div>
            </div>
        </div>
    </header>
{% endblock %}
{% block content %}
    {{ block.super }}
    <!-- Main Content -->
    <div class="row">
        <div class="col-lg-8 col-md-10 mx-auto">
            {% for post in object_list %}
                <div class="post-preview">
                    <a href="{% url 'posts:detail' post.pk %}">
                        <h2 class="post-title">
                            {{ post.subject }}
                        </h2>
                        <h3 class="post-subtitle">
                            {{ post.content|safe|striptags|slice:":64" }}
                        </h3>
                    </a>
                    <p class="post-meta">
                        Posted by {{ post.user }}
                        on {{ post.created|date:"M" }} {{ post.created.day }}, {{ post.created.year }}
                    </p>
                </div>
            {% endfor %}
            <!-- Pager -->
            <div class="clearfix">
                <a class="btn btn-primary float-right" href="#">Older Posts &rarr;</a>
            </div>
        </div>
    </div>
    <hr>
{% endblock %}
```



