
# Guia Completo do WP_Query no WordPress

O `WP_Query` é uma classe do WordPress usada para buscar posts com critérios personalizados. É a base para consultas personalizadas, loops avançados e criação de páginas dinâmicas.

---

## 📌 Índice

- [Introdução](#introdução)
- [Estrutura Básica](#estrutura-básica)
- [Principais Parâmetros](#principais-parâmetros)
  - [Posts](#posts)
  - [Autores](#autores)
  - [Categorias e Taxonomias](#categorias-e-taxonomias)
  - [Meta Query (Campos personalizados)](#meta-query-campos-personalizados)
  - [Date Query](#date-query)
  - [Ordenação](#ordenação)
- [Exemplo Completo de Loop](#exemplo-completo-de-loop)
- [Resetando a Query](#resetando-a-query)
- [Otimização e Boas Práticas](#otimização-e-boas-práticas)

---

## Introdução

O `WP_Query` permite criar loops personalizados diferentes do loop principal do WordPress. Ideal para criar listas de posts, sliders, carrosséis, postagens relacionadas, etc.

```php
$query = new WP_Query( $args );
```

---

## Estrutura Básica

```php
$args = [
    'post_type' => 'post',
    'posts_per_page' => 10,
];

$query = new WP_Query( $args );

if ( $query->have_posts() ) {
    while ( $query->have_posts() ) {
        $query->the_post();
        // Seu conteúdo aqui
    }
    wp_reset_postdata();
}
```

---

## Principais Parâmetros

### Posts

| Parâmetro | Descrição |
|----------|-----------|
| `post_type` | Tipo de post (ex: `post`, `page`, `custom_post_type`) |
| `posts_per_page` | Quantidade de posts retornados |
| `paged` | Suporte à paginação |
| `post__in` | Array de IDs para incluir |
| `post__not_in` | Array de IDs para excluir |
| `name` | Slug do post |

---

### Autores

```php
'author'       => 1, // ID
'author_name'  => 'nome-do-autor'
```

---

### Categorias e Taxonomias

```php
'category_name' => 'noticias',
'cat'           => 3, // ID
'tag'           => 'destaque',

'tax_query' => [
    [
        'taxonomy' => 'produto_categoria',
        'field'    => 'slug',
        'terms'    => ['digital', 'fisico'],
        'operator' => 'IN',
    ]
]
```

---

### Meta Query (Campos personalizados)

```php
'meta_query' => [
    [
        'key'     => 'preco',
        'value'   => 100,
        'compare' => '>=',
        'type'    => 'NUMERIC',
    ]
]
```

---

### Date Query

```php
'date_query' => [
    [
        'after'     => '2024-01-01',
        'before'    => '2024-12-31',
        'inclusive' => true,
    ],
]
```

---

### Ordenação

```php
'orderby' => 'date', // title, meta_value, rand, etc
'order'   => 'DESC', // ASC
```

Ordenando por campo personalizado:

```php
'orderby'  => 'meta_value_num',
'meta_key' => 'preco',
```

---

## Exemplo Completo de Loop

```php
$args = [
    'post_type'      => 'produto',
    'posts_per_page' => 6,
    'orderby'        => 'meta_value_num',
    'meta_key'       => 'preco',
    'order'          => 'ASC',
    'meta_query'     => [
        [
            'key'     => 'disponivel',
            'value'   => 'sim',
            'compare' => '=',
        ]
    ]
];

$query = new WP_Query( $args );

if ( $query->have_posts() ) {
    echo '<ul>';
    while ( $query->have_posts() ) {
        $query->the_post();
        echo '<li>' . get_the_title() . ' - R$' . get_post_meta( get_the_ID(), 'preco', true ) . '</li>';
    }
    echo '</ul>';
    wp_reset_postdata();
} else {
    echo 'Nenhum produto encontrado.';
}
```

---

## Resetando a Query

Sempre use `wp_reset_postdata()` após loops personalizados para restaurar os dados do post global original.

---

## Otimização e Boas Práticas

- Use `'fields' => 'ids'` quando quiser apenas os IDs dos posts.
- Evite `'posts_per_page' => -1` em sites com muitos posts.
- Combine filtros (`meta_query`, `tax_query`) com `relation => 'AND'` ou `OR`.
- Não altere a query global diretamente sem necessidade.
- Prefira `WP_Query` ao invés de `query_posts()`.