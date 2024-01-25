   ```mermaid
   %%类型: 饼状图pie 流程图graph 甘特图gantt 序列图sequencediagram  类图classdiagram 状态图 statediagram, 用户旅行图 journey

   stateDiagram-v2
    [*] -->pg_ctl
    pg_ctl --> postmaster : fork() 主进程
    postmaster --> PostmasterMain 
    PostmasterMain-->maybe_start_bgworkers
    PostmasterMain--> ServerLoop
    PostmasterMain --> StartupDataBase

    %%signal 处理入口
    PostmasterMain -->SIGHUP_handler:SIGHUP
    PostmasterMain -->pmdie:SIGINT
    PostmasterMain -->pmdie:SIGQUIT
    PostmasterMain --> pmdie:SIGTERM
    PostmasterMain -->sigusr1_handler:SIGUSR1
    PostmasterMain -->reaper:SIGCHLD

    ServerLoop --> BackendStartup : 新连接建立, 创建独立进程
    maybe_start_bgworkers --> do_start_bgworker
    do_start_bgworker --> fork_process
    ServerLoop --> StartAutoVacLauncher
    ServerLoop --> StartArchiver
    ServerLoop -->StartBackgroundWriter
    ServerLoop -->StartCheckpointer
    ServerLoop -->StartWalWriter
    ServerLoop -->StartWalReceiver

    StartArchiver-->AuxiliaryProcessMain
    StartBackgroundWriter-->AuxiliaryProcessMain
    StartCheckpointer-->AuxiliaryProcessMain
    StartWalWriter-->AuxiliaryProcessMain
    StartWalReceiver-->AuxiliaryProcessMain
    StartupDataBase-->AuxiliaryProcessMain
    
    PostmasterStateMachine --> StartupDataBase
    signal_handlers --> pmdie
    signal_handlers --> reaper
    signal_handlers --> sigusr1_handler
    pmdie --> PostmasterStateMachine
    reaper --> PostmasterStateMachine
    sigusr1_handler --> PostmasterStateMachine
    
