# API com Django 3: Validações, buscas, filtros e deploy

https://cursos.alura.com.br/course/api-django-3-validacoes-buscas-filtros-deploy

* What API versioning is
* Find out how to define different user permission levels
* Learn how to limit the number of resource requests
* Include extra information in the request headers
* Discover how to integrate a Django API with a React front-end.


* [Python 3.9.13](https://www.python.org/)
* [Django 4.1.5](https://www.djangoproject.com/)
* [Django Rest Framework 3.14.0](https://www.django-rest-framework.org/)

## How to run the project?

* clone this repository.
* Create a virtualenv with Python 3.
* Active virtualenv.
* install dependencies.
* Run migrations.
* Run seed.py to create fake data

```
git clone https://github.com/diegoamferreira/api_drf_escola.git
cd api_drf_escola
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Django
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

### Populate client tables with fake clients

run `python seed.py`

## Django Rest Framework

[Django REST framework](https://www.django-rest-framework.org/) is a powerful and flexible toolkit for building Web
APIs.

* The Web browsable API is a huge usability win for your developers.
* Authentication policies including packages for OAuth1a and OAuth2.
* Serialization that supports both ORM and non-ORM data sources.
* Customizable all the way down - just use regular function-based views if you don't need the more powerful features.
* Extensive documentation, and great community support.
* Used and trusted by internationally recognised companies including Mozilla, Red Hat, Heroku, and Eventbrite.

## What did I learn here??

___

# Versioning

API versioning allows you to alter behavior between different clients. REST framework provides for a number of different
versioning schemes.

* **AcceptHeaderVersioning**
    * This scheme requires the client to specify the version as part of the media type in the Accept header. The version
      is included as a media type parameter, that supplements the main media type.
      ```
      GET /bookings/ HTTP/1.1
        Host: example.com
        Accept: application/json; version=1.0
        ```
* **URLPathVersioning**
    * This scheme requires the client to specify the version as part of the URL path.
        ```
        GET /v1/bookings/ HTTP/1.1
        Host: example.com
        Accept: application/json
        urlpatterns = [
        re_path(
            r'^(?P<version>(v1|v2))/bookings/$',
            bookings_list,
            name='bookings-list'
        ),
        re_path(
            r'^(?P<version>(v1|v2))/bookings/(?P<pk>[0-9]+)/$',
            bookings_detail,
            name='bookings-detail'
        )
      ]
        ```
* **NamespaceVersioning**
    * To the client, this scheme is the same as URLPathVersioning. The only difference is how it is configured in your
      Django application, as it uses URL namespacing, instead of URL keyword arguments.
      ```
        GET /v1/something/ HTTP/1.1
        Host: example.com
        Accept: application/json
    
      # bookings/urls.py
      urlpatterns = [
          re_path(r'^$', bookings_list, name='bookings-list'),
          re_path(r'^(?P<pk>[0-9]+)/$', bookings_detail, name='bookings-detail')
      ]
      
      # urls.py
      urlpatterns = [
          re_path(r'^v1/bookings/', include('bookings.urls', namespace='v1')),
          re_path(r'^v2/bookings/', include('bookings.urls', namespace='v2'))
      ]
      ```

* **HostNameVersioning**
    * The hostname versioning scheme requires the client to specify the requested version as part of the hostname in the
      URL.
    * For example the following is an HTTP request to the http://v1.example.com/bookings/ URL:
    ```
        GET /bookings/ HTTP/1.1
        Host: v1.example.com
        Accept: application/json
    ```

* **QueryParameterVersioning**
    * This scheme is a simple style that includes the version as a query parameter in the URL. For example:
      ```
      GET /something/?version=0.1 HTTP/1.1
      Host: example.com
      Accept: application/json
      ```

___

Put in the settings.py the opted versioning config:

```
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.QueryParameterVersioning'
}
```

___

# Permissions

## Using DjangoModelPermissions

Passing the DjangoModelPermissions in permission_classes of DRF View we can use the default permissions seted on Admin

```
    permission_classes = [IsAuthenticated, DjangoModelPermissions]
```

___

### Defaults of DRF in settings.py

We can set versioning, permissions, authentication, etc. as default. With this, we don't need to put this information in
each individual view.

```
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.QueryParameterVersioning',
    'DEFAULT_PERMISSIONS_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
        'rest_framework.permissions.DjangoModelPermissions'
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication'
    ]
}
```

___

# Change methods allowed in ViewSets

We can use the attribute `http_method_names` of a ViewSet to change the allowed methods

```
class MatriculaViewSet(viewsets.ModelViewSet):
    """Listando todas as matrículas"""
    queryset = Matricula.objects.all()
    serializer_class = MatriculaSerializer
    http_method_names = ['get', 'post', 'put', 'patch']
```

___

# Limiting Requests (THROTTLING)

We can set limits on access to our API. We can define one limit for users and another limit for anonymous users. The
rate descriptions used in DEFAULT_THROTTLE_RATES may include second, minute, hour, or day as the throttle period.

```
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```

___

# [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)

The 201 response includes a location attribute with a URI that the client can use to GET the current state of that
resource in the future. The response here also includes a representation of that resource to save the client an extra
call right now.

```
class CursosViewSet(viewsets.ModelViewSet):
    """Exibindo todos os cursos"""
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.serializer_class(data=request.data)
        if serializer.is_valid():
            serializer.save()
            response = Response(serializer.data, status=status.HTTP_201_CREATED)
            id = str(serializer.data['id'])
            response['Location'] = request.build_absolute_uri() + id
            return response
```

___

# CORS

Cross-Origin Resource Sharing (CORS) is an HTTP-header based mechanism that allows a server to indicate any origins (
domain, scheme, or port) other than its own from which a browser should permit loading resources.

## django-cors-headers

With django-cors-headers we can set on settings.py the allowed origins:

```
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
]
```

___

# Upload Files

Like a Django we just need add an Image Field and then set it in the serializer

___

# i18n - Languages

Just add `django.middleware.locale.LocaleMiddleware` to the middlewares list to receive the `Accept-Language` header from
requests and automatically respond with the language specified in the header.

___
# XML on DRF
We can add XML parser & render to response content.
```
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework_xml.parsers.XMLParser',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework_xml.renderers.XMLRenderer',
    ],
```

# Tests
Extend APITestCase to test, check methods, responses, object creation, etc.
```
    def setUp(self):
        self.list_url = reverse('Cursos-list')
        self.curso_1 = Curso.objects.create(
            codigo_curso='CTT1', descricao='Curso teste 1', nivel='B'
        )
        self.curso_2 = Curso.objects.create(
            codigo_curso='CTT2', descricao='Curso teste 2', nivel='A'
        )
        
    def test_requisicao_get_para_listar_cursos(self):
        """Teste para verificar a requisição GET para listar os cursos"""
        response = self.client.get(self.list_url)
        self.assertEquals(response.status_code, status.HTTP_200_OK)
```
