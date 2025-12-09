# Apostila Django - Static Files
## 📁 1. O que são e como funcionam os Static Files (Arquivos Estáticos)

### Conceito Central

Arquivos Estáticos são todos os arquivos que não mudam (ou mudam muito pouco) durante a execução da aplicação e que são servidos diretamente ao navegador do usuário. No contexto do Django, o gerenciamento de estáticos envolve três passos principais:

1.  **Desenvolvimento:** Onde o Django encontra esses arquivos no seu código.
2.  **Referência:** Como você os chama nos seus templates HTML.
3.  **Produção:** Onde os arquivos são coletados para serem servidos de forma otimizada.

### Estrutura de Diretórios 🌲

O Django procura arquivos estáticos em dois locais principais:

1.  **Diretório `static` dentro de cada App:** O local **recomendado** para arquivos estáticos específicos daquele App (Ex: o CSS do App `catalogo`).
2.  **Diretório Global `STATICFILES_DIRS`:** Um local definido no nível do Projeto para arquivos estáticos globais (Ex: o arquivo CSS base de todo o site).

#### Exemplo Prático de Estrutura

Para o nosso App `catalogo`, a estrutura seria:

```
configuracoes_site/
└── catalogo/
    ├── static/
    │   └── catalogo/    <-- (Melhor Prática: Subpasta com o nome do App)
    │       ├── css/
    │       │   └── style.css
    │       └── js/
    │           └── script.js
    └── templates/
```

-----

## 🛠️ 2. Configuração e Referência nos Templates

### 2.1. Configuração do Projeto

Por padrão, o Django já tem a configuração básica, mas precisamos definir o diretório global se você quiser usá-lo.

**Arquivo:** `configuracoes_site/settings.py`

```python
# settings.py

# ... (parte de baixo do arquivo)

# Ponto de acesso à URL para arquivos estáticos (sempre termina com barra)
STATIC_URL = '/static/' 

# Opcional, mas recomendado: Diretório global de estáticos (para arquivos de terceiros ou base)
import os 
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static_global'), # Defina onde estará esta pasta
]

# ⚠️ Variável para Produção: Onde o Django irá COLETAR TODOS os estáticos
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles') 
```

*(Você precisaria criar a pasta `static_global` na raiz do seu projeto se usar `STATICFILES_DIRS`)*.

### 2.2. Referenciando nos Templates (A Tag Mágica)

Para garantir que o Django encontre o caminho correto para seus arquivos estáticos, você deve usar a *template tag* `{% static %}`.

#### Exemplo Prático de Uso no HTML

**Arquivo:** `catalogo/templates/base.html` (ou outro template)

```html
{% load static %} 
<!DOCTYPE html>
<html>
<head>
    <title>...</title>
    <link rel="stylesheet" type="text/css" href="{% static 'catalogo/css/style.css' %}">
</head>
<body>
    <img src="{% static 'catalogo/images/logo.png' %}" alt="Logo">

    <script src="{% static 'js/global_script.js' %}"></script>
</body>
</html>
```

-----

## 📦 3. O Processo de Coleta (`collectstatic`)

O processo de desenvolvimento (`python manage.py runserver`) consegue servir os arquivos estáticos automaticamente a partir das pastas `static/` de cada App.

No entanto, quando você move sua aplicação para **Produção** (servidores reais como AWS, Azure, etc.), o Django não serve esses arquivos diretamente. Para isso, usamos o comando:

### Comando de Aula: Coleta dos Arquivos

```bash
python manage.py collectstatic
```

**O que este comando faz:**

1.  Ele varre todas as pastas `static/` de todos os Apps e o diretório `STATICFILES_DIRS`.
2.  Copia todos os arquivos encontrados para um único local centralizado, definido pela variável `STATIC_ROOT` no seu `settings.py`.
3.  Em produção, você configura o seu servidor web (Apache, Nginx, Gunicorn) para servir o conteúdo desse diretório `STATIC_ROOT` diretamente, de forma eficiente e rápida.

-----

## ⚠️ Checklist

Para ter um sistema funcional e bonito, garanta que siga estes passos:

1.  **Crie a Pasta:** Crie a estrutura `catalogo/static/catalogo/css/` e adicione um `style.css` de teste.
2.  **Use `{% load static %}`:** Coloque-o no topo de todos os templates que usarão estáticos.
3.  **Use `{% static 'caminho' %}`:** Use esta *template tag* em vez de caminhos de URL "escritos na mão" (Ex: `/static/css/style.css`).

Isso garante que seu sistema Django estará pronto tanto para o desenvolvimento quanto para o ambiente de produção\!
