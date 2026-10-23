# Orkesta — copia de la base para el RAG

Snapshot completo de la base de Orkesta (Postgres 16 + pgvector) con esquema y datos:
**221 documentos / 1843 chunks**, embeddings `BAAI/bge-m3` (1024 dim).

Contenido:
- `orkesta.dump` — dump en formato custom de `pg_dump` (esquema + datos).
- `docker-compose.yml` — levanta un Postgres+pgvector idéntico.

## Levantar y restaurar (una vez)

Requiere Docker. Desde esta carpeta:

```bash
# 1. Levantar Postgres + pgvector vacío
docker compose up -d

# 2. Esperar ~5s a que inicialice, luego copiar y restaurar el dump
docker cp orkesta.dump orkesta-db:/tmp/orkesta.dump
docker exec orkesta-db pg_restore -U orkesta -d orkesta --no-owner /tmp/orkesta.dump
```

> El dump trae `CREATE EXTENSION vector` y `pg_trgm`; la imagen `pgvector/pgvector:pg16`
> ya las soporta, así que no hay que instalar nada aparte.

## Verificar

```bash
docker exec -it orkesta-db psql -U orkesta -d orkesta -c "SELECT COUNT(*) FROM documents;" -c "SELECT COUNT(*) FROM chunks;"
```

## Conexión

```
postgresql://orkesta:orkesta@localhost:5432/orkesta
```
(cambialo a 5440 en el compose y acá si ya usás el 5432).

## Tablas

- `documents` (incluye `source`: pdfplumber | ocr), `chunks` (vector 1024 + tsvector + HNSW/GIN),
  `ingest_audit`, `query_log`.
- Ver `SCHEMA.md` (el que ya te pasó Mirko) para el detalle de columnas y los patrones de
  query para el RAG (semántica con `<=>`, full-text en español, e híbrida).

## Importante para el RAG

Para embeber las preguntas tenés que usar el **mismo modelo** con el que se indexó:
`BAAI/bge-m3`, 1024 dim, con `normalize_embeddings=True`. Si usás otro modelo cambia la
dimensión y habría que re-generar la columna `embedding` y el índice HNSW.
