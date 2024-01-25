1, src/bin/pg_ctl 命令:    
    代码执行, pg_ctl.c,
        全局文件指针,
            pid_file //$PGDATA/postmaster.pid
                /*保存主postgres 进程的pid 文件路径*/
                snprintf(pid_file, MAXPGPATH, "%s/postmaster.pid", pg_data);
                /**postmaster.pid 结构:
                    postmaster.pid
                        2021                    #主postmaster 进程的进程ID
                        /var/lib/pgsql/data     #PGDATA 目录
                        1699230517
                        5432                    #监听端口
                        /var/run/postgresql
                        *
                        393217         0
                        ready
                */
            postopts_file //指向 $PGDATA/postmaster.opts
                /*保存postmaster 主进程启动的参数
                eg, /usr/bin/postgres "-D" "/var/lib/pgsql/data" */
                
            version_file //$PGDATA/PG_VERSION
                /*保存postgres 的版本号*/

            backup_file //$PGDATA/backup_label
                /*TBD*/
            
        main()
        {
            /*reload 参数输入的情况下, 调用do_reload()*/
            case RELOAD_COMMAND:
                sig = SIGHUP;
			    do_reload();
        };

        //启动postgres 数据库
        do_start()
        {
            -检查pid_file 是否包含有效进程id (表示postmaster 进程存在);
            -查找pg_ctl 同目录下的指定名称'postgres' 可执行文件
            -fork() 调用创建主postmaster 进程, 同时使用来自postopts_file 的参数(PGDATA 等);
        }

        //重新加载配置
        do_reload
        {
            -读取 pid_file 文件里记载的postmaster 进程pid;
            -通过 kill(pid, sig) 方式发送;
        }


2, src/backend/main.c postgres 进程, 
    main()
    {
        /*根据启动postgres 进程的第一参数, 运行不同的进程入口**/
            '--check': BootstrapModeMain(true) //bootstrapmode 检查启动参数
            '--boot': BootstrapModeMain(false) //bootstrapmode 用于以非SQL 方式运行, 初始化数据库                    
            '--fork': SubPostmasterMain()      
            '--describe-config': GucInfoMain()   //文字打印显示所有的主进程支持的参数, 类型和描述信息, 参见 "主进程所有支持的参数列表"
            '--single': PostgresSingleUserMain() //
            default: PostmasterMain() //
    }

    2.1 // src\backend\utils\misc\guc.c   主进程所有支持的参数列表,
        ConfigureNamesBool[]{}
        ConfigureNamesEnum[]{}
        ConfigureNamesInt[]{}
        ConfigureNamesReal[]{}
        ConfigureNamesString[]{}
        例如: postgres.conf 配置文件里面的, listen_addresses 和 port 参数.
        
    2.2 postgres 新建进程的名称,
        GetBackendTypeDesc(){
            case B_INVALID: backendDesc = "not initialized";
			case B_AUTOVAC_LAUNCHER: backendDesc = "autovacuum launcher";
			case B_AUTOVAC_WORKER: backendDesc = "autovacuum worker";
            case B_BACKEND: backendDesc = "client backend";
            case B_BG_WORKER: backendDesc = "background worker";
            case B_BG_WRITER: backendDesc = "background writer";
            case B_CHECKPOINTER: backendDesc = "checkpointer";
            case B_STARTUP: backendDesc = "startup";
            case B_WAL_RECEIVER: backendDesc = "walreceiver";
            case B_WAL_SENDER: backendDesc = "walsender";
            case B_WAL_WRITER: backendDesc = "walwriter";
            case B_ARCHIVER: backendDesc = "archiver";
            case B_LOGGER: backendDesc = "logger"; }

