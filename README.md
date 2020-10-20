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

- É criada um conjunto de objetos, **Article** que possuem um headline e uma data de publicação e **Autores** que possuem um nome, que serão usados nos testes que vem a seguir.

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
- Verifica a ordenação padrão. A ordenação por padrão dos objetos da classe **Article** é pela data de publicação e depois pela headline em ordem lexicográfica. Então verifica-se se o primeiro da lista é o Article 4 (que possui data de publicação mais recente), e assim por diante.

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

    - O segundo assert testa a sobrescrição da ordenação padrão e do "sentido" da ordenação, agora a ordem é por data de publicação "ascending" e depois por headline "descending"

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
- Verifica a sobrescrição de order_by's seguidos, somente o último order_by deve ter efeito

- O primeiro assert verifica a ordenação por id. No segundo teste, é feita uma chamada de ordenação por id, e depois uma chamada de ordenação por headline descendente, é possivel verificar que a ordenação por id não teve efeito.

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
- Verifica se a operação de slicing apenas com o a parte de 'stop' funciona

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

```python
    def test_random_ordering(self):
        """
        Use '?' to order randomly.
        """
        self.assertEqual(
            len(list(Article.objects.order_by("?"))), 4
        )
```
- Verifica se a ordenação aletória dos elementos usando order_by("?"), verificando se o tamanho da lista retornado é o igual ao tamanho da lista original (nesse caso 4).

