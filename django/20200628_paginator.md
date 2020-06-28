# django 프로젝트에서 페이징 기능 추가
* django는 개발자가 쉽게 페이징 기능을 적용할 수 있도록 Paginator모듈을 제공한다.
* 다음은 Paginator를 이용해서 페이징 기능을 추가하는 방법이다.
https://docs.djangoproject.com/en/3.0/ref/paginator/
## view 코딩하기
```
from django.core.paginator import Paginator, PageNotAnInteger, EmptyPage
from django.db.models import Q
from django.shortcuts import render
from tagging.views import TaggedObjectList
from .models import Post
# FBV에서 적용하기
def home_view(request):
    paginate_by = 15
    query = request.GET.get('q')
    page = request.GET.get('page', 1)
    paginator = Paginator(Post.objects.all(), paginate_by)  # Paginator객체에 object list(model) 및 페이지에 표시델 레코드 수를 인자로 전달한다.
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:  # Paginator에 정수가 아닌 값이 주어지면 예외가 발생합니다.
        posts = paginator.page(1)
    except EmptyPage:  # 해당 페이지에 레코드가 없는 경우에 예외가 발생합니다.
        posts = paginator.page(paginator.num_pages)

    if query:
        posts = posts.filter(Q(subject__icontains=query) | Q(content__icontains=query))

    return render(request, 'index.html', {
        'posts': posts,
    })

# CBV에서 적용하기 (ListView를 상속받는 클래스)
class PostTaggedObjectList(TaggedObjectList):
    model = Post
    template_name = 'tagging_post_list.html'
    paginate_by = 15  # paginate_by에 페이지에 표시될 레코드 수를 지정한다.
```


## Template 코딩하기
### index.html
```
{# home_view에서 생성한 Paginator 객체명을 기반으로 다음과 같이 페이지를 표시한다. #}
<div class="pagination">
    <span class="step-links">
        {% if posts.has_previous %}
            <a href="?page=1">&laquo; first</a>
            <a href="?page={{ posts.previous_page_number }}">previous</a>
        {% endif %}

        <span class="current">
            Page {{ posts.number }} of {{ posts.paginator.num_pages }}.
        </span>

        {% if posts.has_next %}
            <a href="?page={{ posts.next_page_number }}">next</a>
            <a href="?page={{ posts.paginator.num_pages }}">last &raquo;</a>
        {% endif %}
    </span>
</div>
```
### tagging_post_list.html
* LiseView에에서 전달받은 Paginator객체는 page_obj라는 객체명으로 접근이 가능하다.
* https://docs.djangoproject.com/en/3.0/topics/pagination/
```
<div class="pagination">
    <span class="step-links">
        {% if page_obj.has_previous %}
            <a href="?page=1">&laquo; first</a>
            <a href="?page={{ page_obj.previous_page_number }}">previous</a>
        {% endif %}

        <span class="current">
            Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
        </span>

        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">next</a>
            <a href="?page={{ page_obj.paginator.num_pages }}">last &raquo;</a>
        {% endif %}
    </span>
</div>
```
