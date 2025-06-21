# Boas Práticas para Desenvolvimento de Tema WordPress Base

## 🌟 Objetivo

Criar um tema base WordPress totalmente customizado, leve, organizado, reutilizável e compatível com as boas práticas modernas do ecossistema WP.

---

## 📂 1. Estrutura Recomendada de Pastas e Arquivos

```
your-theme/
├── assets/
│   ├── css/
│   ├── js/
│   ├── images/
│   └── scss/
├── includes/
│   ├── setup.php
│   ├── enqueue.php
│   ├── custom-post-types.php
│   └── helpers.php
├── parts/
├── templates/
├── functions.php
├── style.css
├── index.php
├── screenshot.png
└── theme.json
```

---

## 🧱 2. Boas Práticas de Código (com alto detalhamento)

### 2.1 Modularização de Funções

Evite lotar o `functions.php`. Use a pasta `includes/` para modularizar seu código e dividir as funcionalidades.

**Exemplo no `functions.php`:**

```php
    require_once get_template_directory() . '/includes/setup.php';
    require_once get_template_directory() . '/includes/enqueue.php';
    require_once get_template_directory() . '/includes/custom-post-types.php';
```

---

### 2.2 Prefixos ou Namespaces

Use prefixos em todas as funções para evitar conflitos com plugins.

**Errado:**

```php
    function register_menu() { ... }
```

**Certo:**

```php
    function mytheme_register_menu() { ... }
```

Em projetos maiores, considere namespaces:

```php
    namespace MyTheme\Setup;

    function register_menus() { ... }
```

---

### 2.3 Configuração do Tema

**Exemplo em `setup.php`:**

```php
    function mytheme_setup_theme() {
        add_theme_support('title-tag');
        add_theme_support('post-thumbnails');
        add_theme_support('menus');
        add_theme_support('html5', [
            'search-form',
            'comment-form',
            'comment-list',
            'gallery',
            'caption',
        ]);
    }
    add_action('after_setup_theme', 'mytheme_setup_theme');
```

---

### 2.4 Inclusão de Scripts e Estilos

**Exemplo em `enqueue.php`:**

```php
    function mytheme_enqueue_assets() {
        wp_enqueue_style('mytheme-style', get_template_directory_uri() . '/assets/css/main.css', [], '1.0');
        wp_enqueue_script('mytheme-script', get_template_directory_uri() . '/assets/js/main.js', [], '1.0', true);
    }
    add_action('wp_enqueue_scripts', 'mytheme_enqueue_assets');
```

> Nunca inclua `<link>` ou `<script>` diretamente nos arquivos HTML do tema.

---

### 2.5 Escape e Sanitização

**Entradas devem ser sanitizadas:**

```php
    $title = sanitize_text_field($_POST['title']);
```

**Saídas devem ser escapadas:**

```php
    echo esc_html(get_the_title());
    echo esc_url(get_permalink());
```

> **Regra de ouro:**
> Entradas = `sanitize_*`
> Saídas = `esc_*`

---

### 2.6 Ações e Filtros

Use hooks sempre que possível:

```php
    add_action('init', 'mytheme_register_post_types');

    add_filter('excerpt_length', function() {
        return 20;
    });
```

> Hooks mantêm o código desacoplado e extensível.

---

### 2.7 HTML Semântico e Acessível

Evite `<div>` genéricas. Use tags semânticas:

```html
    <header></header>
    <main></main>
    <aside></aside>
    <footer></footer>
    <nav aria-label="Menu principal"></nav>
```

Adicione atributos `aria-*`, `role="..."` para melhorar acessibilidade (a11y).

---

### 2.8 Internacionalização

Use funções de tradução:

```php
    echo __("Leia mais", "mytheme");
    _e("Enviar", "mytheme");
```

> Evite strings fixas no HTML ou PHP.

---

### 2.9 Evite Hardcode: Use funções do WordPress

**Errado:**

```php
<img src="/meutema/assets/logo.png">
```

**Certo:**

```php
<img src="<?php echo get_template_directory_uri(); ?>/assets/logo.png">
```

> Use funções como `get_home_url()`, `get_permalink()`, `the_title()` etc.

---

### 2.10 WordPress Coding Standards

Utilize o PHPCS (PHP Code Sniffer) com o padrão WP:

```bash
composer require --dev wp-coding-standards/wpcs
vendor/bin/phpcs --standard=WordPress ./theme/
```

> Isso garante padronização e qualidade no seu código.

---

## ✅ Checklist de Boas Práticas no Código

| Item                      | Prática                                          |
| ------------------------- | ------------------------------------------------ |
| 🧰 [Modularização]        | Modularize o código com `includes/`              |
| 💼 [Organização]          | Use prefixos ou namespaces                       |
| 🔒 [Segurança]            | Escape e sanitize corretamente                   |
| 🔄 [Hooks]                | Use `add_action()` e `add_filter()`              |
| 🌍 [Internacionalização]  | Internacionalize com `__()` e `_e()`             |
| ✅ [Suporte Tema]         | Use `add_theme_support()` corretamente           |
| 🩼 [Flexibilidade]        | Evite hardcode de paths e URLs                   |
| ♿ [Acessibilidade]       | Use HTML semântico e acessível                   |
| 🎯 [Enfileiramento]       | Use `wp_enqueue_style()` e `wp_enqueue_script()` |
| 🧪 [Qualidade]            | Siga o padrão de código com PHPCS                |
---

## 🔧 Ferramentas e Dicas Úteis

* **Theme Check** – Plugin para validar padrões e práticas.
* **Lighthouse (Chrome DevTools)** – Performance, SEO e acessibilidade.
* **WP CLI** – Ferramenta de linha de comando para tarefas WordPress.
* **Poedit / Loco Translate** – Para traduzir strings e criar `.po/.mo`.

---

## 📋 Conclusão

Ao seguir estas boas práticas:

* Seu tema será mais seguro, escalável e performático.
* Você evitará conflitos com plugins e o core do WordPress.
* Terá mais facilidade em manter e adaptar o tema no futuro.
* Seus projetos se beneficiarão de uma base sólida e confiável.

---