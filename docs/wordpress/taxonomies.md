# Taxonomias no WordPress: Guia Técnico Aprofundado

---

## Definição e Conceito Fundamental
Taxonomia = Sistema de classificação hierárquica (do grego "taxis" = organização + "nomia" = método).  
No WordPress, taxonomias são estruturas de dados que permitem agrupar e relacionar conteúdos (posts) através de metadados categorizados chamados "termos".


### Analogia Detalhada
Imagine uma biblioteca digital:
- Posts = Livros individuais
- Taxonomia = Sistema de catalogação (ex: Sistema Dewey)
- Termos = Categorias específicas (ex: 500 - Ciências, 530 - Física)
- Relações = Como os livros são organizados nas estantes

---

## Como o WordPress Implementa Taxonomias

### Arquitetura Técnica
1. Registro de Taxonomia:
   
   Exemplo de código PHP:
   
    ```PHP
        register_taxonomy(
            'plataforma',  // Nome interno (slug)
            'jogo',        // Post type associado
            array(         // Parâmetros de configuração
                'labels' => array(
                    'name' => 'Plataformas',
                    'add_new_item' => 'Adicionar Nova Plataforma'
                ),
                'hierarchical' => true,  // Comportamento hierárquico
                'show_in_rest' => true,   // Disponível no editor Gutenberg
                'rewrite' => array('slug' => 'plataformas-jogos')
            )
        );
    ```

2. Armazenamento em Banco de Dados:
   - wp_terms: Armazena termos (term_id, name, slug)
   - wp_term_taxonomy: Define o tipo de taxonomia (taxonomy)
   - wp_term_relationships: Liga posts a termos
   - wp_termmeta: Metadados adicionais (desde WP 4.4)

### Fluxo de Operação
1. Usuário atribui termos a um post
2. WordPress armazena as relações nas tabelas:
   - wp_term_relationships (object_id + term_taxonomy_id)
   - Atualiza contagem em wp_term_taxonomy (count)
3. Ao acessar uma URL de termo (/plataformas-jogos/playstation/):
   - WordPress identifica a taxonomia pela estrutura de URL
   - Executa uma query para obter todos posts com esse termo
   - Renderiza usando template hierarchy

---

## Taxonomias Nativas vs. Customizadas

### Nativas (Core)
Tipo         | Hierárquico | Obrigatório | Exemplo de Uso
-------------|-------------|-------------|---------------
Categoria    | Sim         | Sim         | Classificação principal (ex: Gêneros Musicais > Rock > Alternativo)
Tag          | Não         | Não         | Micro-classificação (ex: #cover, #ao-vivo, #remasterizado)
Autor        | Implícito   | Sim         | Agrupamento por criador de conteúdo

*Para post type 'post', com fallback para "Sem categoria"

### Customizadas
Quando criar?
- Quando precisa de classificação específica para um post type
- Quando categorias/tags são semanticamente inadequadas
- Quando necessita de comportamento especializado

Exemplo Prático: Videoteca
```PHP
    // Taxonomia para Classificação Etária
    register_taxonomy('classificacao_etaria', 'filme', array(
        'hierarchical' => false,
        'labels' => array(
            'name' => 'Classificação Etária',
            'menu_name' => 'Classif. Etária'
        ),
        'meta_box_cb' => 'classificacao_meta_box' // Callback customizado
    ));

    // Taxonomia para Diretores (hierárquica)
    register_taxonomy('diretores', 'filme', array(
        'hierarchical' => true,
        'labels' => array(
            'name' => 'Diretores',
            'add_new_item' => 'Adicionar Novo Diretor'
        ),
        'rewrite' => array('slug' => 'diretores-filmes')
    ));
```

---

## Operações Avançadas

### 1. Queries Complexas com Taxonomias
```PHP
    // Buscar filmes de ação para maiores de 18 anos
    $query = new WP_Query(array(
        'post_type' => 'filme',
        'tax_query' => array(
            'relation' => 'AND',
            array(
                'taxonomy' => 'genero',
                'field'    => 'slug',
                'terms'    => 'acao'
            ),
            array(
                'taxonomy' => 'classificacao_etaria',
                'field'    => 'name',
                'terms'    => '18+',
                'operator' => '>='
            )
        ),
        'orderby' => 'release_date',
        'posts_per_page' => 12
    ));
```

### 2. Term Meta Avançado
```PHP
    // Registrar campo "Ano de Estreia" para diretores
    register_term_meta('diretores', 'ano_estreia', array(
        'type' => 'number',
        'description' => 'Ano de estreia na direção',
        'single' => true,
        'show_in_rest' => true,
        'sanitize_callback' => 'absint'
    ));

    // Adicionar campo na interface admin
    add_action('diretores_add_form_fields', 'add_diretor_ano_field');
    function add_diretor_ano_field($taxonomy) {
        echo '<div class="form-field">
            <label for="ano_estreia">Ano de Estreia</label>
            <input type="number" name="ano_estreia" id="ano_estreia">
            <p>Primeiro ano como diretor</p>
        </div>';
    }

    // Salvar dados
    add_action('created_diretores', 'save_diretor_meta');
    function save_diretor_meta($term_id) {
        if (isset($_POST['ano_estreia'])) {
            update_term_meta($term_id, 'ano_estreia', sanitize_text_field($_POST['ano_estreia']));
        }
    }
```

### 3. Hierarquias Complexas
Exemplo de estrutura:
```
Gêneros Musicais
├── Popular
│   ├── MPB
│   └── Rock
│       ├── Rock Alternativo
│       └── Heavy Metal
└── Erudito
    ├── Ópera
    └── Sinfônico
```

---

## Performance e Otimização

### Problemas Comuns
1. Slow Queries em taxonomias com >10,000 termos
2. Contagem incorreta de posts por termo
3. Carregamento lento da interface admin

### Soluções Técnicas
```php
// 1. Otimizar contagem de termos
add_filter('update_term_count_cache', '__return_false');
```

```PHP
// 2. Paginação personalizada
function otimizar_query_taxonomia(WP_Query $query) {
    if ($query->is_tax('produto_categoria')) {
        $query->set('posts_per_page', 50);
        $query->set('no_found_rows', true);
        $query->set('update_post_term_cache', false);
    }
}
add_action('pre_get_posts', 'otimizar_query_taxonomia');
```

```SQL
-- 3. Indexação de banco de dados
CREATE INDEX wp_terms_name_idx ON wp_terms(name(20));
CREATE INDEX wp_term_taxonomy_taxonomy_idx ON wp_term_taxonomy(taxonomy);
```

---

## Integração com o Ecossistema WordPress

### 1. REST API
Endpoint: /wp-json/wp/v2/diretores

Exemplo de resposta:

```JSON
{
    "id": 42,
    "name": "Christopher Nolan",
    "slug": "christopher-nolan",
    "meta": {
        "ano_estreia": 1998
    },
    "_links": {
        "wp:post_type": [
            {"href": "https://site.com/wp-json/wp/v2/filmes?diretores=42"}
        ]
    }
}
```


### 2. Editor Gutenberg
Componente React para seleção de termos:

import { useSelect } from '@wordpress/data';
import { TaxonomySelect } from '@wordpress/block-editor';

const DiretorSelector = ({ value, onChange }) => {
    const diretores = useSelect((select) => {
        return select('core').getEntityRecords('taxonomy', 'diretores', { per_page: -1 });
    }, []);
    
    return (
        <TaxonomySelect
            label="Selecione o Diretor"
            taxonomies={diretores}
            value={value}
            onChange={onChange}
        />
    );
};

### 3. WP-CLI
Comandos úteis:
```shell
# Exportar todos termos de uma taxonomia
wp term list diretores --format=csv > diretores.csv

# Importar termos com metadados
wp term import from-csv diretores.csv --meta=ano_estreia

# Recriar contagens
wp term recount all
```

---

## Casos de Uso Complexos

### Taxonomia Polimórfica
```PHP
// Mesma taxonomia com comportamentos diferentes para post types

add_filter('get_terms_args', function($args, $taxonomies) {
    if (in_array('regiao', $taxonomies)) {
        if (is_admin() && get_current_screen()->post_type === 'imovel') {
            $args['meta_query'] = array(
                array(
                    'key' => 'disponivel_imoveis',
                    'value' => '1'
                )
            );
        } elseif (is_admin() && get_current_screen()->post_type === 'evento') {
            $args['meta_query'] = array(
                array(
                    'key' => 'disponivel_eventos',
                    'value' => '1'
                )
            );
        }
    }
    return $args;
}, 10, 2);
```

### Taxonomia Transversal
```PHP
// Taxonomia compartilhada entre múltiplos post types
register_taxonomy('departamentos', array('funcionario', 'projeto', 'equipamento'), array(
    'label' => 'Departamentos',
    'hierarchical' => true,
    'capabilities' => array(
        'manage_terms' => 'manage_departamentos',
        'edit_terms'   => 'edit_departamentos',
        'delete_terms' => 'delete_departamentos',
        'assign_terms' => 'assign_departamentos'
    )
));

// Adicionar campo "Chefe de Departamento"
register_term_meta('departamentos', 'chefe_departamento', array(
    'type' => 'integer',
    'description' => 'ID do funcionário chefe',
    'show_in_rest' => array(
        'schema' => array(
            'type' => 'integer',
            'context' => array('view', 'edit')
        )
    )
));
```

---

## Melhores Práticas e Segurança

### Padrões de Codificação
1. Prefixo único para taxonomias customizadas
    ```PHP
    register_taxonomy('minhaempresa_departamentos', ...);
    ```
   
2. Validação rigorosa de inputs
    ```PHP
    add_filter('pre_insert_term', function($term, $taxonomy) {
        if ('departamentos' === $taxonomy) {
            if (!preg_match('/^[a-z0-9\- ]+$/i', $term)) {
                return new WP_Error('invalid_term', 'Nome inválido');
            }
            if (term_exists($term, $taxonomy)) {
                return new WP_Error('duplicate_term', 'Termo já existe');
            }
        }
        return $term;
    }, 10, 2);
    ```


3. Controle de Capacidades
    ```PHP
    register_taxonomy('projetos_confidenciais', 'documento', array(
        'capabilities' => array(
            'manage_terms' => 'manage_confidential',
            'edit_terms'   => 'edit_confidential',
            'delete_terms' => 'delete_confidential',
            'assign_terms' => 'assign_confidential'
        )
    ));
    ```

### Auditoria e Manutenção
1. Monitorar crescimento de termos:
    ```SQL
    SELECT taxonomy, COUNT(*) as term_count 
    FROM wp_term_taxonomy 
    GROUP BY taxonomy 
    ORDER BY term_count DESC;
    ```
   
2. Limpeza periódica de termos não utilizados:
    ```PHP
    $empty_terms = get_terms(array(
        'taxonomy' => 'tags',
        'hide_empty' => false,
        'fields' => 'ids',
        'number' => 50,
        'meta_query' => array(
            array(
                'key' => 'last_used',
                'value' => strtotime('-1 year'),
                'compare' => '<',
                'type' => 'NUMERIC'
            )
        )
    ));
    
    foreach ($empty_terms as $term_id) {
        wp_delete_term($term_id, 'tags');
    }
    ```

---

## Solução de Problemas Avançados

### Debugging de Taxonomias
1. Verificar registro:
    ```PHP
    print_r(get_taxonomy('diretores'));
    ```

2. Inspecionar termos:
    ```PHP
    $terms = get_terms(array('taxonomy' => 'diretores'));
    echo '<pre>'; print_r($terms); echo '</pre>';
    ```

3. Monitorar queries:
    ```PHP
    add_filter('posts_request', function($sql) {
        if (is_tax('diretores')) {
            error_log('Tax Query: ' . $sql);
        }
        return $sql;
    });
    ```

### Migração de Dados

    ```PHP
    // Migrar de categorias para taxonomia customizada
    $posts = get_posts(array(
        'post_type' => 'filme',
        'numberposts' => -1
    ));

    foreach ($posts as $post) {
        $categories = wp_get_post_categories($post->ID);
        foreach ($categories as $cat_id) {
            $category = get_category($cat_id);
            $new_term = term_exists($category->name, 'genero');
            if (!$new_term) {
                $new_term = wp_insert_term($category->name, 'genero');
            }
            wp_set_post_terms($post->ID, $new_term['term_id'], 'genero', true);
        }
    }
    ```