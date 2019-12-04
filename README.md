# pg_statements_usage
queries to analyze pg_stat_statement


CPU usage : 


~~~sql
SELECT substring(query, 1, 50) AS short_query,
              round(total_time::numeric, 2) AS total_time,
              calls,
              round(mean_time::numeric, 2) AS mean,
              round((100 * total_time / sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM  pg_stat_statements
ORDER BY total_time DESC
LIMIT 20;
~~~

Data size written from biggest to smallest :

~~~~sql
SELECT 
  short_query, 
  rows, 
  calls, 
  total_dirtied, 
  total_written, 
  temp_written 
FROM (

  SELECT substring(query, 1, 50) AS short_query,
                rows,
                calls,
                shared_blks_dirtied + local_blks_dirtied as tot_blks_dirtied,
                shared_blks_written + local_blks_written as tot_blks_written,
                temp_blks_written,
                pg_size_pretty(nullif(shared_blks_dirtied + local_blks_dirtied, 0) * current_setting('block_size')::numeric) AS total_dirtied,
               pg_size_pretty(nullif(shared_blks_written + local_blks_written, 0) * current_setting('block_size')::numeric) AS total_written,
                pg_size_pretty(nullif(temp_blocks_written, 0) * current_setting('block_size')::numeric) AS temp_written
  FROM  pg_stat_statements
  ) as t
ORDER BY t.tot_blks_written DESC, t.tot_blks_dirtied DESC, t.temp_blks_written DESC
LIMIT 20;
~~~~
