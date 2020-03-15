# 【FFMPEG】设置FFMPEG载流为TCP以及设置TIMEOUT

标签（空格分隔）： 【FFMPEG】

---

    char filepath[]="rtsp://admin:jjchen12345678@192.168.3.25:554/h264/ch1/main/av_stream";
    AVDictionary *optionsDict = NULL;
    av_register_all();
    avformat_network_init();
    
    pFormatCtx = avformat_alloc_context();
    
    av_dict_set(&optionsDict, "rtsp_transport", "tcp", 0); //采用tcp传输
    av_dict_set(&optionsDict, "stimeout", "2000000", 0); //如果没有设置 stimeout，那么把ipc网线拔掉，av_read_frame会阻塞（时间单位是微妙）
    if(avformat_open_input(&pFormatCtx,filepath,NULL,&optionsDict)!=0){
    printf("无法打开文件\n");
    return -1;
    }

--------
2019-12-25 张朋艺