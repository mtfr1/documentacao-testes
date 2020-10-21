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

Nesse caso está sendo emulado um erro durante o parsing de um arquivo de imagem, assim o tamanho da imagem deve ser inválido (None, None). O erro vai ser gerado em todas as chamadas porque o side_effect do mock é igual a um erro de struct.

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

Em seguida é feito um mock para simular a resposta de um usuário, nesse caso a resposta seria, de acordo com o return_value definido, 'no'. No comportamento padrão, ao responder 'no' ao prompt durante a remoção de um banco dados deve levantar uma exceção de 'SystemExit'.

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
É criado um banco sqlite mock com versão = 3.8.2 (que não é suportada). Então verifica-se se a mensagem de erro de um banco mal configurado é levantada para o usuário.