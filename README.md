# DJANGO

https://github.com/django/django


## Mateus Figueiredo

### TESTES DE UNIDADE

Códigos retirados de https://github.com/django/django/blob/master/tests/ordering/tests.py

```python
class OrderingTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.a1 = Article.objects.create(headline="Article 1", pub_date=datetime(2005, 7, 26))
        cls.a2 = Article.objects.create(headline="Article 2", pub_date=datetime(2005, 7, 27))
        cls.a3 = Article.objects.create(headline="Article 3", pub_date=datetime(2005, 7, 27))
        cls.a4 = Article.objects.create(headline="Article 4", pub_date=datetime(2005, 7, 28))
        cls.author_1 = Author.objects.create(name="Name 1")
        cls.author_2 = Author.objects.create(name="Name 2")
        for i in range(2):
            Author.objects.create()
```

- É criado um conjunto de objetos, **Article** que possuem um headline e uma data de publicação e **Autores** que possuem um nome, que serão usados nos testes que vem a seguir.

**1 -**
```python
    def test_default_ordering(self):
        """
        By default, Article.objects.all() orders by pub_date descending, then
        headline ascending.
        """
        self.assertQuerysetEqual(
            Article.objects.all(), [
                "Article 4",
                "Article 2",
                "Article 3",
                "Article 1",
            ],
            attrgetter("headline")
        )

        # Getting a single item should work too:
        self.assertEqual(Article.objects.all()[0], self.a4)
```
- Verifica a ordenação padrão. A ordenação por padrão dos objetos da classe **Article** é pela data de publicação e depois pelo headline em ordem lexicográfica. Então, verifica-se se o primeiro da lista é o Article 4 (que possui data de publicação mais recente), e assim por diante.

**2 -**
```python
    def test_default_ordering_override(self):
        """
        Override ordering with order_by, which is in the same format as the
        ordering attribute in models.
        """
        self.assertQuerysetEqual(
            Article.objects.order_by("headline"), [
                "Article 1",
                "Article 2",
                "Article 3",
                "Article 4",
            ],
            attrgetter("headline")
        )
        self.assertQuerysetEqual(
            Article.objects.order_by("pub_date", "-headline"), [
                "Article 1",
                "Article 3",
                "Article 2",
                "Article 4",
            ],
            attrgetter("headline")
        )
```
- Testes de sobrescrição do método de ordenção padrão

    - O primeiro assert sobrescreve a ordenação padrão verificando se os elementos estão, agora, na ordem lexicográfica ao invés de estarem ordem de publicação mais recente.

    - O segundo assert testa a sobrescrição da ordenação padrão e do "sentido" da ordenação, agora a ordem é por data de publicação crescente e depois por headline decrescente.

**3 -**
```python
    def test_order_by_override(self):
        """
        Only the last order_by has any effect (since they each override any
        previous ordering).
        """
        self.assertQuerysetEqual(
            Article.objects.order_by("id"), [
                "Article 1",
                "Article 2",
                "Article 3",
                "Article 4",
            ],
            attrgetter("headline")
        )
        self.assertQuerysetEqual(
            Article.objects.order_by("id").order_by("-headline"), [
                "Article 4",
                "Article 3",
                "Article 2",
                "Article 1",
            ],
            attrgetter("headline")
        )
```
- Verifica a sobrescrição de order_by's seguidos, somente o último order_by deve ter efeito.

- O primeiro assert verifica a ordenação por id. No segundo teste, é feita uma chamada de ordenação por id, e depois uma chamada de ordenação por headline decrescente, é possivel verificar que a ordenação por id não teve efeito.


**4 -**
```python
    def test_stop_slicing(self):
        """
        Use the 'stop' part of slicing notation to limit the results.
        """
        self.assertQuerysetEqual(
            Article.objects.order_by("headline")[:2], [
                "Article 1",
                "Article 2",
            ],
            attrgetter("headline")
        )
```
- Verifica se a operação de slicing apenas com o 'stop' funciona.

**5 -**
```python
    def test_stop_start_slicing(self):
        """
        Use the 'stop' and 'start' parts of slicing notation to offset the
        result list.
        """
        self.assertQuerysetEqual(
            Article.objects.order_by("headline")[1:3], [
                "Article 2",
                "Article 3",
            ],
            attrgetter("headline")
        )
```

- Similar ao teste anterior, mas agora verificando se ambas as operações de 'stop' e 'start' do slicing funcionam dando offset na lista de resultado.

**6 -**
```python
    def test_random_ordering(self):
        """
        Use '?' to order randomly.
        """
        self.assertEqual(
            len(list(Article.objects.order_by("?"))), 4
        )
```
- Verifica a ordenação aletória dos elementos usando order_by("?"), avaliando se o tamanho da lista retornado é o igual ao tamanho da lista original (nesse caso 4).

### TESTES DE UNIDADE COM MOCKS

Os mocks se tornaram padrão da biblioteca de testes de unidade (*unittest*) na versao 3.33 do python. A maioria dos mocks utilizados no código-fonte do Django faz uso da funcionalidade de patch().

O [patch()](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch) é um decorator que torna simples a substituição temporária de uma classe por um objeto mock.


**1 -**

Código retirado de: https://github.com/django/django/blob/61a0ba43cfd4ff66f51a9d73dcd8ed6f6a6d9915/tests/files/tests.py

Classe de testes relacionados à arquivos.
```python
def test_valid_image(self):
        """
        get_image_dimensions() should catch struct.error while feeding the PIL
        Image parser (#24544).
        Emulates the Parser feed error. Since the error is raised on every feed
        attempt, the resulting image size should be invalid: (None, None).
        """
        img_path = os.path.join(os.path.dirname(__file__), "test.png")
        with mock.patch('PIL.ImageFile.Parser.feed', side_effect=struct.error):
            with open(img_path, 'rb') as fh:
                size = images.get_image_dimensions(fh)
                self.assertEqual(size, (None, None))
```

O side_effect declarado dentro do patch é chamado sempre que o mock é chamado. 

- Nesse caso está sendo emulado um erro durante o parsing de um arquivo de imagem, assim o tamanho da imagem deve ser inválido (None, None). O erro vai ser gerado em todas as chamadas porque o side_effect do mock é igual a um erro de struct.

**2 -**

Código retirado de: https://github.com/django/django/blob/335c9c94acf263901fb023404408880245b0c4b4/tests/backends/mysql/test_creation.py

Classe de testes relacionados à criação de banco de dados MySQL.

```python
def patch_test_db_creation(self, execute_create_test_db):
        return mock.patch.object(BaseDatabaseCreation, '_execute_create_test_db', execute_create_test_db)

    @mock.patch('sys.stdout', new_callable=StringIO)
    @mock.patch('sys.stderr', new_callable=StringIO)
    def test_create_test_db_database_exists(self, *mocked_objects):
        # Simulate test database creation raising "database exists"
        creation = DatabaseCreation(connection)
        with self.patch_test_db_creation(self._execute_raise_database_exists):
            with mock.patch('builtins.input', return_value='no'):
                with self.assertRaises(SystemExit):
                    # SystemExit is raised if the user answers "no" to the
                    # prompt asking if it's okay to delete the test database.
                    creation._create_test_db(verbosity=0, autoclobber=False, keepdb=False)
            # "Database exists" shouldn't appear when keepdb is on
            creation._create_test_db(verbosity=0, autoclobber=False, keepdb=True)
```

A primeira função *patch_test_db_creation*, define a criação de um banco de dados mock que será utilizado nos próximos testes.

- É criado um mock para simular a resposta de um usuário, nesse caso a resposta seria, de acordo com o return_value definido, 'no'. No comportamento padrão, ao responder 'no' no prompt durante a remoção de um banco dados deve ser levantada uma exceção de 'SystemExit'.

**3 -**

Código retirado de: https://github.com/django/django/blob/3b8857b51ab4ab2ef19bcfc665a427e19a7aabc0/tests/backends/sqlite/tests.py

Classe de testes relacionados ao SQLite.

```python
def test_check_sqlite_version(self):
        msg = 'SQLite 3.8.3 or later is required (found 3.8.2).'
        with mock.patch.object(dbapi2, 'sqlite_version_info', (3, 8, 2)), \
                mock.patch.object(dbapi2, 'sqlite_version', '3.8.2'), \
                self.assertRaisesMessage(ImproperlyConfigured, msg):
            check_sqlite_version()
```
- É criado um banco sqlite mock com versão 3.8.2 (que não é suportada). Então verifica-se se a mensagem de erro de um banco mal configurado é levantada para o usuário.


## Álvaro Rodrigues

### TESTES DE UNIDADE

Códigos retirados de [tests/admin_registration/models.py](https://github.com/django/django/blob/master/tests/admin_registration/models.py), [tests/admin_registration/tests.py](https://github.com/django/django/blob/master/tests/admin_registration/tests.py) e [contrib/admin/sites.py](https://github.com/django/django/blob/master/django/contrib/admin/sites.py)

```python
# file models.py
class Person(models.Model):
    name = models.CharField(max_length=200)

class Traveler(Person):
    pass

class Location(models.Model):
    class Meta:
        abstract = True

class Place(Location):
    name = models.CharField(max_length=200)

class NameAdmin(admin.ModelAdmin):
    list_display = ['name']
    save_on_top = True

class CustomSite(admin.AdminSite):
    pass

# file tests.py
class TestRegistration(SimpleTestCase):
    def setUp(self):
        self.site = admin.AdminSite()

    # tests
    # ...
```
- São criados diferentes modelos para que possam ser registrados pelo admin. Em sequência é definido uma classe de testes para testar o registro.

**1 -**
```python
def test_bare_registration(self):
    self.site.register(Person)
    self.assertIsInstance(self.site._registry[Person], admin.ModelAdmin)
```
- Nesse teste nenhum modelo é passado como parâmetro durante o registro, então é verificado se **Person** é uma instância de **ModelAdmin**, uma vez que foi registrada por um admin e portanto também devera ser um admim.

**2 -**
```python
def test_registration_with_model_admin(self):
    self.site.register(Person, NameAdmin)
    self.assertIsInstance(self.site._registry[Person], NameAdmin)
```
- Nesse teste é passado um modelo como parâmetro durante o registro. **NameAdmin** é uma classe que extende a classe **admim.ModelAdmin** e portanto **Person** também será considerada um admin. O teste então verifca se **Person** é uma instância de **NameAdmin**.

**3 -**
```python
# file sites.py
class AlreadyRegistered(Exception):
    pass

def register(self, model_or_iterable, admin_class=None, **options):
    # ...
    for model in model_or_iterable:
        # ...

        if model in self._registry:
            registered_admin = str(self._registry[model])
            msg = 'The model %s is already registered ' % model.__name__
            if registered_admin.endswith('.ModelAdmin'):
                # Most likely registered without a ModelAdmin subclass.
                msg += 'in app %r.' % re.sub(r'\.ModelAdmin$', '', registered_admin)
            else:
                msg += 'with %r.' % registered_admin
            raise AlreadyRegistered(msg)
    # ...

# file tests.py
def test_prevent_double_registration(self):
    self.site.register(Person)
    msg = "The model Person is already registered in app 'admin_registration'."
    with self.assertRaisesMessage(admin.sites.AlreadyRegistered, msg):
        self.site.register(Person)
```
- Esse teste verifica se uma exceção será disparada ao tentar registrar uma instância previamente resgistrada. Nesse caso **Person** será registrada com o modelo **ModelAdmin** e portanto a condição do segundo **if** da função **register** será verdadeira. 

**4 -**
```python
def test_prevent_double_registration_for_custom_admin(self):
    class PersonAdmin(admin.ModelAdmin):
        pass

    self.site.register(Person, PersonAdmin)
    msg = "The model Person is already registered with 'admin_registration.PersonAdmin'."
    with self.assertRaisesMessage(admin.sites.AlreadyRegistered, msg):
        self.site.register(Person, PersonAdmin)
```
- Esse teste é similar ao anterior, entretanto é criado um modelo que extende o modelo **admin.ModelAdmin**, modelo esse que é usado durante o registro de **Person**. É verificado então, se ao registrar novamente **Person** utilizando o modelo customizado de admin se a exceção será disparada.

**5 -**
```python
# file sites.py
def register(self, model_or_iterable, admin_class=None, **options):
    # ...
    # If we got **options then dynamically construct a subclass of
    # admin_class with those **options.
    if options:

        options['__module__'] = __name__
        admin_class = type("%sAdmin" % model.__name__, (admin_class,), options)

    self._registry[model] = admin_class(model, self)

# file tests.py
def test_registration_with_star_star_options(self):
    self.site.register(Person, search_fields=['name'])
    self.assertEqual(self.site._registry[Person].search_fields, ['name'])
```
- Python e outras linguagens como Ruby e Scala tem suporte nativo a *keyword arguments*, que são argumentos extras para uma função. Em Pyhthon são precedidos por __**__ e comumente definidos como __**kwargs__. A função **register** possuir um *keyword argument* __**options__, e o teste em questão testa se esse argumento foi corretamente salvo após o regsitro.

**6 -**
```python
# file sites.py
def register(self, model_or_iterable, admin_class=None, **options):
    # ...
    for model in model_or_iterable:
        if model._meta.abstract:
            raise ImproperlyConfigured(
                'The model %s is abstract, so it cannot be registered with admin.' % model.__name__
            )
    # ...

# file models.py
class Location(models.Model):
    class Meta:
        abstract = True

# file tests.py
def test_abstract_model(self):
    msg = 'The model Location is abstract, so it cannot be registered with admin.'
    with self.assertRaisesMessage(ImproperlyConfigured, msg):
        self.site.register(Location)
```
- O modelo **Location** é um modelo abstrato e portanto não pode ser registrado como admin. Na função **register** essa condição é verificada e o teste em questão está testando se a exceção será disparada corretamente.

### TESTES DE UNIDADE COM MOCKS

Os testes a seguir irão testar funcionalidades da classe **ModelAdmin**, uma classe que encapsula todas as opções e funcionalidades de admin para um dado modelo. Os testes ultilzam mocks manuais implementados pelos próprios desenvolvedores do Django. Códigos retirados de [django/tests/modeladmin/tests.py](https://github.com/django/django/blob/master/tests/modeladmin/tests.py), [django/tests/modeladmin/models.py](https://github.com/django/django/blob/master/tests/modeladmin/models.py), [django/django/contrib/admin/utils.py](https://github.com/django/django/blob/master/django/contrib/admin/utils.py) e [django/django/contrib/admin/options.py](https://github.com/django/django/blob/master/django/contrib/admin/options.py).

```python
# file models.py
class Band(models.Model):
    name = models.CharField(max_length=100)
    bio = models.TextField()
    sign_date = models.DateField()

    class Meta:
        ordering = ('name',)

    def __str__(self):
        return self.name


class Song(models.Model):
    name = models.CharField(max_length=100)
    band = models.ForeignKey(Band, models.CASCADE)
    featuring = models.ManyToManyField(Band, related_name='featured')

    def __str__(self):
        return self.name


class Concert(models.Model):
    main_band = models.ForeignKey(Band, models.CASCADE, related_name='main_concerts')
    opening_band = models.ForeignKey(Band, models.CASCADE, related_name='opening_concerts', blank=True)
    day = models.CharField(max_length=3, choices=((1, 'Fri'), (2, 'Sat')))
    transport = models.CharField(max_length=100, choices=(
        (1, 'Plane'),
        (2, 'Train'),
        (3, 'Bus')
    ), blank=True)

# file tests.py
class MockRequest:
    pass

class MockSuperUser:
    def has_perm(self, perm, obj=None):
        return True

class ModelAdminTests(TestCase):

    @classmethod
    def setUpTestData(cls):
        cls.band = Band.objects.create(
            name='The Doors',
            bio='',
            sign_date=date(1965, 1, 1),
        )

    def setUp(self):
        self.site = AdminSite()

    # tests
    # ...
```
- Inicialmente é criado um mock que emula uma request de um usuário além de um mock que emula um usuário com todas as permissões. A classe de testes **ModelAdminTests** contém testes que utilizam modelos criados exclusivamente para os testes, definidos no arquivo [models.py](https://github.com/django/django/blob/master/tests/modeladmin/models.py).

**1 -**
```python
# file utils.py
def get_deleted_objects(objs, request, admin_site):
    # ...
    perms_needed = set()

    def format_callback(obj):
        # ...
        if has_admin:
            if not admin_site._registry[model].has_delete_permission(request, obj):
                perms_needed.add(opts.verbose_name)
    # ...

# file tests.py
def test_get_deleted_objects(self):
    mock_request = MockRequest()
    mock_request.user = User.objects.create_superuser(username='bob', email='bob@test.com', password='test')
    self.site.register(Band, ModelAdmin)
    ma = self.site._registry[Band]
    deletable_objects, model_count, perms_needed, protected = ma.get_deleted_objects([self.band], request)
    self.assertEqual(deletable_objects, ['Band: The Doors'])
    self.assertEqual(model_count, {'bands': 1})
    self.assertEqual(perms_needed, set())
    self.assertEqual(protected, [])
```
- O teste emula uma resquest usando o mock e em seguida seta um super usuário para a request. Em seguida é registrado um modelo **Band** que receberá as funcionalidades de um modelo **ModelAdmin**. É testado então se o método **get_deleted_objects**, atrelado ao objeto **Band**, retorna a saída correta em relação ao permissionamento do usuário. Como o usuário em questão é um super usuário então o objeto previamente definido de **Band** é deletável e nenhuma permissão a mais é necessária.

---

Os testes a seguir dizem respeito as permissões do **ModelAdmin**. Alguns mocks foram criados para simular diferentes permissões do usuário.

```python
class ModelAdminPermissionTests(SimpleTestCase):

    class MockUser:
        def has_module_perms(self, app_label):
            return app_label == 'modeladmin'

    class MockViewUser(MockUser):
        def has_perm(self, perm, obj=None):
            return perm == 'modeladmin.view_band'

    class MockAddUser(MockUser):
        def has_perm(self, perm, obj=None):
            return perm == 'modeladmin.add_band'

    class MockChangeUser(MockUser):
        def has_perm(self, perm, obj=None):
            return perm == 'modeladmin.change_band'

    class MockDeleteUser(MockUser):
        def has_perm(self, perm, obj=None):
            return perm == 'modeladmin.delete_band'

    # tests
    # ...
```

**2 -**
```python
# file options.py
def has_view_permission(self, request, obj=None):
    opts = self.opts
    codename_view = get_permission_codename('view', opts)
    codename_change = get_permission_codename('change', opts)
    return (
        request.user.has_perm('%s.%s' % (opts.app_label, codename_view)) or
        request.user.has_perm('%s.%s' % (opts.app_label, codename_change))
    )

# file tests.py
def test_has_view_permission(self):
    """
    has_view_permission() returns True for users who can view objects and
    False for users who can't.
    """
    ma = ModelAdmin(Band, AdminSite())
    request = MockRequest()
    request.user = self.MockViewUser()
    self.assertIs(ma.has_view_permission(request), True)
    request.user = self.MockAddUser()
    self.assertIs(ma.has_view_permission(request), False)
    request.user = self.MockChangeUser()
    self.assertIs(ma.has_view_permission(request), True)
    request.user = self.MockDeleteUser()
    self.assertIs(ma.has_view_permission(request), False)
```
- O teste acima é usado para verifcar se as permissões de visualização de um objeto estão corretamente definidas dependendo do usuário. Cada mock de usuário atribuido a request possui uma permissão diferente, e a função **has_view_permission** retorna True apenas para usuários que possuem permissões de **view** e **change**, e isso deve ser garantido nos asserts.

**3 -**
```python
# file options.py
def get_inline_instances(self, request, obj=None):
    inline_instances = []
    for inline_class in self.get_inlines(request, obj):
        inline = inline_class(self.model, self.admin_site)
        if request:
            if not (inline.has_view_or_change_permission(request, obj) or
                    inline.has_add_permission(request, obj) or
                    inline.has_delete_permission(request, obj)):
                continue
            if not inline.has_add_permission(request, obj):
                inline.max_num = 0
        inline_instances.append(inline)

    return inline_instances

# file tests.py
def test_inline_has_add_permission_uses_obj(self):
    class ConcertInline(TabularInline):
        model = Concert

        def has_add_permission(self, request, obj):
            return bool(obj)

    class BandAdmin(ModelAdmin):
        inlines = [ConcertInline]

    ma = BandAdmin(Band, AdminSite())
    request = MockRequest()
    request.user = self.MockAddUser()
    self.assertEqual(ma.get_inline_instances(request), [])
    band = Band(name='The Doors', bio='', sign_date=date(1965, 1, 1))
    inline_instances = ma.get_inline_instances(request, band)
    self.assertEqual(len(inline_instances), 1)
    self.assertIsInstance(inline_instances[0], ConcertInline)
```
- O teste em questão cria um modelo inline **ConcertInline** que extende a classe **TabularInline** ao invés de extender a classe **models.Model**. Em uma instância inline é possível setar o modelo a ser utilizado, **Concert** no caso, e na mesma classe já definir suas permissões. É nessesário testar se um usuário com permissões de **add** consegue adicionar modelos na instância. Quando nenhum objeto é passado ao método **get_inline_instances** ele então deve retornar lista vazia. Já quando um obeto é passado então é retornado uma lista de tamanho 1, e como o moodelo em questão é um **BandAdmin** que possui um inline definino como **ConcertInline**, então o último assert garante que a instância retornada na posição 0 da lista é um **ConcertInline**.