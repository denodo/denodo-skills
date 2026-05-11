# Fix LIMIT/FETCH inside a subquery error

LIMIT/FETCH in a subquery is an error that might appear in some versions of Denodo. When your VQL fails due to a LIMIT/FETCH in subquery error, produce an equivalent query **without** LIMIT or FETCH in any subquery by rewriting with **`ROW_NUMBER()`** and using outer filter on the row number.