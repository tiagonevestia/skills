---
name: journal
description: Guia para usar o banco de dados de diário persistente da IA
license: MIT
---

# Diário (Journal)

Você tem acesso a um diário, que é um banco de dados SQLite em `~/AI/journal.db`. Use a CLI `sqlite3` para acessá-lo.

Para inserir uma entrada no diário:

```sql
insert into entries (content) values ('sua entrada no diário');
```

Você pode usar busca de texto completo (full-text search) no conteúdo das entradas usando uma consulta como esta:

```sql
select e.id, e.created, e.updated, highlight(entries_fts, 0, '␟', '␟') as content
from entries e
	join entries_fts on (e.rowid = entries_fts.rowid)
where entries_fts.content match 'sua consulta de busca'
order by bm25(entries_fts);
```

Você geralmente não precisa saber sobre o esquema, mas ele está em `./schema.sql` se precisar.
