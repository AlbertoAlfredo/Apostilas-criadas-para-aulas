# Apostila de Django

## I. Introdução e Setup (O Aluno Começa Aqui)

### 1\. O que é Django? O Conceito MVT/MTV

**Conceito Central:** Django é um *framework web* de alto nível para Python, que incentiva o desenvolvimento rápido e um design limpo e pragmático. Sua filosofia é "Don't Repeat Yourself" (DRY).

| Acrônimo | Significado | Papel no Django |
| :---: | :--- | :--- |
| **M** | **Model (Modelo)** | Responsável por definir a estrutura dos dados (banco de dados). É a sua "fonte da verdade". |
| **V** | **View (Visão)** | É o "controlador" da aplicação. Contém a lógica de negócio, recebe a requisição, interage com o **Model** e decide qual **Template** usar. |
| **T** | **Template** | Define o *layout* e a estrutura de apresentação (HTML). O que o usuário vê. |

### 2\. Configuração e Ambiente Virtual

**Objetivo:** Isolar as dependências do seu projeto para evitar conflitos com outros projetos Python.

#### Exemplo Prático de Setup (Comandos de Aula)

1.  **Crie o Ambiente Virtual (venv):**
    ```bash
    python3 -m venv meu_projeto_django
    ```
2.  **Ative o Ambiente:**
      * **Linux/macOS:**
        ```bash
        source meu_projeto_django/bin/activate
        ```
      * **Windows (CMD):**
        ```bash
        meu_projeto_django\Scripts\activate.bat
        ```
3.  **Instale o Django:**
    ```bash
    pip install django
    ```
4.  **Inicie o Projeto (Cria a Estrutura Base):**
    ```bash
    django-admin startproject configuracoes_site .
    # O ponto final '.' é crucial para criar a estrutura no diretório atual
    ```
5.  **Inicie o Servidor Local:**
    ```bash
    python manage.py runserver
    ```
    *(Resultado esperado: Servidor rodando em [http://127.0.0.1:8000/](https://www.google.com/search?q=http://127.0.0.1:8000/))*

-----

## II. Core de uma Aplicação: Project vs. App

### 1\. Diferença Fundamental

  * **Project (Projeto):** É a instalação completa do Django, que contém as configurações globais (fuso horário, banco de dados, arquivos estáticos).
  * **App (Aplicação):** É um módulo autocontido dentro do Project, que faz algo específico (ex: um App de Blog, um App de Usuários). **A regra de ouro do Django é: crie um App para cada funcionalidade principal.**

### 2\. Criando e Configurando o App

**Objetivo:** Criar um módulo para nossa funcionalidade (Ex: um sistema de Catálogo de Livros).

#### Exemplo Prático (Criando o App "catalogo")

1.  **Crie o App:**

    ```bash
    python manage.py startapp catalogo
    ```

2.  **Registre o App (Passo Crucial\!):** O Django precisa saber que o novo App existe. Abra o arquivo de configurações globais: `configuracoes_site/settings.py`.

3.  **Adicione o nome do App na lista `INSTALLED_APPS`:**

    ```python
    # settings.py
    INSTALLED_APPS = [
        # ... apps padrão do Django ...
        'django.contrib.staticfiles',
        
        # 📚 Nosso novo App
        'catalogo', 
    ]
    ```

-----

## III. Modelos (Models) e Banco de Dados

### 1\. O Mapeamento Objeto-Relacional (ORM)

**Conceito Central:** O **Django ORM** (Object-Relational Mapper) permite que você interaja com seu banco de dados (BD) usando código Python simples, sem precisar escrever SQL. Uma *Classe Python* que herda de `models.Model` é uma *tabela* no seu BD, e um *atributo* dessa classe é uma *coluna* na tabela.

### 2\. Tipos de Campos e Relações

#### Exemplo Prático (Definindo a Estrutura de Dados do Catálogo)

No arquivo **`catalogo/models.py`**, vamos definir como é um `Livro` e um `Autor`.

```python
# catalogo/models.py
from django.db import models

class Autor(models.Model):
    # Campos que definem um Autor
    nome = models.CharField(max_length=100)
    data_nascimento = models.DateField(null=True, blank=True)

    def __str__(self):
        # Representação legível do objeto
        return self.nome

class Livro(models.Model):
    titulo = models.CharField(max_length=200)
    
    # Exemplo de Relação: Um Autor pode ter vários Livros (ForeignKey)
    # Se o Autor for deletado, os Livros dele também serão (CASCADE)
    autor = models.ForeignKey(Autor, on_delete=models.CASCADE)
    
    isbn = models.CharField(max_length=13, unique=True)
    resumo = models.TextField(help_text="Breve descrição do livro")
    
    class Meta:
        # Ordem padrão de exibição dos livros
        ordering = ['titulo']

    def __str__(self):
        return self.titulo
```

### 3\. Criando as Tabelas (Migrations)

**Objetivo:** Converter as Classes Python (`models.py`) em comandos SQL e aplicá-los ao banco de dados.

#### Comandos de Aula (Onde a Mágica Acontece)

1.  **Crie os Arquivos de Migração:** O Django detecta as mudanças no `models.py` e prepara o SQL.

    ```bash
    python manage.py makemigrations
    ```

    *(Resultado esperado: Criação de um arquivo `0001_initial.py` dentro da pasta `catalogo/migrations`)*

2.  **Aplique as Migrações:** O Django executa o SQL no banco de dados (por padrão, usa o SQLite, um arquivo `.sqlite3` na raiz do projeto).

    ```bash
    python manage.py migrate
    ```

    *(Resultado esperado: Criação das tabelas no BD, incluindo `catalogo_autor` e `catalogo_livro`)*

-----

## IV. Views e Lógica (O Processador de Requisições) 🧠

### 1\. O que é uma View?

**Conceito Central:** A View é uma função (ou classe) Python que aceita uma requisição web (`HttpRequest`) e retorna uma resposta web (`HttpResponse`). Ela contém toda a **lógica de negócio** da sua aplicação:

1.  Receber a requisição do usuário (via URL).
2.  Acessar o banco de dados (interagir com os **Models**).
3.  Processar os dados.
4.  Renderizar um **Template** (ou retornar outros tipos de resposta, como JSON).

### 2\. Views Baseadas em Função (FBV) vs. Classe (CBV)

| Tipo | Descrição | Uso Principal |
| :---: | :--- | :--- |
| **Função (FBV)** | Simples função Python que recebe `request`. | Ideal para lógica simples, leitura (GET), ou quando a personalização é muito específica. |
| **Classe (CBV)** | Classes que herdam de `View` ou outras classes genéricas do Django. | Ideal para operações CRUD (Criação, Leitura, Atualização, Deleção) complexas e repetitivas, pois o Django fornece classes prontas (*Generic Views*). |

### 3\. Exemplo Prático: Lista de Livros (FBV)

Vamos criar uma *View* que busca todos os livros que definimos no `models.py` e os prepara para serem exibidos.

**Arquivo:** `catalogo/views.py`

```python
from django.shortcuts import render
from .models import Livro # Importamos o nosso Model

# A View: Recebe a requisição e retorna a resposta
def lista_livros(request):
    # 1. Lógica de Negócio: Consulta ao Banco de Dados (usando o ORM)
    livros_encontrados = Livro.objects.all().order_by('titulo')
    
    # 2. Contexto: Dicionário que envia dados para o Template
    contexto = {
        'titulo_pagina': 'Catálogo Completo de Livros',
        'livros_list': livros_encontrados # Variável que será usada no HTML
    }
    
    # 3. Resposta: Renderiza o template, passando os dados
    return render(request, 'catalogo/lista_livros.html', contexto)

# O que a View faz:
# request -> (View) -> Model (Consulta ao BD) -> Dados -> (View) -> Template (Renderiza HTML) -> Response
```

-----

## VI. Rotas (URLs) (O Mapa da Aplicação) 🗺️

### 1\. O Fluxo da URL

O Django utiliza dois arquivos principais de `urls.py`:

1.  **Project `urls.py` (Global):** Ponto de entrada. Mapeia grandes blocos de URLs para seus respectivos Apps.
2.  **App `urls.py` (Local):** Define as rotas internas de um App, mapeando cada URL específica para uma **View**.

### 2\. Exemplo Prático: Mapeando a View

#### Passo 1: Configuração Global (Project)

Conecte o App "catalogo" ao projeto.
**Arquivo:** `configuracoes_site/urls.py` (Do seu projeto principal)

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    
    # 🔗 Conecta todas as URLs que começam com 'catalogo/' ao App 'catalogo'
    path('catalogo/', include('catalogo.urls')), 
]
```

#### Passo 2: Configuração Local (App)

Crie este arquivo dentro da pasta `catalogo`.
**Arquivo:** `catalogo/urls.py` (Crie este arquivo)

```python
from django.urls import path
from . import views

# Namespace: ajuda a referenciar URLs de forma única no projeto (ex: "catalogo:lista")
app_name = 'catalogo' 

urlpatterns = [
    # Mapeia a URL 'catalogo/' (o path está vazio) para a função lista_livros
    # O 'name' é um identificador que permite referenciar a URL no código Python/Templates
    path('', views.lista_livros, name='lista'), 
]

# Resultado final da rota: http://127.0.0.1:8000/catalogo/
```

-----

## V. Templates e Frontend (A Apresentação) 🎨

### 1\. Django Template Language (DTL)

**Conceito Central:** O Template é um arquivo HTML que usa a sintaxe especial do Django para receber os dados do **Contexto** (enviados pela View) e renderizá-los dinamicamente.

| Sintaxe | Descrição | Exemplo |
| :---: | :--- | :--- |
| `{{ variável }}` | **Variável:** Imprime o valor da variável do contexto. | `<h1>Bem-vindo, {{ usuario.nome }}</h1>` |
| `{% tag %}` | **Tag:** Executa lógica, como loops, condicionais ou herança. | `{% for item in lista %}` |

### 2\. Exemplo Prático: Herança e Exibição de Dados

Para evitar repetir HTML (cabeçalhos, rodapés), usamos a **Herança de Templates**.

#### Passo 1: Crie um Template Base (O layout principal)

Você precisará criar uma pasta `templates` dentro do seu app `catalogo` e colocar este arquivo lá: `catalogo/templates/base.html`.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Catálogo Django - {% block title %} {% endblock %}</title>
</head>
<body>
    <header><h1>Meu Catálogo Literário</h1></header>
    <main>
        {% block content %}
        {% endblock %}
    </main>
    <footer>&copy; 2025 - Projeto de Estudos</footer>
</body>
</html>
```

#### Passo 2: Crie o Template da View (Conteúdo Específico)

**Arquivo:** `catalogo/templates/catalogo/lista_livros.html`

```html
{% extends 'base.html' %} 
{% block title %} {{ titulo_pagina }} {% endblock %}

{% block content %}
    <h2>{{ titulo_pagina }}</h2> 

    {% if livros_list %}
        <ul>
            {% for livro in livros_list %}
                <li>
                    <strong>{{ livro.titulo }}</strong> por {{ livro.autor.nome }} 
                    <p>{{ livro.resumo|truncatechars:100 }}</p>
                    </li>
            {% endfor %}
        </ul>
    {% else %}
        <p>Ainda não há livros cadastrados. Crie alguns no Admin!</p>
    {% endif %}

{% endblock %}
```

-----
## VII. Administração (Admin Site) 👑

### 1\. O que é o Admin Site?

**Conceito Central:** O Django vem com um painel administrativo pronto para uso, que permite a administradores (ou usuários autorizados) criar, ler, atualizar e deletar (**CRUD**) dados dos Models sem escrever código de *frontend*. É perfeito para gerenciamento imediato de conteúdo.

### 2\. Passo 1: Criando o Superusuário

Você precisa de uma conta de administrador para acessar o painel.

#### Comando de Aula:

```bash
python manage.py createsuperuser
```

*(Resultado: O sistema pedirá Nome de Usuário, Email e Senha.)*

### 3\. Passo 2: Registrando os Models

Para que seus Models (`Autor` e `Livro`) apareçam no painel, eles precisam ser explicitamente registrados no arquivo `admin.py` do seu App.

#### Exemplo Prático de Registro

**Arquivo:** `catalogo/admin.py`

```python
from django.contrib import admin
from .models import Autor, Livro

# 1. Registro Simples
admin.site.register(Autor)

# 2. Registro Personalizado (Para melhorar a interface)
@admin.register(Livro)
class LivroAdmin(admin.ModelAdmin):
    # 📝 Personalização da lista de objetos:
    # Mostra título, autor e ISBN na tabela principal
    list_display = ('titulo', 'autor', 'isbn') 
    
    # 🔍 Adiciona uma caixa de busca
    search_fields = ('titulo', 'autor__nome')
    
    # 📅 Adiciona um filtro lateral por campos de data
    list_filter = ('autor', 'isbn')
    
# Depois de registrar, inicie o servidor (python manage.py runserver)
# e acesse http://127.0.0.1:8000/admin/
```

*(Resultado: Você poderá adicionar e editar Autores e Livros usando o formulário gerado automaticamente pelo Django.)*

-----

## VIII. Formulários (Forms) ✍️

### 1\. O Papel do Formulário

**Conceito Central:** O sistema de Formulários do Django gerencia três coisas:

1.  **Preparação de Dados:** Renderiza o formulário HTML na tela.
2.  **Validação:** Garante que os dados enviados pelo usuário (POST) sejam válidos e seguros (ex: o campo de email é um email?).
3.  **Persistência:** Salva os dados limpos no banco (ou os utiliza na lógica da View).

### 2\. ModelForm: O Facilitador

O `ModelForm` é o tipo de formulário mais útil quando se trabalha com Models, pois ele se baseia na estrutura do seu Model (`Livro`) para gerar os campos, fazer a validação e salvar o objeto automaticamente.

#### Passo 1: Criando o ModelForm

Crie um novo arquivo para guardar a definição do seu formulário.

**Arquivo:** `catalogo/forms.py` (Crie este arquivo)

```python
from django import forms
from .models import Livro

class LivroModelForm(forms.ModelForm):
    class Meta:
        model = Livro
        # 📚 Campos que queremos que o usuário possa preencher
        fields = ['titulo', 'autor', 'isbn', 'resumo']
        
        # Opcional: Personalizar labels
        labels = {
            'titulo': 'Título da Obra',
            'autor': 'Nome do Autor'
        }
```

#### Passo 2: A View que Processa o Formulário (CRUD: Create)

Esta View recebe o formulário, tenta validá-lo e o salva.

**Arquivo:** `catalogo/views.py` (Adicione a nova View)

```python
from django.shortcuts import render, redirect
from .forms import LivroModelForm # 👈 Importe o Form
# ... (outras imports) ...

def criar_livro(request):
    # 1. Checa o tipo de requisição
    if request.method == 'POST':
        # Instancia o form com os dados do POST
        form = LivroModelForm(request.POST) 
        
        # 2. Validação e Salvamento
        if form.is_valid():
            form.save() # Salva no banco de dados
            return redirect('catalogo:lista') # Redireciona para a lista após salvar
    
    else: # Requisição GET: apenas mostra o formulário vazio
        form = LivroModelForm()
        
    contexto = {'form': form}
    return render(request, 'catalogo/criar_livro.html', contexto)
```

#### Passo 3: Configurando URL e Template

**a) URL (catalogo/urls.py):**

```python
# ...
urlpatterns = [
    # ... path('', views.lista_livros, name='lista'),
    path('criar/', views.criar_livro, name='criar'), # 👈 Nova rota
]
```

**b) Template (catalogo/templates/catalogo/criar\_livro.html):**

```html
{% extends 'base.html' %} 

{% block title %} Criar Novo Livro {% endblock %}

{% block content %}
    <h2>Adicionar Novo Livro ao Catálogo</h2>
    
    <form method="POST"> 
        {% csrf_token %} 
        
        {{ form.as_p }} 
        
        <button type="submit">Salvar Livro</button>
    </form>

{% endblock %}
```

-----

## 🚀 Conclusão da Apostila Base

Você acaba de cobrir o ciclo completo de desenvolvimento de uma aplicação web com Django:

1.  **Setup/Estrutura**
2.  **Modelagem de Dados (M)**
3.  **Lógica (V)**
4.  **Apresentação (T)**
5.  **Rotas (URLs)**
6.  **Gestão (Admin)**
7.  **Interação do Usuário (Forms)**

**Com este conhecimento, você pode construir qualquer aplicação CRUD (Create, Read, Update, Delete) básica no Django.**