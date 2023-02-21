# Django MVT

參考內容來自：

https://ithelp.ithome.com.tw/users/20129810/ironman/3317 （django 學習筆記）
https://ithelp.ithome.com.tw/articles/10212502 (資料庫與模型進階技巧)
https://www.learncodewithmike.com/2020/04/django-custom-form-field-validation.html （Django ModelForm客製化表單欄位驗證的技巧）

Django 採用了MVT的軟體設計模式，即模型（Model），視圖（View）和模板（Template）

![](https://i.imgur.com/srifiDB.png)

* Model : 主要負責與資料有關的直接處理，定義物件關聯對映(ORM)。有對資料庫直接存取的權力。
* View : 它主要負責處理 Model 和 Template 之間的邏輯，並做出回應。
* Template : 負責資料的顯示，其實就是使用者最後所看到的頁面啦！

### 建立 Django 專案
```
django-admin startproject <projectname>
```

manage.py 是 Django 提供的命令列工具，提供許多不同功能的指令，輸入 help 或 -h 指令會列出所有指令列表:
```
python3 manage.py -h
```

### 啟動 django 伺服器
```
python3 manage.py runserver
```

### 建立 Django app
```
python3 manage.py startapp <appname>
```

為了讓 Django 知道要管理哪些 app，我們必須將這個 app 加入 Django 的設定檔中。
打開 ```<projectname>/settings.py``` 並找到 INSTALLED_APPS 後，將自己新增的 app 放進去
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    <appname>,
]
```

### Django (View)

前往 myapp/views.py
```
from django.http import HttpResponse

def hello_world(request):
    return HttpResponse("Hello World!")
```

前往 myproject/urls.py 中
```
from myapp.views import hello_world
```

在 urlpatterns 中新增
```
path('hello/', hello_world),
```

成功了！

![](https://i.imgur.com/H0oEjcZ.png)

### Django (Templates)

在 settings.py 搜尋 TEMPLATES, 在 DIRS 中加入```os.path.join(BASE_DIR, 'templates')```

![](https://i.imgur.com/FoymsOz.png)

myproject & myapp 之外再建立一個 templates， 裡頭建立 一個 hello_world.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>hello_world</title>
</head>
<body>
    <h1>hello_world!</h1>
</body>
</html>
```

進入 myapp/views.py 中，修改 hello_world
```
from django.shortcuts import render
from django.http import HttpResponse

def hello_world(request):  
   return render(request, 'hello_world.html') #修改
```

增加變數
```
def hello_world(request): #修改
    a = 10
    b = 5
    s = a + b
    d = a - b
    p = a * b
    q = a / b
    return render(request, 'hello_world.html', locals())
```

locals()函數會以字典類型返回當前位置的全部局部變量, 使用locals()就不用一個一個手動輸入欲回傳


修改 hello_world.html 
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>hello_world</title>
</head>
<body>
    <h1>hello_world!</h1>
    <p>a = {{a}}</p>
    <p>b = {{b}}</p>
    <p>a + b = {{s}}</p>
    <p>a - b = {{d}}</p>
    <p>a * b = {{p}}</p>
    <p>a / d = {{q}}</p>
</body>
</html>
```

### Django (Model)

進入 myapp 並在 models.py 貼上
```
from django.db import models

class Restaurant(models.Model):
    name = models.CharField(max_length=20) #餐廳名稱
    phone_number = models.CharField(max_length=15) #餐廳電話
    address = models.CharField(max_length=50, blank=True)  #餐廳地址

class Food(models.Model):
    name = models.CharField(max_length=20) #食物名稱
    price=models.DecimalField(max_digits=3,decimal_places=0) #食物價錢
    is_spicy = models.BooleanField(default=False) #會不會辣
    food_restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE) #由哪個餐廳製作
```

檢查自己設計的model有沒有問題
```
python3 manage.py check
```

建立migration記錄檔
```
python manage.py makemigrations myapp
```

正式將模型與資料庫同步
```
python3 manage.py migrate myapp 0001
```

![](https://i.imgur.com/H9TCwrx.png)

![](https://i.imgur.com/swvVPRt.png)

#### 資料新增

進圖 django shell
```
python3 manage.py shell
```

引入剛剛建立的models.py當中的 table:
```
>>> from restaurants.models import Restaurant, Food
```

新增資料 方法（一）
```
>>> restaurant1=Restaurant(name='一號店', phone_number='01-1111111', address='一號路')
>>> restaurant1.save()
```

新增資料 方法（一）
```
restaurant2=Restaurant.objects.create(name='二號店', phone_number='02-2222222', address='二號路')
```

查看資料庫
```
Restaurant.objects.all()
```

#### 更新資料

用 filter 對資料過慮並搜尋，透過 update 更新
```
Restaurant.objects.filter(address='一號路').update(address='一號巷')
```

查看更新後的資料（adress）
```
>>> r1 = Restaurant.objects.get(name='一號店')
>>> r1.address
```

#### 刪除資料

```
Restaurant.objects.get(name='一號店').delete()
```

#### 關聯

![](https://i.imgur.com/JXzhIBj.png)

```
>>> r2 = Restaurant.objects.get(name='二號店')
>>> food = Food.objects.create(name='便當', price='60', is_spicy=False, food_restaurant=r2)
>>> food.food_restaurant.name
'二號店'
```

透過食物關聯到餐廳，並取得餐廳的名子

### Django (Form)

表單驗證：開發網站表單時，預設會依據資料模型(Model)中，所定義的欄位類型及參數設定，執行基本的檢核，但是實務上，有一些欄位可能需要特殊的檢核邏輯，需要客製化自己的表單驗證機制。

在models.py檔案中，新增一個資料模型類別
```
from django.db import models
class Customer(models.Model):
    name = models.CharField(max_length=30, blank=False, null=False)
    email = models.EmailField(blank=False, null=False)
    tel = models.IntegerField()
```

同步至資料庫中
```
python3 manage.py makemigrations
python3 manage.py migrate
```

將Django資料模型加入至Django Administration(管理員後台)，在 admin.py 新增：
```
from django.contrib import admin
from .models import Customer
class CustomerAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'email', 'tel')  # 顯示欄位
admin.site.register(Customer, CustomerAdmin)  # 加入至Administration(管理員後台)
```

執行 & 檢視資料的新增狀況：
```
python3 manage.py runserver
```

![](https://i.imgur.com/69lKMKj.png)
