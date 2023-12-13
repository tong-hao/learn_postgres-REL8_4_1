
# 代码

## 后端主要模块
- parser 解析SQL语句为查询树
- commands SQL命令，如CREATE、DROP、ALTER
- rewrite 重写查询树
- optimizer 优化器
- executor 执行计划
- storage 数据存储
- catalog 元数据，如表结构、索引信息


## 命令行客户端启动流程
```
startup.c/main
 parse_psql_options
 // -f
 process_file
 // -c
 psql_scan_setup
 HandleSlashCmds
 SendQuery
 MainLoop //循环读取命令行的查询请求-->将请求发往后端-->从后端获取请求的结果
  psql_scan_setup
  psql_scan
   yylex()
  SendQuery
   PQexec // 获取数据库后台返回的结果
    PQsendQuery
     pqPutMsgStart
     pqPuts
     pqPutMsgEnd
    ProcessResult // 把运行结果显示在屏幕上
     handleCopyOut
     handleCopyIn
```

## 后端启动流程
```
src/backend/main/main.c/main
 MemoryContextInit()
 set_pglocale_pgservice //获取并设置环境变量
 init_locale //初始化环境变量 
 PostmasterMain()
  StartupDataBase() //启动后台子进程
  ServerLoop() //监听新的建立连接消息
   BackendStartup //fork一个后台进程
    BackendInitialize(port) //完成Backend的初始化
    BackendRun
     PostgresMain
      ReadCommand(StringInfo inBuf)
       exec_simple_query
```

## 执行sql流程
```
src/backend/tcop/postgres.c/exec_simple_query
 pg_parse_query // 词法分析和语法分析
  raw_parser
   base_yyparse // Lex和Yacc配合生成
 pg_analyze_and_rewrite // 语义分析
  parse_analyze //语义分析
   transformTopLevelStmt //处理语义分析
    transformStmt
    nodeTag //获取传进来的语法树
    switch语句 //根据NodeTag区分不同的命令类型
    transformInsertStmt
    transformDeleteStmt
    transformUpdateStmt
    transformSelectStmt
    transformCreateTableAsStmt
    transformExplainStmt
   pg_rewrite_query //查询重写
```

查询重写:
```
pg_rewrite_query
 QueryRewrite
  RewriteQuery
   fireRIRrules
```

执行计划：
```
exec_simple_query
  pg_plan_queries
   pg_plan_query
    planner
     standard_planner
      subquery_planner
       SS_xxx //预处理
       pull_up_sublinks //预处理
       pull_up_subqueries //预处理
       Preprocess_xxx //预处理
       inheritance_planner
       grouping_planner //生成查询计划【重要】
       SS_finalize_plan //清理查询计划树
       set_plan_references //清理工作
```
