# django admin에 summernote 적용하기
: (https://github.com/summernote/django-summernote)
## 설치
### 설치하기
```$ pip install django-summernote```

### settings.py에 추가
```
INSTALLED_APPS = [
    # ...
    'django_summernote',
]
MEDIA_URL = '/media/'  # 그림 등의 첨부파일이 저장될 미디어 경로 URL
MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')  # 미디어 경로 패스
```
### urls.py에 추가
```
from django.urls import include
...
urlpatterns = [
    ...
    path('summernote/', include('django_summernote.urls')),
    ...
]

from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:  # DEBUG 활성화 되었을때 urlpatterns에 다음 값을 더한다.
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### Admin에 적용하기
#### 모델의 모든 TextField에 Summernote 적용하기
```
from django_summernote.admin import SummernoteModelAdmin
from .models import SomeModel
from django.contrib import admin

@admin.registor(SomeModel)
class SomeModelAdmin(SummernoteModelAdmin):  # instead of ModelAdmin
    summernote_fields = '__all__'

admin.site.register(SomeModel, SomeModelAdmin)
```

#### 모델의 특정 필드에 Summernote 적용하기
```
from django_summernote.admin import SummernoteModelAdmin
from .models import Post
from django.contrib import admin

@admin.register(Post)
class PostAdmin(SummernoteModelAdmin):
    summernote_fields = ('content',)  # Content필드에만 Summernote가 적용됨
```

#### 템플릿에서 출력하기
```
{{ post.content | safe }}
```

### Form에 적용하기
#### 일반 폼(form)에 적용하기
```
from django_summernote.widgets import SummernoteWidget, SummernoteInplaceWidget

# Apply summernote to specific fields.
class SomeForm(forms.Form):
    foo = forms.CharField(widget=SummernoteWidget())  # instead of forms.Textarea

# If you don't like <iframe>, then use inplace widget
# Or if you're using django-crispy-forms, please use this.
class AnotherForm(forms.Form):
    bar = forms.CharField(widget=SummernoteInplaceWidget())
```

#### 모델 폼(modelform)에 적용하기
```
class FormFromSomeModel(forms.ModelForm):
    class Meta:
        model = SomeModel
        widgets = {
            'foo': SummernoteWidget(),
            'bar': SummernoteInplaceWidget(),
        }
```

#### 템플릿에 출력
```
{{ foobar|safe }}
```

## 추가 내용
경고 : 위젯은 이스케이프를 제공하지 않습니다. 이를 관리하지 않고 위젯을 외부 사용자에게 노출하면 주입 취약점이 발생할 수 있습니다. 따라서, mozilla's package bleach를 통해 모든 유해한 태그를 피하는 SummernoteTextFormField 또는 SummernoteTextField를 사용할 수 있습니다.
### form에서
```
from django_summernote.fields import SummernoteTextFormField, SummernoteTextField

class SomeForm(forms.Form):
    foo = SummernoteTextFormField()
```

### ModelForm에서
```
class FormForSomeModel(forms.ModelForm):
    foo = SummernoteFormField()
```

