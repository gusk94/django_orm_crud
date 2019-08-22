# Django ORM



## Create

### 기초설정

- Shell

  ```bash
  $ python manage.py shell
  ```

- Import model

  ```python
  from articles.models import Articel
  ```



### 데이터를 저장하는 3가지 방법

1. 첫번째 방식

   - ORM 을 쓰는 이유는? => DB를 조작하는 것을 객체지향 프로그래밍 (클래스)처럼 하기 위해서

     ```python
     # >>> Article.objects.all()
     # <QuerySet []>
     
     >>> article = Article()
     >>> article # <Article: Article object (None)>
     >>> article.title = 'First article'
     >>> article.content = 'Hello, article'
     >>> article.save()
     >>> article # <Article: Article object (1)>
     
     # >>> Article.objects.all()
     # <QuerySet [<Article: Article object (1)>]>
     ```

2. 두번째 방식

   - 함수에서 keyword 인자 넘기기 방식과 동일

     ```python
     >>> article = Article(title='Second article', content='hi, article')
     >>> article.save()
     >>> article 
     <Article: Article object (2)>
     ```

3. 세번째 방식

   - `create` 를 사용하면 쿼리셋 객체를 생성하고 저장하는 로직이 한번의 스텝

     ```python
     >>> Article.objects.create(title='Third article', content='Django!')
     <Article: Article object (3)>
     ```

4.  검증

   - `full_clean` 함수를 통해 저장하기 전 데이터 검증을 할 수 있다.

     ```python
     >>> article = Article()
     >>> article.title = 'Python'
     >>> article.full_clean()
     Traceback (most recent call last):
       File "<console>", line 1, in <module>
       File "C:\Users\student\development\django\django_orm_crud\venv\lib\site-packages\django\db\models\base.py", line 1203, in full_clean
         raise ValidationError(errors)
     django.core.exceptions.ValidationError: {'content': ['이 필드는 빈 칸으로 둘 수 없습니다.']}
     ```



## Read

- 객체 표현 변경

  ```python
  class Article(models.Model):
      ...
      def __str__(self):
          return f'{self.id}번 글 - {self.title} : {self.content}'
  
  >>> Article.objects.all()
  <QuerySet [<Article: 1번 글 - First article : Hello, article?>, <Article: 2번 글 - Second article : hi,
  article>, <Article: 3번 글 - Third article : Django!>, <Article: 4번 글 - title : >]>
  ```

- 모든 객체

  ```python
  >>> Article.objects.all()
  <QuerySet [<Article: Article object (1)>, <Article: Article object (2)>, <Article: Article object (3)>,
  <Article: Article object (4)>]>
  ```

- DB에 저장된 글 중에서 title 이 정해진 글만 가지고 오기

  ```python
  >>> Article.objects.filter(title='Second article')
  <QuerySet [<Article: 2번 글 - Second article : hi, article>]>
  # title이 Second article인 모든 글을 가지고 오므로 복수형인 QuerySet이 나옴
  
  >>> Article.objects.create(title='Second article', content='contect')
  <Article: 5번 글 - Second article : contect>
  >>> Article.objects.filter(title='Second article')
  <QuerySet [<Article: 2번 글 - Second article : hi, article>, <Article: 5번 글 - Second article : contect>]>
  ```

- DB에 저장된 글에서 title 이 Second article 인 글 중에서 첫번째만 가지고 오기

  ```python
  >>> querySet = Article.objects.filter(title='Second article')
  >>> querySet
  <QuerySet [<Article: 2번 글 - Second article : hi, article>, <Article: 5번 글 - Second article : contect>]>
  >>> querySet.first()
  <Article: 2번 글 - Second article : hi, article>
          
  >>> Article.objects.filter(title='Second article').first()
  <Article: 2번 글 - Second article : hi, article>
  ```

- DB에 저장된 글 중에서 pk(primary key = ID) 가 1인 글만 가지고 오기

  - PK 만 `get()` 으로 가지고 올 수 있다.

  ```python
  >>> Article.objects.get(pk=1)
  <Article: 1번 글 - First article : Hello, article?>
          
          
  >>> article = Article.objects.get(pk=1)
  >>> article.pk
  1
  >>> article.title
  'First article'
  ```

  - `get()` 에러

    ```python
    # 없는 id를 가져옴
    >>> Article.objects.get(pk=10)
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
      File "C:\Users\student\development\django\django_orm_crud\venv\lib\site-packages\django\db\models\manager.py", line 82, in manager_method
        return getattr(self.get_queryset(), name)(*args, **kwargs)
      File "C:\Users\student\development\django\django_orm_crud\venv\lib\site-packages\django\db\models\query.py", line 408, in get
        self.model._meta.object_name
    articles.models.Article.DoesNotExist: Article matching query does not exist.
    
    # get은 하나만 가져오기 때문에 여러개가 있을 경우 에러가 뜬다
    >>> Article.objects.get(title='Second article')
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
      File "C:\Users\student\development\django\django_orm_crud\venv\lib\site-packages\django\db\models\manager.py", line 82, in manager_method
        return getattr(self.get_queryset(), name)(*args, **kwargs)
      File "C:\Users\student\development\django\django_orm_crud\venv\lib\site-packages\django\db\models\query.py", line 412, in get
        (self.model._meta.object_name, num)
    articles.models.Article.MultipleObjectsReturned: get() returned more than one Article -- it returned 2!
        
    # filter의 경우 없을 때 빈 querySet 가져옴
    >>> Article.objects.filter(pk=10)
    <QuerySet []>
    ```

  - 오름차순

    ```python
    >>> articles = Article.objects.order_by('pk')
    >>> articles
    <QuerySet [<Article: 1번 글 - First article : Hello, article?>, <Article: 2번 글 - Second article : hi, article>, <Article: 3번 글 - Third article : Django!>, <Article: 4번 글 - title : >, <Article: 5번 글 - Second article : contect>]>
    ```

  - 내림차순

    ```python
    >>> articles = Article.objects.order_by('-pk')
    >>> articles
    <QuerySet [<Article: 5번 글 - Second article : contect>, <Article: 4번 글 - title : >, <Article: 3번 글 - Third article : Django!>, <Article: 2번 글 - Second article : hi, article>, <Article: 1번 글 - First article : Hello, article?>]>
    ```

  - index 접근

    ```python
    >>> articles = Article.objects.order_by('-pk')
    >>> article = articles[2]
    >>> article
    <Article: 3번 글 - Third article : Django!>
    >>> articles = Article.objects.all()[1:3]
    >>> articles
    <QuerySet [<Article: 2번 글 - Second article : hi, article>, <Article: 3번 글 - Third article : Django!>]>
    ```

  - LIKE  - 문자열을 포함하고 있는 값을 가지고 옴

    - django ORM 은 이름과 필터를 더블 언더스코어로 구분합니다.

    ```python
    >>> articles = Article.objects.filter(title__contains='Sec')
    >>> articles
    <QuerySet [<Article: 2번 글 - Second article : hi, article>, <Article: 5번 글 - Second article : contect>]>
    ```

  - startswith

    ```python
    >>> articles = Article.objects.filter(title__startswith='first')
    >>> articles
    <QuerySet [<Article: 1번 글 - First article : Hello, article?>]>
    ```

  - endswith

    ```python
    >>> articles = Article.objects.filter(content__endswith='!')
    >>> articles
    <QuerySet [<Article: 3번 글 - Third article : Django!>]>
    ```



## Delete

### article 인스턴스 생성 후 `.delete() `함수를 실행한다.

```python
>>> article = Article.objects.get(pk=4)
>>> article.delete()
(1, {'articles.Article': 1})
```



## Update

### article 인스턴스 호출 후 값 변경하여 `.save()` 함수 실행

```python
>>> article.content
'contect'
>>> article.content = 'new contents'
>>> article
<Article: 5번 글 - Second article : new contents>
>>> article.save()
```



