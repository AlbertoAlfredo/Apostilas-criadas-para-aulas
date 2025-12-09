# Apostila Django - Autenticação e Autorização

## 🔒 IX. Autenticação e Autorização

### 1\. Conceitos Fundamentais

| Termo | Definição |
| :--- | :--- |
| **Autenticação (Authentication)** | É o processo de **verificar a identidade** do usuário (login). Quem é você? |
| **Autorização (Authorization)** | É o processo de **verificar o que o usuário pode fazer** no sistema (permissões). Você pode fazer isso? |
| **Usuário (`User`)** | A conta básica, armazenada no modelo `django.contrib.auth.models.User`. |
| **Permissões (`Permissions`)** | O Django cria permissões para cada ação CRUD nos seus Models (ex: `add_livro`, `delete_autor`). |

### 2\. Configuração Inicial (Mínima)

O sistema de autenticação já vem habilitado por padrão em `settings.py` (via `INSTALLED_APPS` e `MIDDLEWARE`), mas precisamos garantir que ele saiba para onde ir.

**Arquivo:** `configuracoes_site/settings.py`

```python
# settings.py

# URL para onde o usuário será redirecionado após o login
LOGIN_REDIRECT_URL = '/' # Redireciona para a página inicial

# URL onde está a view de login. 
# Por padrão, o Django espera '/accounts/login/' 
# Se você tiver uma view customizada, defina aqui.
LOGIN_URL = '/login/' 
```

### 3\. Criando as Páginas de Login e Logout

Você não precisa criar o formulário nem a View de login do zero\! O Django fornece as Views e os Formulários prontos. Você só precisa configurar as URLs e criar os Templates.

#### Passo 1: Configuração de URLs

No seu arquivo de URLs do projeto (e não do App), você pode incluir as URLs padrões do Django.

**Arquivo:** `configuracoes_site/urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('catalogo.urls', namespace='catalogo')),
    
    # 🔑 Adiciona todas as URLs padrões de autenticação do Django
    path('accounts/', include('django.contrib.auth.urls')), 
    
    # Se você optou por LOGIN_URL = '/login/' em settings.py
    # Você terá que mapear a URL manualmente:
    # path('login/', auth_views.LoginView.as_view(), name='login'),
]
```

#### Passo 2: Criando os Templates Necessários

Quando você inclui `django.contrib.auth.urls`, o Django espera encontrar os seguintes arquivos em uma pasta específica.

**Estrutura de Templates:**

```
configuracoes_site/
└── templates/
    └── registration/
        ├── login.html     <-- Template para a View de Login
        └── logged_out.html <-- Template para a View de Logout
```

**Conteúdo Mínimo de `login.html`:**

```html
{% extends 'base.html' %} 

{% block title %} Entrar no Sistema {% endblock %}

{% block content %}
    <h2>Login</h2>
    
    <form method="POST">
        {% csrf_token %}
        
        {{ form.as_p }} 
        
        <button type="submit">Entrar</button>
    </form>
{% endblock %}
```

### 4\. Controlando o Acesso em Views

O principal uso da autorização é restringir o que usuários anônimos (não logados) podem ver ou fazer.

#### Opção A: Decorador em Views Baseadas em Função (FBV)

É a maneira mais simples de proteger uma View inteira. Se o usuário não estiver logado, ele será redirecionado para a página de login (`LOGIN_URL`).

**Arquivo:** `catalogo/views.py`

```python
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required # 👈 Importe o decorador

@login_required 
def criar_livro(request):
    """Esta view só pode ser acessada por usuários autenticados."""
    # ... (lógica da criação do livro) ...
    pass
```

#### Opção B: Verificação Manual no Template

Você pode mostrar ou esconder conteúdo dependendo do status do usuário, usando a variável global `{{ user }}`.

**Arquivo:** `catalogo/templates/base.html`

```html
<body>
    <header>
        {% if user.is_authenticated %} 
            <p>Olá, {{ user.username }}!</p>
            <a href="{% url 'logout' %}">Sair</a> 
            <a href="{% url 'catalogo:criar' %}">Adicionar Livro</a>
        {% else %}
            <a href="{% url 'login' %}">Entrar</a>
        {% endif %}
    </header>
```

### 5\. Boas Práticas: Testando Permissões Específicas

Para um controle de acesso mais granular (ex: "apenas administradores podem deletar livros"), você usa as permissões.

#### Exemplo: Permitir Edição Apenas se o Usuário Puder Mudar Livros

**Arquivo:** `catalogo/views.py` (Usando um decorador mais específico)

```python
from django.contrib.auth.decorators import permission_required

@permission_required('catalogo.change_livro') 
# O formato é: [app_label].[action]_[model_name]
def editar_livro(request, pk):
    """Esta view só pode ser acessada por usuários que possuam a permissão 'can change livro'."""
    # ... (lógica da edição do livro) ...
    pass
```

