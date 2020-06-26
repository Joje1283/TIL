# 장고 어드민에서 액션 필드를 이용한 객체(레코드) 복사
PS. 작성한 기능에서 당장 필요없는 코드는 주석처리 함.
## 기능
* 장고 어드민에서 모델의 객체(데이터베이스에서 테이블의 레코드)를 복사하는 UI를 구현한다.
* 어드민의 해당 모델에 대한 리스트에서 UI를 확인할 수 있다.

```
# admin.py
from django.contrib import admin
import copy  # (1) use python copy

# action
def copy_semester(modeladmin, request, queryset):  # action method
    for sd in queryset:
        sd_copy = copy.copy(sd)  # (2) django copy object
        sd_copy.id = None  # (3) set 'id' to None to create new object
        sd_copy.save()  # initial save

        # (4) copy M2M relationship: instructors
        # for instructor in sd.instructors.all():
        #     sd_copy.instructors.add(instructor)

        # (5) copy M2M relationship: requirements_met
        # for req in sd.requirements_met.all():
        #     sd_copy.requirements_met.add(req)

        # zero out enrollment numbers.
        # (6) Use __dict__ to access "regular" attributes (not FK or M2M)
        # for attr_name in ['enrollments_entered', 'undergrads_enrolled', 'grads_enrolled', 'employees_enrolled',
        #                   'cross_registered', 'withdrawals']:
        #     sd_copy.__dict__.update({attr_name: 0})
        #
        # sd_copy.save()  # (7) save the copy to the database for M2M relations

@admin.register(RedeemableCodeCreation)
class MyModelAdminAdmin(admin.ModelAdmin):
    actions = [copy_semester]  # 추가
    list_display = ('pk', 'title')

# 출처: https://blogs.harvard.edu/rprasad/2012/08/24/using-django-admin-to-copy-an-object/
```
