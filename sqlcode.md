``` sql
SELECT TOP 20 total_worker_time / execution_count AS AvgCPU ,
        total_worker_time AS TotalCPU ,
		CAST(ROUND(100.00 * total_worker_time / (SELECT SUM(total_worker_time) FROM sys.dm_exec_query_stats), 2) AS MONEY) AS PercentCPU,
        total_elapsed_time / execution_count AS AvgDuration ,
        total_elapsed_time AS TotalDuration ,
		CAST(ROUND(100.00 * total_elapsed_time / (SELECT SUM(total_elapsed_time) FROM sys.dm_exec_query_stats), 2) AS MONEY) AS PercentDuration,
        total_logical_reads / execution_count AS AvgReads ,
        total_logical_reads AS TotalReads ,
		CAST(ROUND(100.00 * total_logical_reads / (SELECT SUM(total_logical_reads) FROM sys.dm_exec_query_stats), 2) AS MONEY) AS PercentReads,
        execution_count ,
		CAST(ROUND(100.00 * execution_count / (SELECT SUM(execution_count) FROM sys.dm_exec_query_stats), 2) AS MONEY) AS PercentExecutions,
	executions_per_minute = CASE  DATEDIFF(mi, creation_time, qs.last_execution_time)
		WHEN 0 THEN 0
		ELSE CAST((1.00 * execution_count / DATEDIFF(mi, creation_time, qs.last_execution_time)) AS money)
	END,
        qs.creation_time AS plan_creation_time,
        qs.last_execution_time,
        SUBSTRING(st.text, ( qs.statement_start_offset / 2 ) + 1, ( ( CASE qs.statement_end_offset
                                                                        WHEN -1 THEN DATALENGTH(st.text)
                                                                        ELSE qs.statement_end_offset
                                                                      END - qs.statement_start_offset ) / 2 ) + 1) AS QueryText ,
        query_plan
    FROM sys.dm_exec_query_stats AS qs
        CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st 
        CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) AS qp
    ORDER BY TotalCPU DESC;
	--ORDER BY TotalDuration DESC; 
	--ORDER BY TotalReads DESC;
	--ORDER BY execution_count DESC; 
GO
```
