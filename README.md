# Guia de PostgreSQL 🐘

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
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

O que está a acontecer aqui:

- `CREATE TABLE users` → cria uma tabela chamada `users`
- Cada linha dentro dos parênteses define **uma coluna**: `nome_da_coluna TIPO restrições`
- `INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY` → cria um `id` que se **auto-incrementa** (1, 2, 3...) e é a chave primária
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
| `GENERATED ALWAYS AS IDENTITY` | Faz um `INTEGER` se auto-incrementar automaticamente — ótimo para IDs. É a forma moderna e recomendada pelo padrão SQL (substitui o antigo `SERIAL`) |
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
| `UUID` | Identificador único universal, alternativa ao `GENERATED ALWAYS AS IDENTITY` |

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

> Não precisas de indicar o `id` — como é `GENERATED ALWAYS AS IDENTITY`, o PostgreSQL gera automaticamente.

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
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
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
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
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
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
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

# Constraints na Criação vs. Depois da Tabela Criada

Já vimos constraints como `NOT NULL`, `DEFAULT` e `UNIQUE` sendo definidas **dentro** do `CREATE TABLE`. Mas todas elas também podem ser adicionadas (ou removidas) **depois** de a tabela já existir, com `ALTER TABLE`. É útil comparar os dois lados lado a lado.

## `DEFAULT`

Define um valor **automático** para a coluna quando nenhum valor é passado no `INSERT`.

```sql
-- Ao criar a tabela
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    department VARCHAR(50) NOT NULL,
    price INTEGER DEFAULT 999,
    weight INTEGER
);
```

```sql
-- Depois de a tabela já existir
ALTER TABLE products
ALTER COLUMN price SET DEFAULT 999;
```

> 💡 **Dica:** para remover um `DEFAULT` já definido, usa `ALTER TABLE products ALTER COLUMN price DROP DEFAULT;`.

## `NOT NULL`

Obriga a que a coluna tenha sempre um valor — não pode ficar vazia (`NULL`).

```sql
-- Ao criar a tabela
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    price INTEGER NOT NULL,
    weight INTEGER
);
```

```sql
-- Depois de a tabela já existir
ALTER TABLE products
ALTER COLUMN price SET NOT NULL;
```

> ⚠️ **Atenção:** para conseguires aplicar `SET NOT NULL` numa coluna já existente, todas as linhas atuais têm de já ter um valor preenchido nessa coluna. Se houver algum `NULL` na tabela, o PostgreSQL rejeita o comando.

## `UNIQUE`

Garante que **não existem valores repetidos** nessa coluna entre as várias linhas da tabela.

```sql
-- Ao criar a tabela
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE,
    department VARCHAR(50),
    price INTEGER,
    weight INTEGER
);
```

```sql
-- Depois de a tabela já existir
ALTER TABLE products
ADD UNIQUE (name);
```

> 💡 **Dica:** também podes dar um nome à constraint, o que ajuda a identificá-la depois (ex: para a remover): `ALTER TABLE products ADD CONSTRAINT products_name_unique UNIQUE (name);`.

## `UNIQUE` em várias colunas (Multi-column Uniqueness)

Por vezes a unicidade não faz sentido numa coluna sozinha, mas sim na **combinação** de várias colunas. Um exemplo clássico: numa tabela `enrollments`, o mesmo `student_id` pode aparecer várias vezes (um aluno em várias disciplinas) e o mesmo `course_id` também (uma disciplina com vários alunos) — mas o **par** `(student_id, course_id)` não se pode repetir, porque isso significaria o aluno inscrito duas vezes na mesma disciplina.

```sql
-- Ao criar a tabela
CREATE TABLE enrollments (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    student_id INTEGER REFERENCES students(id),
    course_id INTEGER REFERENCES courses(id),
    UNIQUE (student_id, course_id)
);
```

```sql
-- Depois de a tabela já existir
ALTER TABLE enrollments
ADD UNIQUE (student_id, course_id);
```

> 💡 **Dica:** tal como no `UNIQUE` de uma coluna, também podes nomear a constraint para facilitar a gestão depois: `ALTER TABLE enrollments ADD CONSTRAINT enrollments_student_course_unique UNIQUE (student_id, course_id);`.

> ⚠️ **Atenção:** `UNIQUE (student_id, course_id)` é diferente de `UNIQUE (student_id), UNIQUE (course_id)` separados. A versão combinada só bloqueia a repetição do **par exato**; cada coluna sozinha continua a poder repetir-se à vontade.

**Regra de ouro:** usa `UNIQUE` numa coluna quando o valor tem de ser único sozinho (ex: `email`); usa `UNIQUE` em várias colunas quando é a **combinação** delas que representa um registo que não pode duplicar-se (ex: um aluno não pode inscrever-se duas vezes na mesma disciplina).

## `CHECK`

Garante que os valores de uma coluna **respeitam uma condição** definida por ti. Se um `INSERT` ou `UPDATE` tentar gravar um valor que viole a condição, o PostgreSQL rejeita o comando.

```sql
-- Ao criar a tabela
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    department VARCHAR(50) NOT NULL,
    price INTEGER CHECK (price > 0),
    weight INTEGER
);
```

```sql
-- Depois de a tabela já existir
ALTER TABLE products
ADD CHECK (price > 0);
```

> 💡 **Dica:** tal como as outras constraints, também podes nomear o `CHECK`, o que ajuda a identificá-lo depois (ex: para o remover): `ALTER TABLE products ADD CONSTRAINT products_price_positive CHECK (price > 0);`.

> ⚠️ **Atenção:** para conseguires adicionar um `CHECK` a uma tabela já existente, todas as linhas atuais têm de já respeitar a condição. Se houver algum valor que viole a regra (ex: um `price` negativo), o PostgreSQL rejeita o comando.

## `CHECK` em várias colunas (Multi-column CHECK)

Tal como o `UNIQUE`, o `CHECK` também pode envolver **mais do que uma coluna** na mesma condição — útil quando a regra de negócio depende da **relação** entre dois (ou mais) valores da mesma linha, e não de uma coluna isolada.

**Exemplo:** numa tabela `products`, o `price` de venda tem de ser sempre maior que o `cost` (custo), senão a empresa vende com prejuízo.

```sql
-- Ao criar a tabela
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    cost INTEGER NOT NULL,
    price INTEGER NOT NULL,
    CHECK (price > cost)
);
```

```sql
-- Depois de a tabela já existir
ALTER TABLE products
ADD CHECK (price > cost);
```

> 💡 **Dica:** também podes combinar várias condições numa só constraint com `AND` / `OR`: `CHECK (price > cost AND price > 0)`.

Outro exemplo comum: garantir que uma `date_end` nunca é anterior a uma `date_start`:

```sql
ALTER TABLE bookings
ADD CHECK (date_end >= date_start);
```

> ⚠️ **Atenção:** tal como no `CHECK` de uma coluna, ao adicionares a uma tabela já existente, todas as linhas atuais têm de já respeitar a condição — senão o PostgreSQL rejeita o comando.

**Regra de ouro:** usa `CHECK` numa coluna quando a regra depende só dela (ex: `price > 0`); usa `CHECK` em várias colunas quando a regra depende da **relação** entre elas (ex: `price > cost`, `date_end >= date_start`).

## `CHECK` não funciona entre várias linhas

Uma limitação importante: o `CHECK` só consegue avaliar valores **dentro da mesma linha**. Não há forma de escrever um `CHECK` que compare uma linha com outras linhas da tabela (ex: "o `price` desta linha não pode ser maior que a média de todos os `price`", ou "só pode haver 1 produto marcado como `is_featured = true`").

```sql
-- ❌ Isto NÃO funciona — CHECK não pode olhar para outras linhas
ALTER TABLE products
ADD CHECK (price < (SELECT AVG(price) FROM products));
-- Erro: subqueries não são permitidas dentro de CHECK
```

### Alternativas quando precisas de validar entre linhas

| Situação | Alternativa |
|----------|-------------|
| Impedir combinações repetidas (ex: um aluno não pode inscrever-se 2x na mesma disciplina) | `UNIQUE` em várias colunas |
| Impedir sobreposição de intervalos (ex: duas reservas não podem ocupar o mesmo quarto ao mesmo tempo) | `EXCLUDE` constraint (usa `btree_gist`) |
| Validações mais complexas envolvendo várias linhas (ex: "só pode haver 1 produto em destaque") | `TRIGGER` que corre um `SELECT` antes do `INSERT`/`UPDATE` e rejeita se a condição falhar |

Exemplo simples de `TRIGGER` para o caso do "só 1 produto em destaque":

```sql
CREATE OR REPLACE FUNCTION check_single_featured()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.is_featured = TRUE AND
       EXISTS (SELECT 1 FROM products WHERE is_featured = TRUE AND id != NEW.id) THEN
        RAISE EXCEPTION 'Já existe um produto marcado como destaque';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_single_featured
BEFORE INSERT OR UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION check_single_featured();
```

> **Regra de ouro:** `CHECK` é para regras **dentro da própria linha**. Assim que a regra depende de comparar com outras linhas da tabela, `CHECK` deixa de servir — precisas de `UNIQUE`, `EXCLUDE` ou um `TRIGGER`.

## Resumo (Constraints)

| Constraint | Na criação da tabela | Depois de criada |
|------------|------------------------|---------------------|
| `DEFAULT` | `price INTEGER DEFAULT 999` | `ALTER COLUMN price SET DEFAULT 999;` |
| `NOT NULL` | `price INTEGER NOT NULL` | `ALTER COLUMN price SET NOT NULL;` |
| `UNIQUE` | `name VARCHAR(50) UNIQUE` | `ADD UNIQUE (name);` |
| `UNIQUE` (multi-coluna) | `UNIQUE (student_id, course_id)` | `ADD UNIQUE (student_id, course_id);` |
| `CHECK` | `price INTEGER CHECK (price > 0)` | `ADD CHECK (price > 0);` |

**Regra de ouro:** o padrão é sempre o mesmo — `ALTER TABLE tabela ALTER COLUMN coluna SET ...` para `DEFAULT` e `NOT NULL`, e `ALTER TABLE tabela ADD ...` para constraints como `UNIQUE`, `CHECK` e `FOREIGN KEY`, que envolvem a tabela como um todo em vez de só um atributo da coluna.

---

# `DELETE COLUMN` — Remover Colunas

```sql
ALTER TABLE users
DROP COLUMN phone_number;
```

> Isto remove a coluna **e todos os dados dentro dela**, de forma permanente.

---

# `TRUNCATE` — Esvaziar uma Tabela

`TRUNCATE` apaga **todos os dados** de uma tabela, mantendo a estrutura (colunas, tipos, constraints) intacta. É o equivalente a um `DELETE FROM tabela;` sem `WHERE`, mas muito mais rápido.

```sql
TRUNCATE TABLE products;
```

> O `TABLE` é opcional — `TRUNCATE products;` funciona exatamente da mesma forma.

## `TRUNCATE` vs `DELETE`

| | `DELETE FROM` | `TRUNCATE` |
|---|----------------|------------|
| Remove dados | Sim, linha a linha | Sim, a tabela inteira de uma vez |
| Aceita `WHERE` | Sim, pode apagar só algumas linhas | Não, apaga sempre tudo |
| Velocidade | Mais lento (regista cada linha apagada) | Muito mais rápido (não regista linha a linha) |
| Reinicia `IDENTITY`/`SERIAL` | Não, por padrão | Não, a menos que uses `RESTART IDENTITY` |
| Aciona `TRIGGER`s por linha | Sim | Não (só triggers a nível de `TRUNCATE`) |

> ⚠️ **Atenção:** ao contrário do `DELETE`, `TRUNCATE` **não aceita `WHERE`** — apaga sempre a tabela inteira. Se precisares de apagar só algumas linhas, usa `DELETE FROM tabela WHERE ...`.

## `RESTART IDENTITY`

Por padrão, `TRUNCATE` apaga os dados mas **não reinicia** o contador de `GENERATED ALWAYS AS IDENTITY` / `SERIAL`. Se inserires um novo registo depois, o `id` continua a partir de onde ficou.

```sql
-- Apaga os dados, mas o próximo id continua a sequência anterior
TRUNCATE TABLE products;

-- Apaga os dados E reinicia o id a partir de 1
TRUNCATE TABLE products RESTART IDENTITY;
```

## `CASCADE`

Se outras tabelas tiverem `FOREIGN KEY` a apontar para a tabela que estás a truncar, o PostgreSQL recusa o comando por padrão (para não deixar referências "órfãs"). `CASCADE` resolve isso, truncando também as tabelas dependentes.

```sql
TRUNCATE TABLE users CASCADE;
```

> ⚠️ **Atenção:** `CASCADE` aqui também apaga os dados das tabelas relacionadas (ex: `posts`, se `posts.user_id` referenciar `users.id`). Usa com cuidado — o efeito é em cadeia.

**Regra de ouro:** usa `DELETE FROM ... WHERE` quando precisas de apagar linhas específicas; usa `TRUNCATE` quando queres esvaziar a tabela inteira rapidamente, mantendo a estrutura para continuar a usá-la depois.

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
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR(50)
);
```

## `FOREIGN KEY`

Cria uma **ligação** entre uma coluna desta tabela e a chave primária de outra tabela, garantindo integridade referencial (não podes referenciar algo que não existe).

```sql
CREATE TABLE posts (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
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
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
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

# Resumo (JOINs)

| JOIN | Retorna | Uso mais comum |
|------|----------|----------------|
| `INNER JOIN` | Apenas registros relacionados | Buscar dados relacionados |
| `LEFT JOIN` | Todos da tabela da esquerda | Manter todos os registros da tabela principal |
| `RIGHT JOIN` | Todos da tabela da direita | Pouco utilizado |
| `FULL JOIN` | Todos os registros das duas tabelas | Comparações e auditorias |

---

# * Regra de ouro (JOINs)

Antes de escrever um `JOIN`, faça duas perguntas:

1. **Qual é a minha tabela principal?**
2. **Quero manter todos os registros dela ou apenas os que possuem relacionamento?**

Se responder essas duas perguntas, ficará fácil escolher o `JOIN` correto.

---

# Agregação e Agrupamento

Até agora vimos como buscar e filtrar linhas individuais. Mas e quando queremos **resumir** dados — tipo "quantos posts cada usuário tem?" ou "qual a nota média das reviews?". Para isso existem as **funções de agregação** e o `GROUP BY`.

## Funções de agregação

São funções que pegam várias linhas e retornam **um único valor**.

| Função | O que faz |
|--------|-----------|
| `COUNT()` | conta quantas linhas |
| `SUM()` | soma valores |
| `AVG()` | calcula a média |
| `MIN()` | pega o menor valor |
| `MAX()` | pega o maior valor |

Exemplo simples, sem agrupamento — aplicando à tabela toda:

```sql
SELECT COUNT(*) FROM posts;
-- quantos posts existem no total

SELECT AVG(price) FROM products;
-- preço médio de todos os produtos
```

## `GROUP BY`: agregando por categoria

Sozinhas, as funções de agregação resumem a tabela inteira. O `GROUP BY` permite agrupar por uma coluna e aplicar a agregação **dentro de cada grupo**.

```sql
SELECT user_id, COUNT(*)
FROM posts
GROUP BY user_id;
```

Isso responde: "quantos posts cada usuário tem?" — uma linha de resultado por `user_id`.

**Regra de ouro:** toda coluna que aparece no `SELECT` (e não está dentro de uma função de agregação) **precisa** estar no `GROUP BY`. O PostgreSQL não sabe qual valor mostrar se houver várias linhas por grupo e a coluna não estiver agrupada nem agregada.

```sql
-- ❌ Erro: title não está agregada nem no GROUP BY
SELECT user_id, title, COUNT(*)
FROM posts
GROUP BY user_id;

-- ✅ Certo
SELECT user_id, COUNT(*) AS total_posts
FROM posts
GROUP BY user_id;
```

## Combinando com `JOIN`

Isso fica ainda mais útil junto com `JOIN` — por exemplo, número de posts por usuário, já mostrando o nome:

```sql
SELECT users.username, COUNT(posts.id) AS total_posts
FROM users
JOIN posts ON users.id = posts.user_id
GROUP BY users.username;
```

## `HAVING`: filtrando grupos

O `WHERE` filtra **linhas antes** de agrupar. Mas se quisermos filtrar **depois** de agregar — por exemplo, "só usuários com mais de 2 posts" — usamos o `HAVING`.

```sql
SELECT user_id, COUNT(*) AS total_posts
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 2;
```

**Por que não usar `WHERE COUNT(*) > 2`?** Porque o `WHERE` é avaliado antes do agrupamento acontecer — nesse momento o PostgreSQL ainda não calculou o `COUNT()`. O `HAVING` roda depois do `GROUP BY`, então já tem acesso aos valores agregados.

## Ordem de execução (importante para entender de verdade)

O SQL é escrito numa ordem, mas o banco executa noutra:

```
FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

Isso explica por que:

- `WHERE` não pode usar aliases criados no `SELECT`
- `HAVING` pode usar funções de agregação, mas `WHERE` não
- `ORDER BY` pode usar aliases do `SELECT`, porque roda por último

## Exemplo completo juntando tudo

```sql
SELECT users.username, COUNT(posts.id) AS total_posts
FROM users
JOIN posts ON users.id = posts.user_id
GROUP BY users.username
HAVING COUNT(posts.id) > 1
ORDER BY total_posts DESC;
```

> "Mostre cada usuário com mais de 1 post, junto com o total de posts, ordenado do maior para o menor."

---

# Resumo (Agregação e Agrupamento)

| Cláusula / Função | Papel |
|--------------------|-------|
| `COUNT / SUM / AVG / MIN / MAX` | Resume várias linhas num único valor |
| `GROUP BY` | Divide os resultados em grupos, aplicando a agregação em cada um |
| `HAVING` | Filtra grupos **depois** da agregação (o `WHERE` filtra linhas **antes**) |

**Regra de ouro:** toda coluna "solta" no `SELECT` (fora de uma função de agregação) tem que estar no `GROUP BY`.

---

# Ordenando e Limitando Resultados

Falta controlar **em que ordem** os resultados aparecem e **quantos** queremos ver. Para isso existem `ORDER BY`, `LIMIT` e `OFFSET`.

## `ORDER BY`

Por padrão, o PostgreSQL **não garante nenhuma ordem** nos resultados. O `ORDER BY` ordena as linhas por uma ou mais colunas.

```sql
SELECT * FROM products
ORDER BY price;
```

`ASC` (crescente, padrão) e `DESC` (decrescente):

```sql
SELECT * FROM products
ORDER BY price DESC;
```

Pode ordenar por várias colunas. A segunda só serve para **desempatar** quando a primeira tiver valores iguais:

```sql
SELECT * FROM users
ORDER BY username ASC, created_at DESC;
```

Como o `ORDER BY` roda **depois** do `SELECT`, pode usar aliases criados nele:

```sql
SELECT username, COUNT(posts.id) AS total_posts
FROM users
JOIN posts ON users.id = posts.user_id
GROUP BY username
ORDER BY total_posts DESC;
```

---

## `LIMIT`

Corta o resultado, devolvendo apenas as primeiras N linhas.

```sql
SELECT * FROM products
ORDER BY price DESC
LIMIT 5;
```

> `LIMIT` sem `ORDER BY` não faz sentido, já que a ordem das linhas não é garantida. Use sempre os dois juntos.

---

## `OFFSET`

Ignora as primeiras N linhas antes de retornar o resultado.

```sql
SELECT * FROM products
ORDER BY price ASC
OFFSET 10;
```

### Paginação: `LIMIT` + `OFFSET`

Juntos, formam a base da paginação — mostrar resultados em "páginas".

```sql
SELECT * FROM products
ORDER BY id ASC
LIMIT 10 OFFSET 20;
```

> Pula os primeiros 20 registos e devolve os 10 seguintes: página 3, com 10 itens por página.

```
OFFSET = (página - 1) * itens_por_página
```

| Página | LIMIT | OFFSET |
|--------|-------|--------|
| 1 | 10 | 0 |
| 2 | 10 | 10 |
| 3 | 10 | 20 |

---

## Exemplo completo

```sql
SELECT users.username, COUNT(posts.id) AS total_posts
FROM users
JOIN posts ON users.id = posts.user_id
GROUP BY users.username
HAVING COUNT(posts.id) > 1
ORDER BY total_posts DESC
LIMIT 5
OFFSET 0;
```

> Os 5 usuários com mais de 1 post, do maior para o menor.

---

# UNION, UNION ALL, INTERSECT, INTERSECT ALL, EXCEPT e EXCEPT ALL

Até agora combinámos tabelas na **horizontal** com `JOIN` — juntando colunas de tabelas diferentes numa mesma linha. Os operadores de conjunto fazem o oposto: combinam resultados na **vertical**, empilhando as linhas de duas (ou mais) queries `SELECT`.

Imagina que tens duas tabelas separadas de contactos:

### `clientes`

| nome   | email                |
|--------|----------------------|
| Makene | makenedev@gmail.com  |
| Ana    | ana@email.com        |

### `fornecedores`

| nome | email               |
|------|---------------------|
| Ana  | ana@email.com       |
| João | joao@email.com      |

## Regras antes de começar

Para combinar duas queries com estes operadores, ambas precisam:

- Ter o **mesmo número de colunas**
- Ter colunas com **tipos compatíveis**, na mesma ordem (a 1ª coluna da query 1 é comparada com a 1ª da query 2, e assim por diante)

Os **nomes das colunas** no resultado final vêm sempre da **primeira** query. Se a segunda usar nomes diferentes, isso não importa — só a posição e o tipo contam.

---

# 1. `UNION`

Junta os resultados de duas queries, **removendo duplicados**.

```sql
SELECT nome, email FROM clientes
UNION
SELECT nome, email FROM fornecedores;
```

Resultado:

| nome   | email                |
|--------|----------------------|
| Makene | makenedev@gmail.com  |
| Ana    | ana@email.com        |
| João   | joao@email.com       |

Repara que "Ana" aparece nas duas tabelas, mas só sai **uma vez** no resultado — o `UNION` compara linha a linha e descarta repetições.
> UNION — usa quando queres juntar os resultados de duas queries num único conjunto, e não te interessa ter linhas duplicadas. Pensa nisto: se tens uma tabela de clientes_luanda e outra de clientes_benguela, e queres uma lista única de todos os emails de contacto sem repetição, UNION resolve isso. Ele remove duplicados automaticamente, o que implica um custo — o Postgres tem de ordenar ou fazer hash nos resultados para detectar e eliminar as repetições.

## `UNION ALL`

Faz a mesma junção, mas **mantém os duplicados**.

```sql
SELECT nome, email FROM clientes
UNION ALL
SELECT nome, email FROM fornecedores;
```

Resultado:

| nome   | email                |
|--------|----------------------|
| Makene | makenedev@gmail.com  |
| Ana    | ana@email.com        |
| Ana    | ana@email.com        |
| João   | joao@email.com       |

**Porque escolher `UNION ALL` em vez de `UNION`?** Desempenho. O `UNION` precisa de ordenar e comparar todas as linhas para eliminar duplicados — um trabalho extra que o `UNION ALL` não faz. Se souberes que não há sobreposição entre as queries (ou se duplicados não forem um problema), usa sempre `UNION ALL`.

> UNION ALL — usa quando queres a mesma junção de resultados, mas sabes que não vão existir duplicados (ou não te importas se existirem). É mais rápido que UNION porque salta o passo de deduplicação. Regra prática: se estás a combinar dados de fontes que já são mutuamente exclusivas (ex: pedidos de 2024 UNION pedidos de 2025), usa ALL — não há motivo para pagar o custo de verificar duplicados que nunca vão aparecer.

---

# 2. `INTERSECT`

Retorna **apenas as linhas que existem nos dois resultados** ao mesmo tempo.

```sql
SELECT nome, email FROM clientes
INTERSECT
SELECT nome, email FROM fornecedores;
```

Resultado:

| nome | email          |
|------|----------------|
| Ana  | ana@email.com  |

Só "Ana" aparece nas duas tabelas, então é a única linha que sobrevive.

> INTERSECT — usa quando precisas de saber que linhas aparecem nas duas queries ao mesmo tempo. É útil para perguntas do tipo "quais clientes compraram tanto no mês passado como este mês?" — pegas na lista de IDs de cada mês e fazes a intersecção. O resultado dá-te só o que é comum, sem duplicados.

## `INTERSECT ALL`

A versão `ALL` também considera **quantas vezes** cada linha aparece em cada lado, mantendo o menor número de repetições entre as duas queries.

Exemplo: se "Ana" aparecer 3 vezes em `clientes` e 2 vezes em `fornecedores`, o `INTERSECT ALL` devolve "Ana" **2 vezes** (o mínimo entre os dois).

> INTERSECT ALL — a diferença aqui é sutil e tem a ver com quantidade. Se um cliente aparece 3 vezes na primeira query e 2 vezes na segunda, o INTERSECT normal dá-te só 1 linha (porque é conjunto, sem repetição). O INTERSECT ALL dá-te o mínimo entre as duas contagens — neste caso, 2 linhas. Usa isto quando a frequência importa, não só a presença.

---

# 3. `EXCEPT`

Retorna as linhas que existem na **primeira** query, mas **não** na segunda.

```sql
SELECT nome, email FROM clientes
EXCEPT
SELECT nome, email FROM fornecedores;
```

Resultado:

| nome   | email                |
|--------|----------------------|
| Makene | makenedev@gmail.com  |

"Makene" só existe em `clientes`, então é o único que sobra. "Ana" foi excluído porque também aparece em `fornecedores`.

> EXCEPT — usa quando queres saber o que está numa query mas não está na outra. Exemplo clássico: "quais produtos estão no catálogo mas nunca foram vendidos?" — catálogo EXCEPT produtos_vendidos. Dá-te a diferença de conjuntos, sem duplicados.

## `EXCEPT ALL`

Mantém duplicados, subtraindo as ocorrências. Se "Ana" aparecer 3 vezes em `clientes` e 1 vez em `fornecedores`, o `EXCEPT ALL` devolve "Ana" **2 vezes** (3 - 1).

### A ordem importa aqui

Ao contrário de `UNION` e `INTERSECT`, o `EXCEPT` **não é comutativo** — trocar a ordem das queries muda o resultado.

```sql
-- Clientes que não são fornecedores
SELECT nome, email FROM clientes
EXCEPT
SELECT nome, email FROM fornecedores;

-- Fornecedores que não são clientes (resultado diferente)
SELECT nome, email FROM fornecedores
EXCEPT
SELECT nome, email FROM clientes;
```

A pergunta a fazer é sempre: *"o que tem a primeira query que a segunda não tem?"*

> EXCEPT ALL — mesma lógica de diferença, mas preservando contagem. Se um produto aparece 5 vezes numa lista e 2 vezes na outra que estás a subtrair, o EXCEPT ALL deixa 3 linhas desse produto. É raramente necessário, mas aparece quando estás a comparar registos linha-a-linha (tipo auditoria: "quais destas transações na tabela A não têm correspondência na tabela B, contando repetições").

# Resumo (Operadores de Conjunto)

| Operador         | Retorna                              | Remove duplicados? |
|------------------|---------------------------------------|---------------------|
| `UNION`          | Linhas de ambas as queries            | Sim |
| `UNION ALL`      | Linhas de ambas as queries            | Não |
| `INTERSECT`      | Linhas presentes nas duas queries     | Sim |
| `INTERSECT ALL`  | Linhas presentes nas duas queries     | Não (mantém o mínimo de ocorrências) |
| `EXCEPT`         | Linhas só na primeira query           | Sim |
| `EXCEPT ALL`     | Linhas só na primeira query           | Não (subtrai ocorrências) |

**Regra de ouro:** se não precisas de eliminar duplicados, usa sempre a versão `ALL` — é mais rápida porque poupa o banco de ter que comparar e ordenar todas as linhas.

---

# `ORDER BY` com operadores de conjunto

O `ORDER BY` só pode aparecer **uma vez**, no final de tudo, e aplica-se ao **resultado combinado** — não a cada query individualmente. Também não podes usar o prefixo da tabela, só o nome da coluna (ou alias) do resultado final.

```sql
SELECT nome, email FROM clientes
UNION
SELECT nome, email FROM fornecedores
ORDER BY nome ASC;
```

---

# Subqueries (Consultas Aninhadas)

Uma **subquery** é um `SELECT` dentro de outro `SELECT`. Serve para casos em que a resposta de uma pergunta depende do resultado de outra pergunta — tipo "quais produtos custam mais que a média?". Não dás para responder isso com um `WHERE` direto, porque "a média" também é uma query. É aqui que entra a subquery: a query de dentro corre primeiro, e o resultado dela alimenta a query de fora.

Vamos usar as tabelas `users`, `posts` e `products` já conhecidas.

---

## 1. Subquery Escalar (retorna um único valor)

Quando a subquery devolve **uma única linha e uma única coluna**, pode ser usada onde normalmente usarias um valor fixo — dentro do `SELECT` ou do `WHERE`.

```sql
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

Aqui:

- `(SELECT AVG(price) FROM products)` corre primeiro e devolve **um número** (a média de preços)
- Esse número substitui o lugar onde está escrito, como se fosse uma constante

> Se a subquery devolver mais do que uma linha aqui, o PostgreSQL rejeita a query com erro — subquery escalar tem de devolver exatamente um valor.

Também podes usar no `SELECT`, como coluna calculada:

```sql
SELECT name, price, (SELECT AVG(price) FROM products) AS avg_price
FROM products;
```

---

## 2. Subquery no `FROM` (tabela derivada)

Uma subquery também pode aparecer no lugar de uma tabela, dentro do `FROM`. O resultado dela funciona como uma **tabela temporária**, e por isso precisa sempre de um alias.

```sql
SELECT avg_posts.user_id, avg_posts.total_posts
FROM (
    SELECT user_id, COUNT(*) AS total_posts
    FROM posts
    GROUP BY user_id
) AS avg_posts
WHERE avg_posts.total_posts > 1;
```

Aqui:

- A subquery interna calcula quantos posts cada `user_id` tem
- `AS avg_posts` dá nome a essa tabela derivada
- A query externa trata `avg_posts` como se fosse uma tabela normal, filtrando e selecionando dela

> Repara que isto parece com `HAVING COUNT(*) > 1`, mas usar subquery no `FROM` dá mais liberdade quando precisas de reaproveitar o resultado agregado em cálculos adicionais na query de fora.

---

## 3. Subquery no `WHERE` com `IN`

Quando a subquery devolve **uma lista de valores** (uma coluna, várias linhas), usa-se com `IN`.

```sql
SELECT username
FROM users
WHERE id IN (SELECT user_id FROM posts);
```

> "Todos os usuários que têm pelo menos um post."

A subquery `(SELECT user_id FROM posts)` devolve uma lista de IDs, e o `IN` verifica se o `id` do user está nessa lista.

## `NOT IN` — cuidado com `NULL`

```sql
SELECT username
FROM users
WHERE id NOT IN (SELECT user_id FROM posts);
```

> "Todos os usuários que nunca postaram."

**Atenção:** se a subquery devolver **um único `NULL`** entre os resultados, o `NOT IN` inteiro deixa de devolver linhas nenhumas — silenciosamente, sem erro. Isto acontece porque `NOT IN` compara com todos os valores da lista, e comparar com `NULL` dá `NULL` (nem verdadeiro, nem falso). Para evitar essa armadilha, garante que a coluna da subquery é `NOT NULL`, ou usa `NOT EXISTS` (a seguir) em vez de `NOT IN`.

---

## 4. Subquery com `EXISTS`

`EXISTS` verifica **apenas se a subquery devolve alguma linha** — não olha para os valores, só se existe pelo menos um resultado. Por isso é normalmente uma **subquery correlacionada** (ver secção 5).

```sql
SELECT username
FROM users
WHERE EXISTS (
    SELECT 1 FROM posts WHERE posts.user_id = users.id
);
```

> "Todos os usuários que têm pelo menos um post" — mesmo resultado do exemplo com `IN`, mas escrito de outra forma.

`SELECT 1` aqui é só uma convenção — como o `EXISTS` não olha para o valor devolvido, tanto faz selecionar `1`, uma coluna ou `*`. O que importa é se **existe linha**.

## `NOT EXISTS`

```sql
SELECT username
FROM users
WHERE NOT EXISTS (
    SELECT 1 FROM posts WHERE posts.user_id = users.id
);
```

> "Todos os usuários que nunca postaram" — a alternativa segura ao `NOT IN`, porque `NOT EXISTS` não tem o problema do `NULL`.

---

# ALL e SOME (ANY)

Servem para comparar um valor com **cada linha** devolvida por uma subquery, sem precisar de `IN` ou `EXISTS`. A diferença para o `IN` é que `ALL` e `SOME` funcionam com qualquer operador de comparação (`>`, `<`, `>=`, `<=`, `=`, `!=`), não só igualdade.

Vamos usar a tabela `products`, agora com uma coluna `category`:

### `products`

| id | name       | category    | price |
|----|------------|-------------|-------|
| 1  | Teclado    | Informática | 50    |
| 2  | Monitor    | Informática | 300   |
| 3  | Cadeira    | Escritório  | 200   |
| 4  | Secretária | Escritório  | 400   |

---

## `ALL`

A condição só é verdadeira se for verdadeira para **todos** os valores devolvidos pela subquery.

```sql
SELECT name, price
FROM products
WHERE price > ALL (
    SELECT price FROM products WHERE category = 'Escritório'
);
```

> "Produtos mais caros que **todos** os produtos da categoria Escritório."

Isto é equivalente a comparar com o **maior valor** do grupo:

```sql
WHERE price > (SELECT MAX(price) FROM products WHERE category = 'Escritório')
```

## `SOME` / `ANY`

`SOME` e `ANY` são sinónimos — o PostgreSQL trata-os da mesma forma, usa o que preferires. A condição é verdadeira se for verdadeira para **pelo menos um** dos valores devolvidos.

```sql
SELECT name, price
FROM products
WHERE price > SOME (
    SELECT price FROM products WHERE category = 'Escritório'
);
```

> "Produtos mais caros que **pelo menos um** produto da categoria Escritório."

Equivalente a comparar com o **menor valor** do grupo:

```sql
WHERE price > (SELECT MIN(price) FROM products WHERE category = 'Escritório')
```

## `= ANY` é o mesmo que `IN`

```sql
SELECT name FROM products WHERE category = ANY (SELECT category FROM products WHERE price > 100);
-- é o mesmo que:
SELECT name FROM products WHERE category IN (SELECT category FROM products WHERE price > 100);
```

> `IN` só serve para igualdade. `ANY`/`SOME` fazem o mesmo, mas aceitam qualquer operador de comparação — é por isso que existem os dois.

## Resumo (ALL / SOME / ANY)

| Operador     | Verdadeiro quando...               | Equivale a     |
|--------------|--------------------------------------|----------------|
| `> ALL`      | maior que **todos** os valores      | `> MAX(...)`   |
| `> SOME/ANY` | maior que **pelo menos um** valor   | `> MIN(...)`   |
| `= ANY`      | igual a **pelo menos um** valor     | `IN (...)`     |
| `<> ALL`     | diferente de **todos** os valores   | `NOT IN (...)` |

> **Atenção:** tal como no `NOT IN`, se a subquery devolver `NULL` no meio dos resultados, comparações com `ALL` podem dar resultados inesperados (nem verdadeiro, nem falso). O mesmo cuidado aplica-se aqui.

---

# `SELECT` sem `FROM`
 
Nem todo `SELECT` precisa de uma tabela. Quando queres calcular uma expressão, testar uma função, ou gerar um valor fixo, podes usar `SELECT` sozinho — sem `FROM`.
 
```sql
SELECT 1 + 1;
```
 
Resultado: uma linha, uma coluna, valor `2`. Não há tabela nenhuma envolvida — é só o PostgreSQL a avaliar a expressão.
 
## Casos de uso comuns
 
```sql
-- Testar uma expressão ou cálculo rápido
SELECT 100 * 1.14 AS preco_com_iva;
 
-- Ver a data e hora atual
SELECT NOW();
 
-- Ver a versão do PostgreSQL
SELECT version();
 
-- Gerar um UUID
SELECT gen_random_uuid();
 
-- Testar o resultado de uma função de string
SELECT UPPER('makene');
```
 
> Repara que é exatamente a mesma lógica das **colunas calculadas** que já vimos — só que sem nenhuma tabela por trás. `SELECT` sempre devolve uma linha por padrão; sem `FROM`, essa linha é "inventada" na hora, só para caber o resultado da expressão.
 
## Onde isto é útil na prática
 
- Testar rapidamente se uma função faz o que esperas, antes de a usar numa query grande
- Gerar valores (UUID, timestamp) para usar num `INSERT`
- Verificar configurações da base de dados (`SHOW timezone;`, `SELECT current_database();`)
## `SELECT` sem `FROM`, mas com o `FROM` "escondido" dentro dos parênteses
 
Há uma variação deste caso que confunde no início: o `SELECT` de fora **não tem `FROM` nenhum**, mas dentro dos parênteses vai uma subquery completa, com o seu próprio `FROM`. Ou seja, o `FROM` existe — só que pertence à subquery, não à query externa.
 
```sql
SELECT (SELECT MAX(price) FROM products) AS preco_mais_caro;
```
 
Repara na estrutura:
 
- A query de fora é só `SELECT (...) AS preco_mais_caro` — **sem `FROM`**
- Dentro dos parênteses, `(SELECT MAX(price) FROM products)` é uma **subquery escalar completa**, com o seu próprio `FROM`
- O resultado da subquery (um único valor) é tratado como se fosse uma constante, exatamente como uma expressão `1 + 1`
> A lógica é a mesma da secção anterior: `SELECT` sem `FROM` devolve sempre uma linha "inventada" para caber o resultado. A diferença é que aqui o resultado não é um valor literal escrito à mão — é calculado por uma query inteira, escondida dentro dos parênteses.
 
### Podes combinar vários
 
```sql
SELECT
    (SELECT COUNT(*) FROM users)  AS total_users,
    (SELECT COUNT(*) FROM posts)  AS total_posts,
    (SELECT MAX(price) FROM products) AS preco_mais_caro;
```
 
Resultado: **uma única linha**, com três colunas — cada uma vinda de uma subquery independente, cada uma com o seu próprio `FROM` lá dentro. A query externa continua sem `FROM`, porque não está a buscar nada de nenhuma tabela diretamente; só está a juntar os resultados das três subqueries numa linha.
 
> Isto é muito usado para montar "dashboards" rápidos — várias métricas resumidas numa única linha, sem precisar de vários `SELECT`s separados nem de `JOIN`s complicados.
 
### Cuidado: continua a ser uma subquery escalar
 
Tal como na subquery escalar dentro do `WHERE`, cada uma destas subqueries tem de devolver **exatamente uma linha e uma coluna**. Se devolver mais do que uma linha, o PostgreSQL dá erro — porque não há como encaixar várias linhas dentro de uma única célula do resultado.
 
```sql
-- ❌ Erro: devolve várias linhas, não cabe numa célula
SELECT (SELECT price FROM products) AS preco;
 
-- ✅ Certo: agregada para devolver um único valor
SELECT (SELECT MAX(price) FROM products) AS preco;
```

---

# `SELECT DISTINCT`

O `DISTINCT` remove linhas duplicadas do resultado de uma query. Em vez de devolver todos os valores repetidos, o PostgreSQL agrupa os que são idênticos e mostra só um de cada.

Vamos usar a tabela `pedidos`:

### `pedidos`

| id | cidade   | produto  |
|----|----------|----------|
| 1  | Luanda   | Teclado  |
| 2  | Benguela | Monitor  |
| 3  | Luanda   | Cadeira  |
| 4  | Huambo   | Teclado  |
| 5  | Luanda   | Monitor  |

---

## `DISTINCT` numa única coluna

```sql
SELECT DISTINCT cidade
FROM pedidos;
```

Sem `DISTINCT`, a coluna `cidade` traria 5 linhas (com `Luanda` repetido 3 vezes). Com `DISTINCT`, o PostgreSQL elimina as repetições:

**Resultado:**

| cidade   |
|----------|
| Luanda   |
| Benguela |
| Huambo   |

> **Regra de ouro:** `DISTINCT` atua **depois** de o PostgreSQL já ter buscado as linhas — ele não impede a leitura dos dados, só remove duplicados do resultado final.

---

## `COUNT(DISTINCT coluna)`

Enquanto `SELECT DISTINCT` devolve **as linhas** únicas, `COUNT(DISTINCT coluna)` devolve **quantos** valores únicos existem — um único número.

```sql
SELECT COUNT(DISTINCT cidade) AS total_cidades
FROM pedidos;
```

**Resultado:**

| total_cidades |
|---------------|
| 3             |

Compara com um `COUNT(*)` simples, que conta todas as linhas, incluindo repetições:

| Query                     | O que conta                    | Resultado |
|----------------------------|----------------------------------|-----------|
| `COUNT(*)`                 | Todas as linhas da tabela        | 5         |
| `COUNT(cidade)`             | Linhas com `cidade` não nula     | 5         |
| `COUNT(DISTINCT cidade)`   | Valores **únicos** de `cidade`   | 3         |

### Combinando com `GROUP BY`

`COUNT(DISTINCT coluna)` é muito usado junto com `GROUP BY`, para responder perguntas como "quantos produtos diferentes cada cidade comprou?":

```sql
SELECT cidade, COUNT(DISTINCT produto) AS produtos_diferentes
FROM pedidos
GROUP BY cidade;
```

**Resultado:**

| cidade   | produtos_diferentes |
|----------|-----------------------|
| Luanda   | 3                     |
| Benguela | 1                     |
| Huambo   | 1                     |

> 💡 **Dica:** usa `SELECT DISTINCT` quando queres **ver os valores** únicos; usa `COUNT(DISTINCT ...)` quando só precisas de **saber quantos** existem, sem listar cada um.
 
---

# A função `GREATEST`

`GREATEST` recebe vários valores (ou colunas) e devolve **o maior** entre eles, linha a linha.

```sql
SELECT GREATEST(10, 25, 3);
```

| greatest |
|----------|
| 25       |

## `GREATEST` vs `MAX()`

| Função        | Compara     | Devolve |
|----------------|-------------|---------|
| `MAX(coluna)`  | Entre **linhas** | Um valor, resumindo a tabela |
| `GREATEST(a, b, c)` | Entre **colunas/valores**, na mesma linha | Um valor por linha |

## Exemplo com colunas

### `products`

| id | name    | preco_loja_a | preco_loja_b | preco_loja_c |
|----|---------|---------------|---------------|---------------|
| 1  | Teclado | 50            | 45            | 60            |
| 2  | Monitor | 300           | 320           | 280           |

```sql
SELECT name, GREATEST(preco_loja_a, preco_loja_b, preco_loja_c) AS preco_mais_alto
FROM products;
```

| name    | preco_mais_alto |
|---------|-------------------|
| Teclado | 60                |
| Monitor | 320               |

## `LEAST`

O oposto: devolve o menor valor.

```sql
SELECT LEAST(preco_loja_a, preco_loja_b, preco_loja_c) AS preco_mais_baixo
FROM products;
```

> **Regra de ouro:** `GREATEST`/`LEAST` ignoram `NULL`, a menos que todos os argumentos sejam `NULL`.

---

# A expressão `CASE`

`CASE` permite aplicar lógica condicional dentro de uma query — o equivalente a um `if/else` do SQL. Devolve um valor diferente consoante a condição, avaliada linha a linha.

```sql
SELECT nome, preco,
    CASE
        WHEN preco > 300 THEN 'Caro'
        WHEN preco > 100 THEN 'Médio'
        ELSE 'Barato'
    END AS faixa_preco
FROM products;
```

### `products`

| nome    | preco | faixa_preco |
|---------|-------|-------------|
| Teclado | 50    | Barato      |
| Cadeira | 200   | Médio       |
| Monitor | 320   | Caro        |

> As condições `WHEN` são avaliadas **em ordem**, de cima para baixo. Assim que uma for verdadeira, o `CASE` para e usa esse resultado — as seguintes nem são avaliadas.

## `CASE` simples (comparação direta)

Quando a comparação é sempre com a mesma coluna, pode simplificar-se:

```sql
SELECT nome,
    CASE categoria
        WHEN 'Informática' THEN 'Tech'
        WHEN 'Escritório' THEN 'Móveis'
        ELSE 'Outro'
    END AS tipo
FROM products;
```

Equivale a `CASE WHEN categoria = 'Informática' THEN ...`, só que mais direto.

## `ELSE` é opcional

Se nenhuma condição for verdadeira e não houver `ELSE`, o resultado é `NULL`.

## Usando com agregação

```sql
SELECT
    COUNT(CASE WHEN preco > 300 THEN 1 END) AS produtos_caros,
    COUNT(CASE WHEN preco <= 300 THEN 1 END) AS produtos_baratos
FROM products;
```

| produtos_caros | produtos_baratos |
|-----------------|--------------------|
| 1               | 2                  |

> **Regra de ouro:** `COUNT` só conta valores não nulos — por isso `CASE WHEN condição THEN 1 END` (sem `ELSE`) funciona como um contador condicional: linhas que não batem a condição viram `NULL` e são ignoradas.
