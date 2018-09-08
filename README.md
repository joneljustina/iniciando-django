# Django no PyCharm Professional

## Requisitos

- Python 3.6 (Suportada pelo Heroku)
- IDE PyCharm Professional 2018.2.x (Ideal pois já tem suporte nativo ao Django)
- git (controle de versão)

## Ciclo de Request e Response

É importante ter em mente também como o Django trabalha desde que chega uma requisição, até momento que ele entrega uma resposta.

O Ryan Nevius (https://www.ryannevius.com) criou uma figura que explica muito bem como funciona esse ciclo e quais os arquivos responsáveis. Vou deixar a figura aqui pois ela é muito útil para tudo que será feito a seguir.

![Django Request-Response cycle](http://rnevius.github.io/django_request_response_cycle.png)

## Iniciando o projeto

Em location, evitar caminho do projeto com diretórios com "espaço" pois já encontrei alguns problemas por causa disso em algumas versões do PyCharm instalando o sistema todo na no diretório escolhido.

Ex:

```
/Users/thomazrb/python/meu_projeto
```

Verificar alguns pontos:

- Criar um *Virtual Environment* com Virtualenv
- Lembrar no Interpretador base de colocar o Python 3.6 (caso tenha mais uma versão do Python instalada)

Após criado o projeto terá a seguinte estrutura:

```
meu_projeto/
├───meu_projeto/
│   ├───__init__.py
│   ├───settings.py
│   ├───urls.py
│   └───wsgi.py        
├───templates/
├───venv/
├───db.sqlite3
└───manage.py
```

Onde o diretório **meu_projeto** (interno) é o onde estão as configurações da aplicação, o diretório **templates** será onde ficarão os templates html base (que não pertencem a nenhuma app) o diretório **venv** é do ambiente virtual do Virtualenv e não é necessário se preocupar com ele, o arquivo **db.sqlite3** é o banco do ambiente de desenvolvimento e o arquivo **manage.py** é o arquivo para gerenciar tudo no Django (mas no caso do PyCharm Professional este arquivo irá ser chamado através do menu ```Tools > Run manage.py Task...```

No diretório interno **meu_projeto** temos o arquivo vazio **__init__.py**, obrigatório para todo pacote python, o arquivo **settings.py** onde fica todas as configurações do servidor, o arquivo **urls.py** que possui as rotas ou caminhos para todas as views do seu site e o **wsgi.py**, que é a *Web Server Gateway Interface*, ou seja, a interface de comunicação entre o servidor Web e a aplicação. Ela recebe a requisição e devolve a resposta para o servidor web.

Já é possível verificar se está tudo ok inicializando o servidor (no PyCharm Professional não é necessário executar através do **manage.py**) através do botão run (play) na parte superior direita da IDE.

Se tudo correu bem, a tela de boas vindas do Django será exibida no browser no endereço http://127.0.0.1:8000.

## Melhorando a segurança

Neste ponto é interessante instalar o pacote **python-decouple** por motivos de segurança, através da aba terminal da IDE (parte inferior). Desta forma, retiramos informações importantes do **settings.py**:

```
pip install python-decouple
```

O python-decouple utiliza um arquivo chamado **.env** (não confundir com o diretório **venv** do ambiente virtual) para armazenar as informações sensíveis do seu servidor, logo o mesmo deve ser criado na raiz do seu projeto.

```
meu_projeto/
├───meu_projeto/
│   ├───__init__.py
│   ├───settings.py
│   ├───urls.py
│   └───wsgi.py        
├───templates/
├───.env
├───venv/
├───db.sqlite3
└───manage.py
```

No momento a única informação sensível que o **settings.py** possui é a variável SECRET_KEY.

Portanto o arquivo **.env** irá conter sua SECRET_KEY (ao contrário do **settings.py**, não se usa espaço entre o igual, nem aspas nos valores (lembrar de retirar as aspas da secret key).

Outra variável interessante de se colocar neste arquivo é a DEBUG, não por conter informação sensível, mas depois que a aplicação estiver em produção, será mais fácil 'ligar' e 'desligar' o modo de DEBUG.

Portanto eis o **.env** (com uma chave aleatória):

```
SECRET_KEY=bl^f#)qsqa9gfvxrhd&6vg$tip7ednn4a4p4+15-r7fpc8&n0=
DEBUG=True
```

Para que o arquivo **settings.py** consiga ler as informações no arquivo **.env** deve-se importar o módulo do decoulpe no início do arquivo **settings.py**, e trocar as variáveis pela função que ele verifica o config:

```python
from decouple import config

(...)

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
```

Poderia ser utilizado apenas ```DEBUG = config('DEBUG')```, mas da forma que está, caso não encontre a variável DEBUG, o mesmo seja False, ou seja, ambiente de produção.

Verificar se o servidor ainda está rodando e está tudo Ok.

## Inicializando o controle de versão

Nesse ponto é interessante já que iremos brincar com o código, inicializar um controle de versão. Colocaremos um arquivo **.gitignore** na raiz do projeto com os arquivos que serão excluídos do controle de versão. Importantíssimo para evitar que mande arquivos desnecessários ou com dados sensíveis para um repositório público ou compartilhado.

```
meu_projeto/
├───meu_projeto/
│   ├───__init__.py
│   ├───settings.py
│   ├───urls.py
│   └───wsgi.py        
├───templates/
├───.env
├───.gitignore
├───venv/
├───db.sqlite3
└───manage.py
```

Conteúdo do **.gitignore**:
```ini
# Ignorar os arquivos da IDE do PyCharm
.idea
# Ignorar o banco local sqlite3
*.sqlite3
# Ignorar nosso ambiente virtual
venv
# Ignorar o arquivo com dados sensíveis
.env
```

Pode-se então iniciar o git no terminal.

```
git init
```

Adicionar todos os arquivos do projeto (o .gitignore já se encarregará de ignorar os arquivos desnecessários). Para isso basta clicar com botão direito na raiz do projeto e selecionar o menu ```Git > + Add```

É interessante lembrar que o PyCharm utiliza as seguintes cores para os arquivos:

- Arquivo cinza na cor padrão: ou o arquivo está sincronizado com o repositório, ou seja, está em sua última versão ou o arquivo não é versionado pelo controle de versões (por causa do **.gitignore**)
- Arquivo vermelho: o arquivo ainda não está sendo versionado e deve ser adicionado ao controle de versão com o comando ```git add .``` no terminal, ou através do menu popup que foi executado no passo acima.
- Arquivo verde: o arquivo acabou de ser adicionado ao sistema de versão e será monitorado a partir de então).
- Arquivo azul: o arquivo estava sincronizado com o repositório (cinza) e foi modificado, ou seja, este arquivo foi alterado desde o último commit.

Neste ponto, a árvore de diretório estará cheia de arquivos verdes (que serão armazenados no repositório) e cinzas (que não estão sendo monitorados).

Utilizando mais uma vez os poderes do PyCharm vamos commitar essa mudança pelo menu VCS (Version Control System): ```VCS > Commit... ```, onde é possível verificar que todos os novos arquivos estão selecionados e precisamos apenas de uma mensagem para o commit, como *"Iniciando a aplicação"*.

tudo isso pode ser feito também pelo terminal para os amantes do terminal:

```
git init
git add .
git commit -m 'iniciando a aplicação'
```
Só cuidado que sistemas \*nix e Windows usam aspas simples e duplas respectivamente.

Após *commitar* o código, todos os arquivos devem ficar cinza.

## Colocando o Django em português

Uma vez que tudo está preparado, é hora de começar a brincar, e um bom início é deixando o sistema em português. Para isso vamos alterar as variáveis responsáveis pelo idioma e fuso-horário para o português no arquivo **settings.py**:

```python
LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Sao_Paulo'
```

A partir deste ponto, o sistema estará em português, e o resultado pode ser conferido recarregando o navegador ou entrando novamente no endereço http://127.0.0.1:8000.

O arquivo **settings.py** também estará azul indicando que o mesmo foi modificado. É interessante executar um novo commit informando a mudança através do menu ```VCS > Commit... ```, como da vez anterior, porém agora com uma mensagem por ex. *"Mudando o idioma do sistema para português"*.

Na janela que se abre para colocar a mensagem do commit é possível perceber o arquivo que está sendo *commitado* que no caso é o *settings.py* e conferir onde foram feitas as mudanças.

À partir deste ponto não mencionarei novos commits, porém subentende-se que a cada mudança, é interessante a realização de um commit para registrar o que foi feito passo a passo, e caso seja necessário retroceder algum ponto do código, fique fácil.

## Corretor ortográfico para termos em português no PyCharm

Uma coisa chata são as palavras em português não serem reconhecidas no PyCharm. Isso pode ser resolvido adicionando dicionários em português (https://github.com/rafaelsc/IntelliJ.Portuguese.Brazil.Dictionary) à IDE. Basta adicionar os 4 dicionários deste repositório nas preferências do PyCharm ```Preferences > Editor > Spelling``` e pronto! O PyCharm agora reconhece palavras e variáveis em português.

## Um CRUD rápido para o projeto ficar funcional

A idéia é criar um CRUD simples, por exemplo de produtos. Para criar a App produto utiliza-se o **manage.py**. Utilizando os poderes da IDE vamos fazer pelo menu ```Tools > Run manage.py Task...```

### Criando uma App

Para criar a app, na janela do manage.py utiliza-se:

```
startapp produtos
```

Ou, para os amantes do terminal:

```
python manage.py startapp produtos
```

Evito essa aproximação para aproveitar o potencial da IDE do PyCharm onde o mesmo entrega uma janela do terminal, uma do manage.py, uma do servidor rodando, etc... fica mais facil para ver onde está o que, e verificar o que acabou de fazer. Mais organizado que usar uma única janela para tudo.

A estrutura agora ficará:

```
meu_projeto/
├───meu_projeto/
│   ├───__init__.py
│   ├───settings.py
│   ├───urls.py
│   └───wsgi.py
├───produtos/
│   ├───migrations/
│   ├───__init__.py
│   ├───admin.py
│   ├───apps.py
│   ├───models.py
│   ├───tests.py
│   └───views.py
├───templates/
├───.env
├───venv/
├───db.sqlite3
└───manage.py
```

Aconselho também a criar o diretório **templates** para essa app e a o arquivo **urls.py** de forma a deixar as Apps do projeto mais isoladas possível, e o projeto mais organizado.

```
meu_projeto/
├───meu_projeto/
│   ├───__init__.py
│   ├───settings.py
│   ├───urls.py
│   └───wsgi.py
├───produtos/
│   ├───migrations/
│   ├───templates/
│   ├───__init__.py
│   ├───admin.py
│   ├───apps.py
│   ├───models.py
│   ├───tests.py
│   ├───urls.py
│   └───views.py
├───templates/
├───.env
├───venv/
├───db.sqlite3
└───manage.py
```

Deve-se ativar o App recém criado produtos, na variável INSTALLED_APPS do **settings.py**:

```python
INSTALLED_APPS = [
    
    (...)
    
    'produtos',
]
```

O primeiro template será criado no diretório **meu_projeto/produtos/templates/** pois pertencerá a App produto.

Será o **lista_produtos.html**, que por enquanto terá apenas um cabeçalho simples.

```html
<h1>Lista de produtos</h1>
```

Os arquivos do projeto estão desta forma agora:

```
meu_projeto/
├───meu_projeto/
│   ├───__init__.py
│   ├───settings.py
│   ├───urls.py
│   └───wsgi.py
├───produtos/
│   ├───migrations/
│   ├───templates/
│   │   └───lista_produtos.html
│   ├───__init__.py
│   ├───admin.py
│   ├───apps.py
│   ├───models.py
│   ├───tests.py
│   ├───urls.py
│   └───views.py
├───templates/
├───.env
├───venv/
├───db.sqlite3
└───manage.py
```

É necessário configurar uma view para acessar este template, no arquivo **meu_projeto/produtos/views.py**:

```python
def lista_produtos(request):
    return render(request, 'lista_produtos.html')
```

E também criar uma rota para esta view no arquivo recém criado **meu_projeto/produtos/urls.py**:

```python
from django.urls import path
from .views import lista_produtos


urlpatterns = [
    path('', lista_produtos, name="lista_produtos"),
]
```

Foi necessário importar a função *path* do módulo *django.urls*, e importar a view recém criada lista produtos (o .views indica o arquivo views do App atual).

Caso fosse utilizado o arquivo geral de urls que veio no projeto (**meu_projeto/meu_projeto/urls.py**), a view teria que ser importada como ```from produtos.views import lista_produtos```.

A função *path* recebe 3 argumentos, o caminho do servidor, a view que será chamada e um nome para ser possível referenciá-la nos templates HTML.

Uma vez que foi utilizado um arquivo de urls local dentro da App, é necessário referenciar este arquivo no arquivo de urls do projeto (**meu_projeto/meu_projeto/urls.py**). Para isso é necessário importar a função *include*, e as urls da App projeto. Portanto o arquivo **meu_projeto/meu_projeto/urls.py** fica:

```python
from django.contrib import admin
from django.urls import path, include
from produtos import urls as produtos_urls

urlpatterns = [
    path('', include(produtos_urls)),
    path('admin/', admin.site.urls),
]
```

O caminho do servidor está vazio pois é desejavel que a lista apareça na raiz do servidor. Se tudo correu bem, recarregando o navegador ou entrando novamente no endereço http://127.0.0.1:8000, será exibida a página HTML criada no template com a mensagem *"Lista de produtos"*.

Outra rota já criada por padrão é a rota do painel administração do Django, que pode ser acessada através da URL http://127.0.0.1:8000/admin/. Acessando esse endereço irá retornar um erro pois o banco de dados ainda não foi inicializado.

### Inicializando o Banco de Dados

Para iniciar o banco de dados, é necessário criar migrações e logo em seguida deve-se aplicá-las (migrá-las) ao banco de dados. Na situação atual, as migrações já estão criadas por padrão (migrações do admin). Quando cria-se/modifica-se Models, será necessário criar as migrações antes de migrá-las para o banco de dados. No menu ```Tools > Run manage.py Task...```, para criar migrações quando necessário:

```
makemigrations
```
E para aplicá-las ao banco de dados:

```
migrate
```

Executando o *migrate*, a URL http://127.0.0.1:8000/admin/ irá funcionar e irá redirecionar para uma tela de login.


