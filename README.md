# Guia de PostgreSQL

Um guia progressivo de SQL e PostgreSQL, do zero até relacionamentos entre tabelas.

---

# Introdução

## O que é SQL?

**SQL** (*Structured Query Language*, ou Linguagem de Consulta Estruturada) é a linguagem padrão usada para **comunicar com bases de dados relacionais**.

Com SQL conseguimos:

- **Criar** estruturas de dados (tabelas)
- **Inserir** dados
- **Consultar** dados (a parte mais usada no dia a dia)
- **Atualizar** dados existentes
- **Apagar** dados

SQL não é exclusivo de um único sistema — é uma linguagem que a maioria das bases de dados relacionais (PostgreSQL, MySQL, SQL Server, SQLite...) implementa, cada uma com pequenas variações e funcionalidades próprias.

## O que é o PostgreSQL?

**PostgreSQL** (ou apenas "Postgres") é um **Sistema de Gestão de Bases de Dados Relacionais** (SGBD) — ou seja, um programa que guarda os teus dados de forma organizada em **tabelas**, e que entende comandos SQL para manipular esses dados.

### Porque é o PostgreSQL tão usado?

- É **open-source** e gratuito
- É extremamente **robusto e confiável**, mesmo em produção com grandes volumes de dados
- Suporta **tipos de dados avançados** (JSON, arrays, UUID, geolocalização com PostGIS, etc.)
- Segue rigorosamente o **padrão SQL**, o que o torna previsível
- Tem uma comunidade enorme e é usado por empresas como Instagram, Spotify e Apple

### Como os dados são organizados?

```
Base de dados (database)
   └── Tabelas (tables)
         └── Colunas (columns) — definem o tipo de dado
         └── Linhas (rows) — os registos propriamente ditos
```

Por exemplo, numa tabela `users`:

| id | username | email |
|----|----------|-------|
| 1  | Makene   | makenedev@gmail.com |
| 2  | Ana      | ana@email.com |

- `id`, `username` e `email` são **colunas**
- Cada linha (Makene, Ana) é um **registo**

---

# `CREATE TABLE`

É o comando usado para **criar uma nova tabela** dentro da base de dados.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

O que está a acontecer aqui:

- `CREATE TABLE users` → cria uma tabela chamada `users`
- Cada linha dentro dos parênteses define **uma coluna**: `nome_da_coluna TIPO restrições`
- `SERIAL PRIMARY KEY` → cria um `id` que se **auto-incrementa** (1, 2, 3...) e é a chave primária
- `NOT NULL` → obriga a que essa coluna tenha sempre um valor
- `DEFAULT NOW()` → se não passares valor, usa a data/hora atual automaticamente

---

# Tipos de Dados

## Texto

| Tipo | Descrição |
|------|-----------|
| `VARCHAR(n)` | Texto de **tamanho variável**, com limite máximo de `n` caracteres. Ideal para nomes, emails, títulos. |
| `CHAR(n)` | Texto de **tamanho fixo**. Se o texto for menor que `n`, o PostgreSQL preenche com espaços. Usado para casos muito específicos (ex: códigos de país "PT", "BR"). |
| `TEXT` | Texto **sem limite prático de tamanho**. Ideal para descrições, artigos, comentários longos. |

> **Dica:** Na dúvida, usa `VARCHAR(n)` para campos curtos e `TEXT` para conteúdo longo. `CHAR` é raramente necessário.

## Números

| Tipo | Descrição |
|------|-----------|
| `INTEGER` / `INT` | Números inteiros (sem casas decimais) |
| `SERIAL` | Um `INTEGER` que se auto-incrementa — ótimo para IDs |
| `DECIMAL(p, s)` / `NUMERIC(p, s)` | Números com casas decimais exatas (ex: valores monetários). `p` = total de dígitos, `s` = casas decimais |
| `FLOAT` / `REAL` | Números decimais aproximados (evitar em valores financeiros) |

## Data e Hora

| Tipo | Descrição |
|------|-----------|
| `DATE` | Apenas a data (ex: `2026-07-01`) |
| `TIME` | Apenas a hora (ex: `14:30:00`) |
| `TIMESTAMP` | Data **e** hora juntas |

## Outros tipos comuns

| Tipo | Descrição |
|------|-----------|
| `BOOLEAN` | Verdadeiro ou falso (`TRUE` / `FALSE`) |
| `UUID` | Identificador único universal, alternativa ao `SERIAL` |

---

# `INSERT INTO`

Usado para **adicionar dados** a uma tabela.

```sql
INSERT INTO users (username, email)
VALUES ('Makene', 'makenedev@gmail.com');
```

Podes inserir vários registos de uma vez:

```sql
INSERT INTO users (username, email)
VALUES 
    ('Makene', 'makenedev@gmail.com'),
    ('Ana', 'ana@email.com'),
    ('João', 'joao@email.com');
```

> Não precisas de indicar o `id` — como é `SERIAL`, o PostgreSQL gera automaticamente.

---

# Colunas Calculadas

Podes criar **colunas derivadas** dentro de um `SELECT`, calculadas na hora, sem alterar a tabela original.

```sql
SELECT 
    name, 
    population / area AS population_density
FROM cities;
```

Aqui:

- `population / area` é uma **expressão calculada**
- `AS population_density` dá um **nome (alias)** a essa coluna no resultado

Isto é extremamente útil para relatórios e análises sem precisar de guardar dados redundantes na tabela.

---

# Operações e Funções com Strings

O PostgreSQL tem várias funções para manipular texto:

```sql
-- Concatenar textos
SELECT first_name || ' ' || last_name AS full_name FROM users;

-- Converter para maiúsculas / minúsculas
SELECT UPPER(username) FROM users;
SELECT LOWER(username) FROM users;

-- Tamanho do texto
SELECT LENGTH(username) FROM users;

-- Remover espaços em branco nas pontas
SELECT TRIM(username) FROM users;

-- Substituir parte do texto
SELECT REPLACE(email, '@gmail.com', '@outlook.com') FROM users;

-- Extrair parte do texto (substring)
SELECT SUBSTRING(username FROM 1 FOR 3) FROM users;
```

---

# Filtrando Linhas com `SELECT`

O `SELECT` combinado com `WHERE` permite **filtrar** quais linhas queremos ver.

```sql
SELECT * FROM users
WHERE username = 'Makene';
```

Operadores comuns:

```sql
SELECT * FROM products WHERE price > 100;
SELECT * FROM products WHERE price >= 100;
SELECT * FROM products WHERE price != 100;
SELECT * FROM products WHERE category = 'Eletrónica';
```

---

# Cláusulas `WHERE` Compostas

Podes combinar múltiplas condições.

## `AND` / `OR`

```sql
SELECT * FROM products
WHERE category = 'Eletrónica' AND price < 500;

SELECT * FROM products
WHERE category = 'Eletrónica' OR category = 'Informática';
```

## `BETWEEN`

Filtra valores dentro de um intervalo (inclusive):

```sql
SELECT * FROM products
WHERE price BETWEEN 100 AND 500;
```

É equivalente a:

```sql
WHERE price >= 100 AND price <= 500
```

## `IN`

Filtra valores dentro de uma lista de opções:

```sql
SELECT * FROM users
WHERE username IN ('Makene', 'Ana', 'João');
```

## `LIKE`

Filtra por padrões de texto:

```sql
-- Começa com "Ma"
SELECT * FROM users WHERE username LIKE 'Ma%';

-- Contém "an" em qualquer posição
SELECT * FROM users WHERE username LIKE '%an%';
```

## `IS NULL`

```sql
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;
```

---

# `UPDATE` — Atualizar Linhas

```sql
UPDATE users
SET email = 'novo_email@email.com'
WHERE id = 1;
```

> **Atenção:** Se esqueceres o `WHERE`, o `UPDATE` altera **todas as linhas** da tabela. Confirma sempre a condição antes de executar.

Podes atualizar várias colunas ao mesmo tempo:

```sql
UPDATE users
SET username = 'Makene Neto', email = 'makene.neto@email.com'
WHERE id = 1;
```

---

# Relacionamentos entre Tabelas

Numa base de dados relacional, tabelas relacionam-se entre si através de **chaves**. Existem quatro tipos principais de relacionamento:

## 1:1 (Um para Um)

Cada registo de uma tabela corresponde a **exatamente um** registo de outra.

**Exemplo:** `users` e `user_profiles` (cada utilizador tem um único perfil).

```sql
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER UNIQUE REFERENCES users(id),
    bio TEXT
);
```

O `UNIQUE` garante que cada `user_id` só pode aparecer **uma vez** em `user_profiles`.

## 1:N (Um para Muitos)

Um registo de uma tabela pode estar relacionado com **vários** registos de outra.

**Exemplo:** um `user` pode ter vários `posts`, mas cada `post` pertence a apenas um `user`.

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100),
    user_id INTEGER REFERENCES users(id)
);
```

## N:1 (Muitos para Um)

É a **mesma relação** que 1:N, apenas vista do outro lado. Vários `posts` apontam para um único `user`.

## N:N (Muitos para Muitos)

Vários registos de uma tabela podem relacionar-se com vários registos de outra. Isto exige uma **tabela intermediária** (tabela de junção).

**Exemplo:** um `post` pode ter várias `tags`, e uma `tag` pode estar em vários `posts`.

```sql
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id),
    tag_id INTEGER REFERENCES tags(id),
    PRIMARY KEY (post_id, tag_id)
);
```

A tabela `post_tags` é chamada de **tabela pivô** ou **tabela de junção**, e liga as duas entidades.

---

# `ALTER TABLE` — Alterar Colunas

Usado para modificar a estrutura de uma tabela já existente.

## Adicionar uma coluna

```sql
ALTER TABLE users
ADD COLUMN phone VARCHAR(20);
```

## Alterar o tipo de uma coluna

```sql
ALTER TABLE users
ALTER COLUMN phone TYPE VARCHAR(30);
```

## Renomear uma coluna

```sql
ALTER TABLE users
RENAME COLUMN phone TO phone_number;
```

## Definir ou remover valor obrigatório

```sql
ALTER TABLE users
ALTER COLUMN phone_number SET NOT NULL;

ALTER TABLE users
ALTER COLUMN phone_number DROP NOT NULL;
```

---

# `DELETE COLUMN` — Remover Colunas

```sql
ALTER TABLE users
DROP COLUMN phone_number;
```

> Isto remove a coluna **e todos os dados dentro dela**, de forma permanente.

---

# `DROP TABLE` — Remover Tabelas

```sql
DROP TABLE posts;
```

Se quiseres evitar erro caso a tabela não exista:

```sql
DROP TABLE IF EXISTS posts;
```

> **Atenção:** Isto apaga a tabela inteira, estrutura e dados. Não há como desfazer.

Para apagar apenas os **dados**, mantendo a tabela (estrutura) intacta, usa:

```sql
DELETE FROM posts;
-- ou, mais rápido para tabelas grandes:
TRUNCATE TABLE posts;
```

---

# Chaves Primárias e Estrangeiras

## `PRIMARY KEY`

Identifica **de forma única** cada linha de uma tabela. Não pode repetir-se nem ser nula.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);
```

## `FOREIGN KEY`

Cria uma **ligação** entre uma coluna desta tabela e a chave primária de outra tabela, garantindo integridade referencial (não podes referenciar algo que não existe).

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100),
    user_id INTEGER REFERENCES users(id)
);
```

### Adicionar uma Foreign Key a uma tabela já existente

```sql
ALTER TABLE posts
ADD CONSTRAINT fk_user
FOREIGN KEY (user_id) REFERENCES users(id);
```

---

# `ON DELETE` — O que fazer quando o registo referenciado é apagado?

Quando defines uma `FOREIGN KEY`, precisas de decidir **o que acontece** aos registos filhos quando o registo pai é apagado.

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100),
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE
);
```

## Opções disponíveis

| Opção | Comportamento |
|-------|---------------|
| `ON DELETE CASCADE` | Apaga automaticamente os registos filhos quando o pai é apagado |
| `ON DELETE RESTRICT` | Impede a exclusão do pai enquanto existirem filhos relacionados (comportamento padrão) |
| `ON DELETE SET NULL` | Define a coluna `user_id` como `NULL` nos filhos, mantendo os registos |
| `ON DELETE SET DEFAULT` | Define a coluna com o seu valor padrão (`DEFAULT`) |
| `ON DELETE NO ACTION` | Semelhante ao `RESTRICT`, mas a verificação pode ser adiada dentro de uma transação |

### Exemplos práticos

```sql
-- Se apagar o user, todos os seus posts são apagados também
user_id INTEGER REFERENCES users(id) ON DELETE CASCADE

-- Se apagar o user, o post permanece, mas fica "órfão" (user_id = NULL)
user_id INTEGER REFERENCES users(id) ON DELETE SET NULL

-- Impede apagar o user enquanto ele tiver posts
user_id INTEGER REFERENCES users(id) ON DELETE RESTRICT
```

> **Como escolher?** Pergunta-te: *"Se o pai desaparecer, o filho ainda faz sentido sozinho?"* Se não fizer sentido (ex: um post sem autor), usa `CASCADE`. Se o filho deve continuar a existir por outras razões, usa `SET NULL`.

---

# SQL JOINs (PostgreSQL)

Os `JOINs` permitem **combinar dados de duas ou mais tabelas** através de um relacionamento entre elas.

Imagine as seguintes tabelas:

### `users`

| id | username |
|----|----------|
| 1 | Makene |
| 2 | Ana |
| 3 | João |

### `posts`

| id | title | user_id |
|----|-------|---------|
| 1 | SQL | 1 |
| 2 | React | 1 |
| 3 | Docker | 2 |
| 4 | Linux | 4 |

> **Observação:** O post `Linux` possui `user_id = 4`, mas esse usuário não existe na tabela `users`.

---

# A cláusula `ON`

Antes de aprender os tipos de `JOIN`, entenda o papel do `ON`.

```sql
SELECT *
FROM users
JOIN posts
ON users.id = posts.user_id;
```

O `ON` define **como as tabelas devem ser relacionadas**.

Neste caso:

```sql
users.id = posts.user_id
```

Significa:

> Junte um usuário ao seu respectivo post quando o `id` do usuário for igual ao `user_id` do post.

Sem o `ON`, o banco de dados não sabe quais registros devem ser combinados.

---

# 1. INNER JOIN

Retorna **apenas os registros que possuem correspondência nas duas tabelas**.

```sql
SELECT *
FROM users
INNER JOIN posts
ON users.id = posts.user_id;
```

Resultado:

| username | title |
|----------|-------|
| Makene | SQL |
| Makene | React |
| Ana | Docker |

**Não aparecem:**

- João (não possui posts)
- Linux (não possui usuário)

### Quando usar?

Sempre que você quiser apenas registros relacionados.

Exemplos:

- Usuários e seus posts
- Clientes e seus pedidos
- Produtos e suas categorias

---

# 2. LEFT JOIN

Retorna **todos os registros da tabela da esquerda** (`FROM`).

Se não existir correspondência, os campos da outra tabela serão `NULL`.

```sql
SELECT *
FROM users
LEFT JOIN posts
ON users.id = posts.user_id;
```

Resultado:

| username | title |
|----------|-------|
| Makene | SQL |
| Makene | React |
| Ana | Docker |
| João | NULL |

### Quando usar?

Quando a tabela da esquerda é a principal e você não quer perder nenhum registro.

Exemplos:

- Todos os usuários, mesmo sem posts.
- Todos os clientes, mesmo sem pedidos.

---

# 3. RIGHT JOIN

Retorna **todos os registros da tabela da direita** (`JOIN`).

```sql
SELECT *
FROM users
RIGHT JOIN posts
ON users.id = posts.user_id;
```

Resultado:

| username | title |
|----------|-------|
| Makene | SQL |
| Makene | React |
| Ana | Docker |
| NULL | Linux |

### Quando usar?

Na prática, raramente.

Normalmente basta inverter a ordem das tabelas e utilizar um `LEFT JOIN`.

---

# 4. FULL JOIN

Retorna **todos os registros de ambas as tabelas**.

Quando não existir correspondência, o banco preencherá os campos com `NULL`.

```sql
SELECT *
FROM users
FULL JOIN posts
ON users.id = posts.user_id;
```

Resultado:

| username | title |
|----------|-------|
| Makene | SQL |
| Makene | React |
| Ana | Docker |
| João | NULL |
| NULL | Linux |

### Quando usar?

Ideal para:

- Auditorias
- Comparação entre tabelas
- Sincronização de dados
- Encontrar registros sem correspondência

---

# A ordem das tabelas importa?

**Sim.**

A tabela escrita após o `FROM` é a **tabela principal**.

## Exemplo 1

```sql
FROM users
LEFT JOIN posts
```

Leitura:

> Quero **todos os usuários** e, se existirem, seus posts.

---

## Exemplo 2

```sql
FROM posts
LEFT JOIN users
```

Leitura:

> Quero **todos os posts** e, se existirem, seus respectivos usuários.

Perceba que apenas trocar a ordem muda completamente o resultado.

---

# A ordem importa para todos os JOINs?

| JOIN | A ordem altera o resultado? |
|------|------------------------------|
| INNER JOIN | ❌ Não |
| LEFT JOIN | ✅ Sim |
| RIGHT JOIN | ✅ Sim |
| FULL JOIN | ❌ Não (apenas a ordem das colunas) |

---

# Como decorar

Sempre pense assim:

```sql
FROM tabela_principal
LEFT JOIN outra_tabela
```

Leitura:

> Quero **todos os registros da tabela principal**.

A tabela principal é sempre aquela escrita depois do `FROM`.

---

# Resumo

| JOIN | Retorna | Uso mais comum |
|------|----------|----------------|
| `INNER JOIN` | Apenas registros relacionados | Buscar dados relacionados |
| `LEFT JOIN` | Todos da tabela da esquerda | Manter todos os registros da tabela principal |
| `RIGHT JOIN` | Todos da tabela da direita | Pouco utilizado |
| `FULL JOIN` | Todos os registros das duas tabelas | Comparações e auditorias |

---

# Regra de ouro

Antes de escrever um `JOIN`, faça duas perguntas:

1. **Qual é a minha tabela principal?**
2. **Quero manter todos os registros dela ou apenas os que possuem relacionamento?**

Se responder essas duas perguntas, ficará fácil escolher o `JOIN` correto.
