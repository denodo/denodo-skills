# Vector Search

VQL supports embedding model-powered functions capable of performing similarity search operations with vector embedding fields, by comparing them with a search string. For example, `VECTOR_DISTANCE` can search for the query `'black car'` in embedding vectors of images to retrieve the N most relevant rows of images that contain black cars in them.

When searching a view with vector fields, vector search must be prioritized over keyword search and text manipulation.

When working with similarity search a limit must always be imposed for N. Default should be 5 unless otherwise specified.

When working with vectors, you must always include the similarity score in the results.

These are the supported vector functions in VQL:

- `VECTOR_DISTANCE(<target vector:vector>, <search query:text> [, <distance metric:text>]):float`. Embedding-powered. Returns the distance between a textual search query and an embedding vector to filter by similarity. Most similar results will have the smallest distance, so order ASC to obtain most similar.
- `EMBED_AI(<search query:text> [, <embedding model:text>]):vector`. Embeds a given text to obtain it's vector representation.
- `VECTOR_COSINE_DISTANCE(<vector1:vector>, <vector2:vector>):float`. Calculates the distance between two vectors using cosine distance.
- `VECTOR_L1_DISTANCE(<vector1:vector>, <vector2:vector>):float`. Calculates the distance between two vectors using L1 distance.
- `VECTOR_L2_DISTANCE(<vector1:vector>, <vector2:vector>):float`. Calculates the distance between two vectors using L2 distance.

When using a vector function in VQL, follow this query structure:

```vql
SELECT ..., <distance_function>(vector_field, query_text) AS distance
FROM ...
ORDER BY distance ASC
LIMIT N
```

The first `ORDER BY` parameter must always be the distance, even when using sorting by multiple fields.

When performing a `JOIN` between a vector search with a vector function and another view, follow this query structure:

```vql
SELECT ...
FROM (
    SELECT ..., <distance_function>(vector_col, query_vector) AS distance
    FROM ...
    ORDER BY distance ASC
    LIMIT N
) top_matches
JOIN ...
```

## Prepared vector views

Some views may contain raw vector fields but also text input search fields that will take care of the vector search automatically. Whenever working with one of these views you must:

1. Always use this input search field in your VQL query.
2. Never use vector functions on the actual vectors.
3. If multiple input search fields are available, you can use multiple in your query, but always use at least one.

For example:

```vql
SELECT ...
FROM ...
WHERE input_search_field_vectorA = 'query_text'
```

Instead of:

```vql
VECTOR_DISTANCE(vectorA, 'query_text')
```

The condition can only be an equality condition and cannot be used multiple times.
This is invalid:

```vql
WHERE input_search_field = 'query_text' AND input_search_field = 'query_text_2'
```

Sometimes these views may also contain a field to control the number of similar elements to retrieve, like:

```vql
SELECT ..., similarity_score
FROM ...
WHERE input_search_field_vectorA = 'query_text'
AND input_search_limit = 7
ORDER BY similarity_score
LIMIT 7
```

In these cases, set both the search field and the search limit and use LIMIT on the outer query.

NOTE: You cannot combine vector distance functions and prepared input fields in the same VQL query, you must do one or the other. 