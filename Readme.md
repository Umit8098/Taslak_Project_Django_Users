
## Proje Taslak V.01

```powershell
- py -m venv env
- .\env\Scripts\activate
- pip install djangorestframework
- django-admin startproject main .
- pip install python-decouple
- py manage.py runserver
- py manage.py migrate
- pip freeze > requirements.txt
```

- add rest_framework to INSTALLED_APPS
- create .env and .gitignore (hidden to SECRET_KEY)

```powershell
- py manage.py runserver
- py manage.py migrate 
```

### PostgreSQL setup

- To get Python working with Postgres, you will need to install the “psycopg2” module.
(Python'un Postgres ile çalışmasını sağlamak için “psycopg2” modülünü kurmanız gerekecek.)

```powershell
- pip install psycopg2
- pip install psycopg2-binary # for macOS
- pip freeze > requirements.txt
```

### Install Swagger

- Dokümantasyon için kullanılır, backend de oluşturduğumuz endpointlerin daha doğrusu API nin dokümantasyonu.
- Swagger, 2010 yılında bir girişim tarafından başlatılan açık kaynaklı bir projedir. Amaç, bir framework uygulamaktır.kod ile senkronizasyonu korurken, geliştiricilerin API'leri belgelemesine ve tasarlamasına olanak sağlamaktır.
- Bir API geliştirmek, düzenli ve anlaşılır dokümantasyon gerektirir.
- API'leri Django rest frameworküyle belgelemek ve tasarlamak için gerçek oluşturan drf-yasg kullanacağız.
Bir Django Rest Framework API'sinden Swagger/Open-API 2.0 belirtimleri. belgeler burada bulabilirsin

- drf-yasg -> django rest_framework yet another swagger generator

- installation

```powershell
- pip install drf-yasg
- pip freeze > requirements.txt
```

- add drf-yasg to INSTALLED_APPS -> 'drf_yasg'

- swagger için güncellenmiş urls.py aşağıdaki gibidir. Swagger belgelerinde, bu modeller güncel değildir. urls.py'yi aşağıdaki gibi değiştiriyoruz.

urls.py ->

```py
from django.contrib import admin 
from django.urls import path 
 
# Three modules for swagger:
from rest_framework import permissions 
from drf_yasg.views import get_schema_view 
from drf_yasg import openapi 
 
schema_view = get_schema_view( 
    openapi.Info( 
        title="Taslak Project-Flight Reservation API", 
        default_version="v1",
        description="Taslak Project-Flight Reservation API project provides flight and reservation info", 
        terms_of_service="#", 
        contact=openapi.Contact(email="rafe@clarusway.com"),  # Change e-mail on this line! 
        license=openapi.License(name="BSD License"), 
    ), 
    public=True, 
    permission_classes=[permissions.AllowAny], 
) 
 
urlpatterns = [ 
    path("admin/", admin.site.urls), 
    # Url paths for swagger: 
    path("swagger(<format>\.json|\.yaml)", schema_view.without_ui(cache_timeout=0), name="schema-json"), 
    path("swagger/", schema_view.with_ui("swagger", cache_timeout=0), name="schema-swagger-ui"), 
    path("redoc/", schema_view.with_ui("redoc", cache_timeout=0), name="schema-redoc"), 
]
```


```powershell
- py manage.py runserver
- py manage.py migrate 
```

- swagger ı kurduk, urls.py daki ayarlarını da yaptık, http://127.0.0.1:8000/swagger/  endpointine istek attığımızda swagger ana sayfasını gördük.

### Install Debug Toolbar

- The Django Debug Toolbar is a configurable set of panels that display various debug information about the current request/response and when clicked, display more details about the panel’s content
  https://django-debug-toolbar.readthedocs.io/en/latest/
(Django Hata Ayıklama Araç Çubuğu, çeşitli hata ayıklama bilgilerini görüntüleyen yapılandırılabilir bir panel kümesidir. geçerli istek/yanıt ve tıklandığında panel içeriği hakkında daha fazla ayrıntı görüntüler)

- Install the package

```powershell
- pip install django-debug-toolbar
- pip freeze > requirements.txt
```

- Add "debug_toolbar" to your INSTALLED_APPS setting -> "debug_toolbar"

settings.py ->

```py
INSTALLED_APPS = [
    'django.contrib.staticfiles',
    # 3rd party packages
    # ....
    'debug_toolbar',
]
```

- Add django-debug-toolbar’s URLs to your project’s URLconf
 
urls.py ->
```py
from django.urls import path, include

urlpatterns = [ 
    # ... 
    path('__debug__/', include('debug_toolbar.urls')), 
] 
```

- Add the middleware to the top:

settings.py ->

```py
MIDDLEWARE = [ 
    "debug_toolbar.middleware.DebugToolbarMiddleware", 
    # ... 
] 
```

- Add configuration of internal IPs to "settings.py":

settings.py ->

```py
INTERNAL_IPS = [ 
    "127.0.0.1", 
]
```

- test edelim çalışacak mı?

```powershell
- py manage.py runserver
```

- çalıştı,   http://127.0.0.1:8000/   endpointine istek atınca gelen sayfanın sağ tarafında bir tool DjDT açılıyor.


- create superuser
```powershell
- py manage.py creaetsuperuser
```


### Seperate Dev and Prod Settings (Development ve Product ayarlarını ayırma)

- Şu ana kadar yaptığımız projelerde tüm projede geçerli olan ayarları settings.py altında topluyorduk. Ancak ideal bir ortamda çok tavsiye edilen bir durum değil. 
- Projenin bulunduğu safhaya göre settings.py ayarları değişeceği için, her ortam için ayrı bir settings.py oluşturulur. 
- Dokümanda görüldüğü gibi bir structure oluşturup, 

```text
settings/
    |- __init__.py
    |- base.py
    |- ci.py
    |- local.py
    |- staging.py
    |- production.py
    |- qa.py
```

    Yukarıdaki örnekteki gibi tek bir yerde hepsinin toplanmasından sa ayrı ayrı dosyalar oluşturup, mesela tüm projede geçerli olan ortak ayarları base.py içerisine almış, ci->Continuous integrations, qa->test gibi setting.py ı parçalara ayırıyoruz.
    
    Biz şuanda temel olarak hemen hemen her projede olan developer ve produc ortamını birbirinden ayıracağız.

- main klasörü içerisine settings klasörü create et, içerisine;
  - __init__.py  # burasının bir python package olduğunu gösterir
  - base.py   
  - dev.py   
  - prod.py   

- main içerisindeki settings.py içeriğini, yeni oluşturulan settings/base.py içerisine kopyalıyoruz.

- Artık settings.py dosyası silinebilir. (Biz ismini değiştirdik duruyor.)

- Şimdi artık settings.py daki kodlarımızı ayırmaya başlıyoruz, ->

##### development settings ayarları ->

- dev.py da olacak kodlar ->
  -  base.py dakileri import et, ve şu kodları ekle;
 
```py
from .base import *

THIRD_PARTY_APPS = ["debug_toolbar"] 
 
DEBUG = config("DEBUG") 
 
INSTALLED_APPS += THIRD_PARTY_APPS 
 
THIRD_PARTY_MIDDLEWARE = ["debug_toolbar.middleware.DebugToolbarMiddleware"] 
 
MIDDLEWARE += THIRD_PARTY_MIDDLEWARE 
 
# Database
# https://docs.djangoproject.com/en/4.0/ref/settings/#databases 
DATABASES = { 
    "default": { 
        "ENGINE": "django.db.backends.sqlite3", 
        "NAME": BASE_DIR / "db.sqlite3", 
    } 
} 
 
INTERNAL_IPS = [ 
    "127.0.0.1", 
]
```

  -  DEBUG = config("DEBUG") dediğimiz için base.py daki DEBUG kısmını siliyoruz/yoruma alıyoruz. Artık base.py da DEBUG yok, dev.py a bakıyoruz orada diyor ki config den al bunu, config nereden alıyordu? .env den alıyordu, oraya gidip "DEBUG = True" ekliyoruz. DEBUG ı base den kaldırdık, artık developer dayken config e bakacak orda ne yazıyorsa onu kullanacak.

  -  THIRD_PARTY_APPS = ["debug_toolbar"]  ı dev.py a almış, o yüzden base.py daki INSTALLED_APPS deki debug_toolbar ı kaldırıyoruz. debug_toolbar ne zaman çalışacak? sadece developer dayken çalışacak.
  (INSTALLED_APPS += THIRD_PARTY_APPS komutu ile eklemiş.)
 
  -  debug_toolbar ı dev.py a aldığımıza göre yine kendisi ile birlekte gelen middleware ı vardı yine onu da buraya alıyoruz ve base.py da yoruma alıyoruz.

  -  Default olarak djangoda sql3 kullanılıyordu, o da burada, o zaman base dekini yoruma/siliyoruz. Developer dayken sqlite kullan, onu da burada belirtiyorum.

  -  Yine INTERNAL_IPS i developer dayken kullan diyoruz ve base.py dan siliyoruz/yoruma alıyoruz.
  -  development ayarlarını ayırdık.

##### product settings ayarları ->

- Peki product halindeyken nasıl olacak? 
  - Aynı şekilde base.py daki herşeyi import et,

prod.py ->

```py
from .base import * 
 
DATABASES = { 
    "default": { 
        "ENGINE": "django.db.backends.postgresql_psycopg2", 
        "NAME": config("SQL_DATABASE"), 
        "USER": config("SQL_USER"), 
        "PASSWORD": config("SQL_PASSWORD"), 
        "HOST": config("SQL_HOST"), 
        "PORT": config("SQL_PORT"), 
        "ATOMIC_REQUESTS": True, 
    }
}
```
  
  - product ortamındayken şunu ilave et, bu ne? database. dev deyken sqlite idi, product ta iken bu! Burada ne var? PostgreSQL. Bağlantıyı yapacağız şimdi.
  
  - Buraya bakıyoruz yine configden kullandığı değişkenler var. Bu değişkenleri alıp .env file ımıza ekleyelim.
  
    SQL_DATABASE =  
    SQL_USER =
    SQL_PASSWORD =
    SQL_HOST = 
    SQL_PORT = 
  
  - Artık product taki postgresql server ın bağlantıyı kurabilmesi için gerekli parametreleri .env ye aldık. Şimdi bunları dolduralım. pgAdmin i çalıştırıyoruz, ve flight isminde bir db create ediyoruz, daha sonra .env deki değişkenleri create ettiğimiz db deki değişkenlerle update ediyoruz.
  
    SQL_DATABASE = flight
    SQL_USER = postgres
    SQL_PASSWORD = postgres
    SQL_HOST = localhost
    SQL_PORT = 5432
  
  - Böylelikle development da iken sqlite, product da iken postgresql kullanmasını söyledik.


- Biz prod ve dev diye ayırdık ama bunlardan hangisini kullanacağını nasıl bilecek?

##### __init__.py settings ayarları ->

- Onun için settings/ __init__.py file ına;

```py
from .base import * 


env_name = config("ENV_NAME") 
 
if env_name == "prod": 
 
    from .prod import * 
 
elif env_name == "dev": 
 
    from .dev import *
```

- kodları yazıyoruz. Burada ne demek istiyoruz? 
  - base.py dan herşeyi al, 
  - eğer config deki ENV_NAME i prod ise from .prod import * bunu ekle, 
  - değil config deki ENV_NAME i dev ise from .dev import * bunu ekle diyoruz.
  - Ancak config içindeki ENV_NAME i alıp .env file içinde "ENV_NAME = prod" olarak ekliyoruz.
  - Artık init.py dosyamız ENV_NAME i config den prod (burayı dev/prod olarak manuel değiştirebiliriz.) olarak alıyor, madem prod o zaman, "from .base import *"  bunu zaten almıştı bir de  "from .prod import *" bunu çalıştır diyoruz.



- Şimdi biz daha önce bir migrate yapmıştık ama o sqlite içindi. Artık yeni bir db ye bağladığımız için yine bir migrate yapacağız. 

- Şimdi bizim projemiz production modunda ve postgresql db ye bağlı biz artık migrate ile tablolarımızı postgresql db mizde oluşturabiliriz. 

```powershell
- py manage.py migrate
```

- Artık modellerimizin tablo karşılıklarını postgresql db de oluşturduk.

- Default olarak oluşturduğumuz projelerdeki global settings ayarlarını settings.py da tutuyorduk. Hatta buradaki bir ayarı app içerisinde değiştirebiliyoruz. settings.py da tüm ayarları bir arada tutmaktansa projemizin bulunacağı ortama göre ihtiyacımız olan ayarları parçalıyoruz.
- Developer a ne lazım? sqlite ta çalışması lazım, o zaman developer kullandığı zaman sqlite çalışsın/ayarlar buna göre olsun,  product ortamına alındığı zaman da postgresql e  bağlansın ve ayarlar product ortamına göre olsun.
- Tüm projede geçerli olmasını istediğimiz ayarlar base.py da, sonra ortam değişince değişmesini istediğimiz ayarlarımızı da yine mantıksal olarak ilişki kurduğumuz dosyaya aktardık.

- Development ortamında "DEBUG=True" olsun, product ortamında "DEBUG=False" olsun. Şu an onu yapmadık ama ekleyebiliriz. Şu paketi development ortamıda kullan, product ortamında kullanma gibi...

- product ortamında "DEBUG=False" olsun istiyoruz,
  - 1- dev.py da "DEBUG = config("DEBUG")" şeklinde .env den değil de; prod.py da direkt olarak "DEBUG=False", dev.py da da "DEBUG=True" diye ekleyebiliriz.
  - 2- .env de True yu False yapabiliriz.
  - 3- prod.py da zaten DEBUG = config("DEBUG") gibi bir değişken belirtmeyerek otomatik olarak DEBUG =False olacaktır. Ancak   

- Şu anda prod ortamı seçili ve de prod.py da "DEBUG=False" diye bir şey belirlemedik, base.py da da "DEBUG=True" yu yoruma/sildik, böylelikle otomatik olarak prod ortamında "DEBUG=False" olur. Çünkü bir değer tanımlamadık. Ancak "DEBUG=False" olduğu zaman base.py daki "ALLOWED_HOSTS = []" u bu şekilde bırakamayız, ya "ALLOWED_HOSTS = ["127.0.0.1"]" yazmalıyız ya da uğraşmayayım onunla diyip "ALLOWED_HOSTS = ["*"]" yazacaksınız.

- .env den dev ortamına geçtik, bizden tekrar migrate istedi, ve de tekrar superuser istedi.

```powershell
- py manage.py migrate
- py manage.py createsuperuser
```

### Logging

- logging işlemi restframework ile beraber geliyor olması lazım.

- Python programcıları, kodlarında hızlı ve kullanışlı bir hata ayıklama aracı olarak genellikle print() öğesini kullanır. 
- logging framework (günlük kaydı çerçevesi) kullanmak  bundan biraz daha fazla çaba gerektirir, ancak çok daha zarif ve esnektir. 
- Günlüğe kaydetme, hata ayıklama için yararlı olmanın yanı sıra uygulamanızın durumu ve sağlığı hakkında size daha fazla - ve daha iyi yapılandırılmış - bilgi de sağlayabilir.
- Django, sistem günlük kaydı gerçekleştirmek için Python'un yerleşik günlük modülünü kullanır ve genişletir. 
- Bu modül Python'un kendi belgelerinde ayrıntılı olarak ele alınmıştır; bu bölüm hızlı bir genel bakış sunar.

- Dokümandaki LOGGING kodunu alıp base.py da en sona ekliyoruz.
setting/base.py
```py

LOGGING = { 
    "version": 1, 
    # is set to True then all loggers from the default configuration will be disabled. 
    "disable_existing_loggers": True, 
    # Formatters describe the exact format of that text of a log record.  
    "formatters": { 
        "standard": { 
            "format": "[%(levelname)s] %(asctime)s %(name)s: %(message)s" 
        }, 
        'verbose': { 
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}', 
            'style': '{', 
        }, 
        'simple': { 
            'format': '{levelname} {message}', 
            'style': '{', 
        }, 
    }, 
    # The handler is the engine that determines what happens to each message in a logger. 
    # It describes a particular logging behavior, such as writing a message to the screen,  
    # to a file, or to a network socket. 
    "handlers": { 
        "console": { 
            "class": "logging.StreamHandler", 
            "formatter": "standard", 
            "level": "INFO", 
            "stream": "ext://sys.stdout", 
            }, 
        'file': { 
            'class': 'logging.FileHandler', 
            "formatter": "verbose", 
            'filename': './debug.log', 
            'level': 'INFO', 
        }, 
    }, 
    # A logger is the entry point into the logging system. 
    "loggers": { 
        "django": { 
            "handlers": ["console", 'file'], 
            # log level describes the severity of the messages that the logger will handle.  
            # "level": config("DJANGO_LOG_LEVEL", "INFO"), # .env ye alıyoruz.
            "level": config("DJANGO_LOG_LEVEL"), 
            'propagate': True, 
            # If False, this means that log messages written to django.request  
            # will not be handled by the django logger. 
        }, 
    }, 
} 
```

- Logging ayarlarına baktığımızda baktığımızda formatters, handlers, loggers bölümleri var. linkteki dokümantasyonda mevcut.
  https://docs.djangoproject.com/en/4.0/topics/logging/#logging

- loggers kısmının;
  - level kısmında -> Hangi seviyede log tutacak isek onu belirtiyoruz -> INFO (INFO, DEBUG, WARNING, ERROR, CRITICAL) 
  
  - handlers (topluyor, tutuyor) kısmında -> console, file belirtilmiş. Bunlar da hemen üstte detayları var;
        console; stream çıktıyı düzenli bir akış ile terminale console a veriyor.
            level; INFO
            formatter; hemen biraz üstte standart, verbose, simple olarak belirtilmiş olanlardan standart seçili.
        file; bulunduğu yerdeki ./debug.log isimli bir dosyaya kaydediyor.
            level; INFO
            formatter; hemen biraz üstte belirtilmiş olanlardan verbose seçili.


- level ile -> neleri loglayacak, bunu belirtiyoruz, 
- formatter ile -> loglanan şeylerin ne kadar detayını alacak bunu belirtiyoruz, 
  
- çalıştıryoruz; 
- debug.log isimli bir dosya oluşturukmuş olduğunu ve logging de belirttiğimiz kayıtların olduğunu gördük.

- log işlemi yapıyorsa, her yaptığımız işlemde özellikle INFO belirttiysek, backend de gerçekleşen her işlemi bir yere yazması lazım. Eğer WARNING yazarsak, dosyaya sadece WARNING leri kaydeder. vb.


- "formatters" , "handlers" ,  "loggers" diye yapılarımız var.

- hangi seviyede tutacağım? -> "level": config("DJANGO_LOG_LEVEL", "INFO"),

- neleri tutacağım/handle edeceğim? -> "handlers": ["console", 'file'],
  
- handle ediyorsam nerede saklayacağım? -> 'filename': './debug.log',


- logların level ını .env de tutacağız. Kurarken level olarak 'INFO' belirledik tamam, ama bu ayarı .env de tutarak oradan değiştireceğiz.
    "level": config("DJANGO_LOG_LEVEL") yazıp,
    .env de de -> 
        DJANGO_LOG_LEVEL=INFO  olarak kaydederek log level seçeneğini .env ye taşımış oluruz.

- "level": config("DJANGO_LOG_LEVEL", "INFO"), # .env ye alıyoruz.
  "level": config("DJANGO_LOG_LEVEL"), 




- Herhangi bir hatada veya server ın nasıl çalıştığını izleyebilmek için, bu logları tutuyoruz. Bu loglardaki trafiği takip ederek bazı kararlar alabiliyoruz. Mesela bir endpoint e bir çok istek var, demekki birisi birşeyler deniyor.
- Burada önemli olan husus logları hangi seviyede tutacağız?, neleri handle edeceğiz?, handle ediyorsak nerede saklayacağız? bunları belirtiyoruz.

#### BRD -> business requirement document;
  - Projenin tüm isterlerini, özelliklerini, gereklerini yazdığımız doküman. Projedeki ana kaynak. 
  - Projeye başlamadan önce dizayn kısmında, tasarım kısmında hazırlanır.
  - Software Development Life Cycle (SDLC) İlk aşama gerekliliklerin toplanması, projenin dizayn edilmesi, kurgulanması, isterlerin biraraya getirilmesi.
  - Poje ile ilgili herşey yazar, projenin imkan/özellikleri, hangi tip kullanıcılar olacak?, normal user/admin/başka roller, her kullanıcı rolü için yetkiler, bu dokümanda bulunur.
  
- Projeye başlanacağı zaman Agile kavramından hatırlarsanız proje isterlerinin hepsi task olarak product backlog lara alınıyor. Daha sonra sprintler halinde sprint backlog lara gerekli task lar halinde atılıyor, ve ilgili developer lara assign ediliyor. Böyle devam ediliyor.
- Bir projeye başlarken bu kısımlara zaman ayırıp en ince detayına kadar planlanıp, belirlenmesi gerekiyor.

#### ERD -> Entity Relationship Diagram
  - db tablolarını şekillendirmek, bir diyagrama dökmek. 

#### Swagger -> Document + Test (swagger->ortam, redoc->dokuman)
  - çok kullanılan bir tool, yazdığımız backend api, bunu frontend de bizim takım arkadaşlarımız da kullanabilir, veya dışarıya da servis edebiliriz. Bunun için yani servisin nasıl kullanılacağına dair bir klavuz hazırlanması gerekir.
  - end pointler için doküman oluşturur
  - swagger->ortam, redoc->document
  - Nasıl ekleniyor? ->
    - pip install drf-yasg şeklinde install ettikten sonra, drf_yasg ile third party package ile ekledik,
    - main urls.py da schema_view ve endpointlerini ekledik.

#### Debug_Toolbar -> Documentation 
- Development aşamasında projemizi geliştirirken bize ekstra kabiliyetler kazandırıyor.
- Nasıl ekleniyor? ->
    - pip install django-debug-toolbar şeklinde install ettikten sonra, drf_toolbar ile third party package ile ekledik, daha sonra dev settings ayarlarına aldık.
    - sonra MIDDLEWARE e en üst kısma "debug_toolbar.middleware.DebugToolbarMiddleware" ekledik, daha sonra dev settings ayarlarına aldık.
    - sonra INTERNAL_IPS a da ayarını ekledik, daha sonra dev settings ayarlarına aldık.
    - urls.py da bir de debug_toolbar ın   path('__debug__/', include('debug_toolbar.urls')),     şeklinde bir endpointi var, onu ekliyoruz.
    - En son migrate komutunu çalıştırdık.

#### Logger -> işlem geçmişimizi tutmaya yarayan kullanışlı bir tool.
- settings.py da en alta ekledik, ardından base.py dosyasına aktardık.
- Bu bir şablon şeklinde, tekrardan yazmıyoruz, bu şekilde kullanıyoruz. 
  - En alttan başlıyoruz bakmaya,  
  - loggers da handlers kısmında loglarımızı hem console a yazdır, hem de file oluştur demişiz.
  
  - level kısmına INFO demişiz, hangi level de tutabileceğini belirleyebiliyoruz.
  ( DEBUG: Low level system information for debugging purposes
    INFO: General system information
    WARNING: Information describing a minor problem that has occurred.
    ERROR: Information describing a major problem that has occurred.
    CRITICAL: Information describing a critical problem that has occurred.)

  - belirttiğimiz console ve file ayarlarını da hemen üstte handlers ta configüre ediyoruz, console a yazarken ki ayarlar, file a yazarkenki ayarlar.. Dikkat edilmesi gereken formatter kısmı -> 
    - console da standart, 
    - file da verbose demişiz.
  
  - formatter a da bir yukarıda bakıyoruz, standart, verbose, simple ile yani logların hangi formatta nasıl tutulacağını burada belirliyoruz.
  - belirlediğimiz formatları aşağıda handlers kısmında seçebiliyoruz, handlers ları da bir alttaki loggers kısımda seçebiliyoruz, ikisini de seçmişiz ayrıca  logger seviyesini de level ile seçebiliyoruz.(level ı config ile .env ye aktarabiliriz. Buradan env ortamını değiştirebildiğimiz gibi debug level ını da değiştirebiliriz.)


#### postgreSQL
- pgAdmin de bir db oluşturup, 
- oluşturduğumuz db ayarlarını prod.py kısmına yazıyoruz,
- gizlenmesi gereken verileri config ile .env ye aktarıyoruz.


- base(ortak ayarlar), dev(development ayarları), prod(production ayarları)



- Şimdi bunu taslak olarak kullanıp bu taslak üzerine projeler oluşturabiliriz.
- Bunu repomuza push layıp, 
- repomuza pushladık, repoya girdik, settings, template repository seç, -> yeni bir repository dediğimiz zaman yeni oluşturacağımız repoyu bu taslağa alır ve o şekilde oluşturur. Yani bu taslak üzerinden devam edebilirsiniz, tekrar tekrar paket kurmanıza gerek kalmaz.


## Proje_Taslak V.02

```powershell
- py -m venv env
- .\env\Scripts\activate
- pip install -r requirements.txt
```

- create .env and .gitignore (hidden to SECRET_KEY)
  
- .env file ı için dev environment settings ayarları için gerekli değişkenlerimizi yazıyoruz.

.env ->

```
SECRET_KEY = owz8%9)kl-dv7fm)8*$s=m!4frehvy+ke+uw$x2p$+3glpk3pj
DJANGO_LOG_LEVEL = INFO
DEBUG = True
ENV_NAME = dev
```

```powershell
- py manage.py migrate
- py manage.py runserver
- py manage.py createsuperuser
```

- Taslak bir django_api projesi clone layarak ayağa kaldırdık.

#### BRD -> business requirement document;
  - Projenin tüm isterlerini, özelliklerini, gereklerini yazdığımız doküman. Projedeki ana kaynak. 
  - Projeye başlamadan önce dizayn kısmında, tasarım kısmında hazırlanır.
  - Software Development Life Cycle (SDLC) İlk aşama gerekliliklerin toplanması, projenin dizayn edilmesi, kurgulanması, isterlerin biraraya getirilmesi.
  - Poje ile ilgili herşey yazar, projenin imkan/özellikleri, hangi tip kullanıcılar olacak?, normal user/admin/başka roller, her kullanıcı rolü için yetkiler, bu dokümanda bulunur.
  
- Projeye başlanacağı zaman Agile kavramından hatırlarsanız proje isterlerinin hepsi task olarak product backlog lara alınıyor. Daha sonra sprintler halinde sprint backlog lara gerekli task lar halinde atılıyor, ve ilgili developer lara assign ediliyor. Böyle devam ediliyor.
- Bir projeye başlarken bu kısımlara zaman ayırıp en ince detayına kadar planlanıp, belirlenmesi gerekiyor.

#### ERD -> Entity Relationship Diagram
  - db tablolarını şekillendirmek, bir diyagrama dökmek. 

#### Swagger -> Document + Test (swagger->ortam, redoc->dokuman)
  - çok kullanılan bir tool, yazdığımız backend api, bunu frontend de bizim takım arkadaşlarımız da kullanabilir, veya dışarıya da servis edebiliriz. Bunun için yani servisin nasıl kullanılacağına dair bir klavuz hazırlanması gerekir.
  - end pointler için doküman oluşturur
  - swagger->ortam, redoc->document
  - Nasıl ekleniyor? ->
    - pip install drf-yasg şeklinde install ettikten sonra, drf_yasg ile third party package ile ekledik,
    - main urls.py da schema_view ve endpointlerini ekledik.

#### Debug_Toolbar -> Documentation 
- Development aşamasında projemizi geliştirirken bize ekstra kabiliyetler kazandırıyor.
- Nasıl ekleniyor? ->
    - pip install django-debug-toolbar şeklinde install ettikten sonra, drf_toolbar ile third party package ile ekledik, daha sonra dev settings ayarlarına aldık.
    - sonra MIDDLEWARE e en üst kısma "debug_toolbar.middleware.DebugToolbarMiddleware" ekledik, daha sonra dev settings ayarlarına aldık.
    - sonra INTERNAL_IPS a da ayarını ekledik, daha sonra dev settings ayarlarına aldık.
    - urls.py da bir de debug_toolbar ın   path('__debug__/', include('debug_toolbar.urls')),     şeklinde bir endpointi var, onu ekliyoruz.
    - En son migrate komutunu çalıştırdık.

#### Logger -> işlem geçmişimizi tutmaya yarayan kullanışlı bir tool.
- settings.py da en alta ekledik, ardından base.py dosyasına aktardık.
- Bu bir şablon şeklinde, tekrardan yazmıyoruz, bu şekilde kullanıyoruz. 
  - En alttan başlıyoruz bakmaya,  
  - loggers da handlers kısmında loglarımızı hem console a yazdır, hem de file oluştur demişiz.
  
  - level kısmına INFO demişiz, hangi level de tutabileceğini belirleyebiliyoruz.
  ( DEBUG: Low level system information for debugging purposes
    INFO: General system information
    WARNING: Information describing a minor problem that has occurred.
    ERROR: Information describing a major problem that has occurred.
    CRITICAL: Information describing a critical problem that has occurred.)

  - belirttiğimiz console ve file ayarlarını da hemen üstte handlers ta configüre ediyoruz, console a yazarken ki ayarlar, file a yazarkenki ayarlar.. Dikkat edilmesi gereken formatter kısmı -> 
    - console da standart, 
    - file da verbose demişiz.
  
  - formatter a da bir yukarıda bakıyoruz, standart, verbose, simple ile yani logların hangi formatta nasıl tutulacağını burada belirliyoruz.
  - belirlediğimiz formatları aşağıda handlers kısmında seçebiliyoruz, handlers ları da bir alttaki loggers kısımda seçebiliyoruz, ikisini de seçmişiz ayrıca  logger seviyesini de level ile seçebiliyoruz.(level ı config ile .env ye aktarabiliriz. Buradan env ortamını değiştirebildiğimiz gibi debug level ını da değiştirebiliriz.)


#### postgreSQL
- pgAdmin de bir db oluşturup, 
- oluşturduğumuz db ayarlarını prod.py kısmına yazıyoruz,
- gizlenmesi gereken verileri config ile .env ye aktarıyoruz.


- base(ortak ayarlar), dev(development ayarları), prod(production ayarları)


//////////////////////////////////////////////////////

- Css ler gelmez ise! ->
```powershell
- python manage.py collectstatic
```

### Authentication işlemleri -> 

- login, logout, register
- TokenAuthentication kullanacağız

- Daha önceki derste login logout işlemlerini rest_framework.authtoken kullanmıştık,  view için obtain_auth_token kullanmıştık,

#### 3rd party package -> dj-rest-auth 

##### login, logout

- Ancak burada login ve logout işlemleri için third party packages lardan biri olan, oldukça yaygın bir şekilde kullanılan django-rest-auth / dj-rest-auth paketini kullanacağız.
  
- registration özelliği de sunuyor fakat allauth paketini de yüklemek gerekiyor. Biz registration ı kendimiz yazacağız.

- Bu paket çok kapsamlı bir paket, çok fazla özellikleri var. Dokümanına gidiyoruz, 
  
  https://dj-rest-auth.readthedocs.io/en/latest/index.html

- registration için allauth paketi kurmamızı istiyor demiştik, biz kurmayıp registration ı kendimiz yazacağız.

- eğer allauth u yüklersek social authentication imkanı da var. neler bunlar? facebook, google, twitter, github, bunlarla nasıl authentication işlemleri yapılacağını buradan inceleyebilirsiniz. 
- biz django-rest-auth paketinin sadece login, logout özelliğini kullanacağız.

- installation ->
- django-rest-auth paketini yüklüyoruz;

```powershell
- pip install dj-rest-auth
- pip freeze > requirements.txt
```

- settings klasörü içindeki settings.py (base.py) da INSTALLED_APPS kısmına "rest_framework.authtoken" paketi ile birlikte "dj_rest_auth" paketini ekliyoruz. 

settings/base.py (settings.py) ->
```py
INSTALLED_APPS = [
    .....
    'rest_framework.authtoken',
    'dj_rest_auth',
]
```

- Ayrıca TokenAuthentication kullanacağımız için base.py da en alt kısma ekliyoruz;
settings/base.py -> 
```py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

- bundan sonra paketimizin path ini ekleyeceğiz ve migrate yapacağız. Ancak path imizi, authentication işlemleri için oluşturacağımız "users" app in pathine ekleyeceğimiz için önce app imizi oluşturalım ve url configurastonunu yapalım;

#### authentication app 

```powershell
- py manage.py startapp users
```

- Add users app to INSTALLED_APPS in your django settings.py ->
settings/base.py ->
```py
INSTALLED_APPS = [
    'django.contrib.staticfiles',
    # my_app
    'users',
    # 3rd party packages 
]
```

- main/urls.py dan oluşturacağımız users/urls.py yönlendirme yapıyoruz,
main/urls.py ->
```py
urlpatterns = [
    path("users/", include("users.urls")),
]
```

- create urls.py in users app

```py
from django.urls import path
  
urlpatterns = [ 
    path("", ),
]
```

- Şimdi login, logout için kullanacağımız dj-rest-auth paketimizin pathini users app imizin urls.py ına ekliyoruz.

users/urls.py ->
```py
urlpatterns = [
    path('auth/', include('dj_rest_auth.urls')),
]
```

- Biz dokümanda belirtilen paketin path ismini uzun bulduk ve auth/ olarak kısaltıyoruz.

- Bu endpoint bize paketin birtakım özelliklerini sunacak.

- Son olarak migrate istiyor db miz için

```powershell
- py manage.py migrate
```

- Browser a gidip endpointimizle (http://localhost:8000/users/auth/) istek attığımızda bize sağladığı endpointleri de görüyoruz;
(
users/ auth/ password/reset/ [name='rest_password_reset']
users/ auth/ password/reset/confirm/ [name='rest_password_reset_confirm']
users/ auth/ login/ [name='rest_login']
users/ auth/ logout/ [name='rest_logout']
users/ auth/ user/ [name='rest_user_details']
users/ auth/ password/change/ [name='rest_password_change']
)

- Biz buradan sadece login ve logout kullanacağız. Paketin özelliklerinden olan  "password/change/", "password/reset/" için frontend ile birlikte yapılması gereken özellikler olduğu için sadece backend kısmı ile gösterildiğinde havada kalabiliyor. 

- paketin dokümanındaki, frontend i react, backend i django ile yazılmış demo projeyi repodan indirip dokümandak adımları izleyerek kurup, bu paket ile yapılmış özellikleri (password reset, password change) inceleyebilirsiniz.
  https://dj-rest-auth.readthedocs.io/en/latest/demo.html

- Burada şimdi login ve logout işlemlerini paketten kullanacağız.

- Paketimizi kurduk, ayarlarını yaptık;
   http://localhost:8000/users/auth/login/ 
  endpointine istek attığımızda, paketin login özelliği olarak bize bir login page sunuyor. 

- Biz zaten admin ile login olmuş olduğumuz için username ve password dolu bir şekilde bize bir form dönüyor, 

- Dokümandan bakıyoruz, API endpoints kısmından,
  https://dj-rest-auth.readthedocs.io/en/latest/api_endpoints.html
  Biz username ve password yazıp form ile POST isteği attımızda bize token dönüyor.

- Ayrıca admin panelden Token tablosunun create edildiğini ve token oluşturulduğunu gördük.

- Şimdi endpointine istek atalım;
  http://127.0.0.1:8000/users/auth/logout/
evet, gelen page de post methodu ile istek attığımızda  Succesfully logged out mesajı ile logout edildiğimizi gördük.
  
- admin panele gidip browser ımızı refresh ettiğimizde logout olduğumuzu görüyoruz. Tekrar login oluyoruz ve Token tablomuzda login olurken oluşturulmuş token ın hala durduğunu görüyoruz. Normalde biz postmanden istek atsaydık bu token ı silecekti ama browserdan istek attığımız için token ı silmiyor. Deneyelim bakalım postmanden logout için istek atalım; Evet logout/ yapınca token silindi, login/ yapınca token oluştu. Bu yüzden development aşamasında postman kullanıyoruz.

- Artık sadece third party package olan dj-rest-auth paketini yükleyip installation ayarlarını yaparak ve TokenAuthentication kullanarak bize sağladığı endpoint ile login ve logout yapmış olduk. Bizim için login olduğumuzda token üretiyor ve logout olduğumuzda token ı siliyor.

- Şimdi email ile login olmayı deneyelim;
- Normalde default olarak django bizden username ve password ile login olmamızı bekliyor. Ancak bu paketin sağladığı avantajlardan birisi email ve password ile de login olabilmek.
- Bu paket sayesinde çok hızlı bir şekilde login ve logout işlemleri yapabiliyoruz.

##### registration

- Registration için;
  1. bir endpoint yazacağız,
  2. Bu endpointin çalıştıracağı view i yazacağız,
  3. Yazdığımız view de kullanacağımız serializer ı yazacağız.

- Bu üç adım tüm django projelerinde kullandığımız registration saykılıdır. endpoint -> view > register.

###### 3. RegisterSerializer

- Önce register view i için kullanacağımız serializer ımızı, serializer.py file ı oluşturarak yazıyoruz.
- Bir register işlemi demek bir user ın create edilmesi, yeni bir user demektir.
- Bu serializer da ModelSerializer kullanacağız ve gelecek datamız User tablosuna create edilecek.
- Bu yüzden ModelSerializer ı inherit edeceğimiz "serializers" ı rest_framework dan, ve django.contrib.auth.models den "User" ı import ediyoruz.

users/serializers.py ->
```py 
from rest_framework import serializers
from django.contrib.auth.models import User

class RegisterSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = [
        'username',
        'first_name',
        'last_name',
        'email',
        'password',
        'password2'
        ]
``` 

- Burada dikkat etmemiz gereken 3 husus var;

  1. passwor2 ->
  - djangonun default User modelinde password2 diye bir field yok.
  - biz bu password2 ile password verisini match edip aynı olup olmadığının kontrolünü yani confirm validasyonu yaptıktan sonra data içerisinden password2 yi çıkarıp, kalan data ile user ı create edip user tablosuna ekleyeceğiz. Bunu bir sonraki adımda yapacağız.

  2. email ->
  - kullanıcıların email ile login olabilmeleri imkanını dj-rest-auth paketi ile sağlamıştık.
  - Ancak kullandığımız default User modelinde email hem uniqe değil hem de zorunlu alan değil.
  - biz burada email field ını hem zorunlu alan, hem de unique olarak yeniden tanımlayacağız.
    https://www.django-rest-framework.org/api-guide/validators/
  dokümanda belirtildiği şekliyle UniqueValidator ü kullanacağız.
  - önce rest_framework.validators den UniqueValidator ı import ediyoruz, zorunlu alan ve unique olacak ayarları yapıyoruz.
```py
from rest_framework.validators import UniqueValidator

    email = serializers.EmailField(
        required=True,
        validators=[UniqueValidator(queryset=User.objects.all())],
    )
```
  - UniqueValidator bizden queryset parametresi ile hangi tablo da kontrol yapacağımızı istiyor.

Not: Şuan yazdığımız RegisterSerializer ve birazdan yazacağımız RegisterView i diğer projelerimizde kalıp şeklinde direkt alıp kullanabileceğimiz gayet detaylı bir template.

  3. password ->
  - password fieldlarımızı da yine email field ında olduğu gibi yeniden tanımlıyoruz.
  - password -> 
    - write_only=True -> sadece register olurken, frontend den datayı gönderirken (Post) yaz, ama read ederken yani frontend e data gönderirken gösterme. (serializer çift taraflı çalışıyor, hem db ye frontend den gelen json datayı gönderiyor, hem de db den frontend e, json formatında data gönderiyor.) write_only -> register olurken password ü yaz, ama ben sana geri user datasını gönderirken password ü göndermiyeyim. 
  - required=True  -> zorunlu alan
  - validators=[validate_password]  -> djangonun kendi password validasyonu. Bunun için validate_password ü import etmemiz gerekiyor, 
        from django.contrib.auth.password_validation import validate_password
  - style = {"input_type" : "password"} -> password yazılırken karakterler açık açık değil de yıldızlı/kapalı görünsün. Sadece browsable api kısmında geçerli olan bir ayar. 
  - password2 yevalidator yazmadık, çünkü zaten bunların birbiri ile aynı olup olmadığının kontrolünü yapacağız.

```py
from django.contrib.auth.password_validation import validate_password

    password = serializers.CharField(
        write_only = True,
        required = True,
        validators = [validate_password],
        style = {"input_type" : "password"}
    )
    password2 = serializers.CharField(
        write_only = True,
        required = True,
        style = {"input_type" : "password"}
    )
```

- Bir de bazı projelerde görülebilir, açık açık belirtilmeyen fieldlara extra özellikler de eklenebiliyor, (biz burada password ü açık olarak belirttiğimiz içn gerek yok.) örnek:
```py
        # extra_kwargs ={
        #     "password" : {"write_only" : True},
        #     "password2" : {"write_only" : True},
        # }
```

- Şimdi gelelim başlarda belirttiğimiz password ve password2 ye;
  1. password ve password2 yi karşılaştırıp aynı olup olmadığının kontrolünü yapacağız,
  2. eğer aynı ise password2 yi datadan çıkarıp user ı create edeceğiz.

 https://www.django-rest-framework.org/api-guide/serializers/#validation

 - Burada field level ve object level validasyonlar var. Biz object level bir validasyon yapacağız. Çünkü iki tane field var işlem yapacağımız. object level, objedeki tüm datayı alıyor ve içinden istediğimiz fieldlar ile ilgili algoritma kuruyoruz.

```py
    def validate(self, data):
        if data["password"] != data["password2"]:
            raise serializers.ValidationError(
                {"message" : "Password fields didn't match!"}
            )
        return data
```

- Son olarak, bizim kullandığımız User tablosunda password2 isminde bir field yok, eğer bu hali ile create etmeye kalkarsak password2 olmadığı için hata verecektir.Bunun için password2 datasını bu veri setinden çıkartıp, kalan veriler arasındaki password verisini de hash lenmiş olarak, User tablosunda bir user create edeceğiz. 
- Bu işlemi de create() nu override ederek yapacağız.
- create() methodu self ve validate_data alıyor parametre olarak.
- validated_data dan get() methodu ile password ü alıp bir değişkene tanımlıyoruz ki hashleme işlemi yapacağız.
- password2 yi de pop() methodu ile validated_data dan çıkarıyoruz.
- kalan data ile user create ediyoruz, **validated_data -> ilgili fieldları tek tek eşleştirip map ediyor,
- set_password() methodu ile, create ettiğimiz user objesinin password ünü ik satır yukarıdaki password değişkenine atadığımız veri ile hashlenmiş şekilde güncelliyoruz.
- user ı kaydedip,
- return user ile artık db ye create edilmiş user ı, password ü hashlenmiş, password2 verisi çıkarılmış bir şekilde kaydediyoruz. 

```py
    def create(self, validated_data):
        password = validated_data.get("password")
        validated_data.pop("password2")
        user = User.objects.create(**validated_data)
        user.set_password(password)
        user.save()
        return user
```

- RegisterSerializer ımız son halini aldı. Artık diğer projelerde de kullanılabilecek bir RegisterSerializer ımız var.

user/serializer.py ->
```py
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.validators import UniqueValidator
from django.contrib.auth.password_validation import validate_password

class RegisterSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(
        required=True,
        validators=[UniqueValidator(queryset=User.objects.all())],
    )
    password = serializers.CharField(
        write_only = True,
        required = True,
        validators = [validate_password],
        style = {"input_type" : "password"}
    )
    password2 = serializers.CharField(
        write_only = True,
        required = True,
        style = {"input_type" : "password"}
    )

    class Meta:
        model = User
        fields = [
        'username',
        'first_name',
        'last_name',
        'email',
        'password',
        'password2'
        ]
        
        # extra_kwargs ={
        #     "password" : {"write_only" : True},
        #     "password2" : {"write_only" : True},
        # }
    
    def validate(self, data):
        if data["password"] != data["password2"]:
            raise serializers.ValidationError(
                {"message" : "Password fields didn't match!"}
            )
        return data

    def create(self, validated_data):
        password = validated_data.get("password")
        validated_data.pop("password2")
        user = User.objects.create(**validated_data)
        user.set_password(password)
        user.save()
        return user

```

###### 2. RegisterAPI view

- Bu serializer ımızı kullanacağımız view imizi yazıyoruz. Bu view imize bir de endpoint yazacağız. Bu yazacağımız view imizin tek görevi register/create etmek. Yani sadece create yapacak, o yüzden ConcreteView lerden CreateAPIView kullanabiliriz.
- view.py a gidiyoruz;
  - rest_framework.generics den CreateAPIView i import ediyoruz, 
  - queryset field ında kullanacağımız User tablomuzu django.contrib.auth.models den import ediyoruz,
  - serializer_class fieldı için kullanacağımız RegisterSerializer ı yazdığımız serializers dan import ediyoruz,

user/views.py ->
```py
from rest_framework.generics import CreateAPIView
from django.contrib.auth.models import User
from .serializers import RegisterSerializer

class RegisterAPI(CreateAPIView):
    queryset = User.objects.all()
    serializer_class = RegisterSerializer
```

###### 1. RegisterAPI view inin endpointi

- users/urls.py a gidip, views.py dan RegisterAPI view imizi import ediyoruz, 
    from .views import RegisterAPI
- urlpatterns e path ini ekliyoruz.
    path('register/', RegisterAPI.as_view()),


users/urls.py ->
```py
from django.urls import path, include
from .views import RegisterAPI

urlpatterns = [
    path('auth/', include('dj_rest_auth.urls')),
    path('register/', RegisterAPI.as_view()),
]
```

- register işlemini 3 adımda yaptık.
- test edelim, önce browserdan http://127.0.0.1:8000/users/register/  validasyonlarıyla birlikte deneyelim;  evet başarılı, dönüşte bize password verisini göstermedi.



- Bundan sonra;

1. kullanıcı register olduğunda onun için bir token generate edeceğiz. signal ile token generate edeceğiz.

2. kullanıcı register olduğunda dönen register verisine token verisini de ekleyeceğiz ki kullanıcı register ile birlikte login de olmuş olsun. 

3. bir user login olduğunda sadece token key dönüyor. Ancak biz hem token key, hem de login olan user ın datasını döneceğiz. 

- Her yeni bir user register olduğunda User tablosunda create ediliyor ve biz bu user a Token tablosunda bir token generate etmek istiyoruz.

###### Signals

- User tablosunda bir user create edildiğine signals vasıtasıyla Token tablosunda bu usera ait bir token create edeceğiz.
- signal, bir işlem gerçekleştikten sonra veya önce başka bir işlemi tetiklemesi için kullanılıyor.
- Normalde bu signals models.py file ı içerisinde yapılıyor, ancak models.py file ı çok kalabalık olduğu için biz bu signals i farklı bir file da yapıyoruz.
- users/signals.py file ımızı oluşturuyoruz,
  - User tablomuzu import ediyoruz,
  from django.contrib.auth.models import User
  
  - User tablodunda, user create dildikten hemen "sonra" için "post_save" import ediyoruz,
  from django.db.models.signals import post_save

  - gönderilen sinyali yakalayacak "receiver" decorator ünü import ediyoruz, 
  from django.dispatch import receiver
  
  - bu reciever sinyali yakaladığında token ı create edeceğimz Token tablomuzu import ediyoruz.
  from rest_framework.authtoken.models import Token

  - @reciver decoratörü, (post_save -> birşey create edildikten sonra, sender=User -> gönderen User tablosu) bir olay gerçekleştikten sonra, gönderen kim? User
  
  - def create_Token(sender, instance=None, created=False, **kwargs) 
    - parametrelerin sırası, isimleri önemli. instance->register olan user, created=False -> default False ama bir user create edildikten sonra True ya dönüyor, **kwargs->arka planda çalıştırılacak argümanlar.
  
  - if created: # user create edilince True oluyor, yani if true ise:
        Token.objects.create(user=instance) # token tablosunda bir token create et, Token tablosunun user field ını register olan user a eşitliyoruz.

users/signals.py ->
```py
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

@receiver(post_save, sender=User)
def create_Token(sender, instance=None, created=False, **kwargs):
    if created:
        Token.objects.create(user=instance)
```

- Normalde signals lerin model içerisinde tanımlandığını söylemiştik. Eğer bu şekilde ayrı bir file da tanımlarsak, apps.py da bunu belirtmemiz gerekiyor. Şu şekilde belirtiyoruz; indentations lara dikkat.

apps.py ->
```py
def ready(self) -> None:
    import users.signals
```

- signals imizi oluşturduk. Artık User tablosunda bir user create edildikten sonra Token tablosunda da bu signals vasıtasıyla Token tablosunda create edilen user a ait bir token create ediliyor olacağız.

- test ediyoruz; çalıştı.

- Şimdi kullanıcı register olduğunda generate edilen token ı, dönen register verisine de ekleyeceğiz ki kullanıcı register ile birlikte login de olmuş olsun.
- Bu işlemi views.py da CreateAPIView den inherit ederek yazdığımız RegisterAPI view inde yapacağız. Normalde CreateAPIView source code unda create() methoduyla create ettikten sonra tekrar Response olarak serializer datasını dönüyor. Biz bu create() methodunu alıp, içine user ın token datasını da koyup o şekilde user datasını Response etmesini sağlayacağız. 
- self.prform_create(serializer)  -> sadece serializer.save() yaptığı için, onun yerine serializer.save() yazıyoruz.
- Response ve status kullandığı için onları da import ediyoruz.


```py
from rest_framework import status
from rest_framework.response import Response

def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        # self.perform_create(serializer)
        serializer.save()
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)
```

- OOP (object oriented programming) mantığını hatırlayalım; bir methodu, attribute ü kendi üstünde aruyor, bulamazsa parent a bakıyor. bizim bu (CreateAPIView den inherit ettiğimiz) view imiz create işlemi yapacağı zaman bakıyor, create() methodum var mı? varsa kendi üstündekini çalıştırıyor. Yoksa parent a gidip parent takini çalıştırıyor.

- parent tan kendi class ımıza çektik, kendimize göre override edeceğiz.

- create() methodunda serializer.save() diyince, serializer a gidiyoruz, serializer create() methodu çalışıyor ve bize user ı return ediyor. Yani serializer.save() bize aslında user ı return ediyor, biz de bunu bir user değişkenine tanımlayabiliriz. 
  user = serializer.save()

- Artık biz bu user ile Token tablosundan bu user ın token ını çekebilir ve bir değişkene tanımlayabiliriz. user create edildikten sonra token tablosunda bu user için bir de token create ediliyor signal ile.
- Önce Token tablosunu da import edelim;
  from rest_framework.authtoken.models import Token
  token = Token.objects.get(user=user) -> token tablosundaki user ı bizim hemen bir üst satırda tanımladığımız user a eşit olan user ı(token tablosunun bir object i) token değişkenine tanımla.

  serializer.data yı bir değişkene tanımlıyoruz; yani register olduktan sonra bize dönen data
  data = serializer.data

  bu data değişkenine; "token" isminde bir key tanımlıyoruz ve value suna da, yukarıdaki token objesinin key field ını tanımlıyoruz.;
  data["token"] = token.key 

  Artık response olarak serializer.data değil de, data yı dönüyoruz.

  headers kısmına takılmayın.

```py
from rest_framework import status
from rest_framework.response import Response

def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        # self.perform_create(serializer)
        user = serializer.save()
        token = Token.objects.get(user=user)
        data = serializer.data
        data["token"] = token.key
        headers = self.get_success_headers(serializer.data)
        return Response(data, status=status.HTTP_201_CREATED, headers=headers)
```

- test ediyoruz, çalıştı.

- Son kısım;
- Şu anda user login olduğunda sadece token key dönüyor. Ancak biz istiyoruz ki user login olduğunda user bilgilerinin de token key ile birlikte dönmesini sağlayalım. Kullanıcı datasını çekmek için tekrardan istek atmak zorunda olmasın.

- token, token tablosunda kayıtlı, biz db den veri çekip frontend e aktarırken serializer dan geçirip aktarıyoruz. Aynı bunun gibi token tablosundan token ı alıp frontend e aktarırken de arka tarafta bir serializer dan (TokenSerializer) geçiriliyor.

- O TokenSerializer ına ulaşıp içine user datasını da koyacağız.
  https://dj-rest-auth.readthedocs.io/en/latest/configuration.html#login-serializer 

- LOGIN_SERIALIZER biz views.LoginView kısmında da customize edebilirdik ama biz LoginSerializer ını customize edeceğiz.

- serializers.py a gidiyoruz;

- dj_rest_auth.serializers  dan TokenSerializer ı import ediyoruz,
    from dj_rest_auth.serializers import TokenSerializer

- source koduna gidiyoruz, çok basit, TokenModel kullanarak sadece key field ını dönüyor. işte biz buraya user datasını da koymak istiyoruz.

- Bu TokenSeriaizer class ını alıp, inherit ederek kendimiz bir class yazacağız. İnheritance neydi tüm özellikleri ile beraber aynısını almış olduk. Bunun üzerine kendimize göre bir iki değişiklik yapacağız.
  class CustomTokenSerializer(TokenSerializer):

- class Meta(TokenSerializer.Meta) -> class Meta sını almak için bu yapıyı kullanmamız gerekiyor. Meta sını da aldık,
  class Meta(TokenSerializer.Meta):

- Şimdi fields ını belirliyoruz. Bu fields içine user da koyacağız, onun  için user ı da tanımlıyoruz, Ancak daha öncesinde User modelini kullanarak bir ModelSerializer (UserTokenSerializer) yazacağız, ve o serializer ı user değişkenine tanımlayacağız, user değişkenine tanımladığımız bu user datasını da fields olarak CustomTokenSerializer a tanımlayacağız.
  
```py
class UserTokenSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = (
        'id',
        'first_name',
        'last_name',
        'email',
        )
```

```py
class CustomTokenSerializer(TokenSerializer):
    user = UserTokenSerializer(read_only = True)
    class Meta(TokenSerializer.Meta):
        fields = ("key", "user")
```

- Normal TokenSerializer ı aldık, bunu inherit ederek yeni bir serializer (CustomTokenSerializer) oluşturduk, TokenSerializer sadece "key" dönüyordu. Biz bunun class metasını tekrardan çektik ve "key" in yanına "user" koyduk . Ancak user field ımız yoktu, Bunun için User modelinden bir serializer yazdık ve onu user a tanımladık.

- Ancak biz şimdi TokenSerializer ı buraya çekip inherit ederek yeni bir serializer yazdık ama, pakete bu bizim yazdığımız CustomTokenSerializer serializer ı kullan dememiz lazım, yoksa o hala kendi TokenSerializer serializer ını kullanıyor olacaktır. Pakete buraya bir CustomTokenSerializer serializer yazdım, bunu kullan dememiz lazım. Bunu da ;
- settings/base.py içerisinde belirtiyoruz. Yani TOKEN_SERIALIZER olarak users app imin içindeki serializers içindeki CustomTokenSerializer ı kullan diyoruz. Bu da zaten paketin sağladığı serializer ı inherit ederek oluşturduğumuz bir serializer.

```py
REST_AUTH_SERIALIZERS = {
    'TOKEN_SERIALIZER': 'users.serializers.CustomTokenSerializer',
}
```

- şimdi test edelim, postmanden login olalım ve bize ne verisi dönecek bakalım;

- TOKEN_SERIALIZER ı bir türlü override edemedim! 
- token verisinin yanında user verisi dönmüyor!
- kendi TokenSerializer class ını çalıştırıyor!
- benim override ettiğim CustomTokenSerializer ı çalıştırmıyor!





