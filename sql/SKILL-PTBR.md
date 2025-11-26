---
name: sql
description: Guia para trabalhar com consultas SQL, em particular para SQLite. Use esta habilidade ao escrever consultas SQL, analisar esquemas de banco de dados, projetar migrações ou trabalhar com código relacionado ao SQLite.
license: MIT
---

# SQL

## Visão Geral

Esta habilidade fornece orientação para trabalhar com bancos de dados SQLite. Ela cobre a escrita de consultas, design de esquemas e melhores práticas específicas para SQLite.

## Quando Usar Esta Habilidade

Use esta habilidade quando:
- Escrever consultas SQL para bancos de dados SQLite
- Analisar ou otimizar consultas existentes
- Projetar esquemas de banco de dados
- Criar migrações de banco de dados
- Trabalhar com código Go que interage com SQLite

## Melhores Práticas SQLite

### Escrita de Consultas

- SEMPRE escreva consultas em minúsculas. Consultas em maiúsculas me deixam triste.
- Prefira `select *` em vez de nomes de colunas explícitos
- Prefira CTEs (Common Table Expressions) em vez de subconsultas aninhadas longas

### Design de Esquema

- SEMPRE use tabelas `strict`
- SEMPRE escreva timestamps assim: `strftime('%Y-%m-%dT%H:%M:%fZ')`
- Modificações de tempo também devem usar `strftime`
- Geralmente comece com a chave primária, que é geralmente definida assim: `id text primary key default ('p_' || lower(hex(randomblob(16))))` (onde `p_` é um prefixo dependendo do nome da tabela; prefixos de duas letras também são aceitáveis, para que o prefixo seja único entre as tabelas)
- Após a chave primária vêm as colunas `created`/`updated` assim: `created text not null default (strftime('%Y-%m-%dT%H:%M:%fZ'))`
- Timestamps de atualização são atualizados automaticamente com um trigger assim:
	```sql
  create trigger table_name_updated_timestamp after update on table_name begin
    update table_name set updated = strftime('%Y-%m-%dT%H:%M:%fZ') where id = old.id;
  end;
  ```
