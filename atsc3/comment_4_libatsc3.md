# Comment for libatsc3
### atsc3_lls_slt_parser.c::lls_slt_monitor_create()
    lls_slt_monitor_t* lls_slt_monitor = lls_slt_monitor_new();
    -> atsc3_lls_types.c::ATSC3_VECTOR_BUILDER_METHODS_PARENT_IMPLEMENTATION(lls_slt_monitor)
    -> atsc3_vector_builder.h::
        #define ATSC3_VECTOR_BUILDER_METHODS_PARENT_IMPLEMENTATION(vector_struct_name) \
        PPCAT(vector_struct_name,_t)* PPCAT(vector_struct_name,_new)() { \
            PPCAT(vector_struct_name,_t)* vector_struct_name = calloc(1, sizeof(PPCAT(vector_struct_name,_t))); \
            return vector_struct_name; \
        }
        -> lls_slt_monitor_t * lls_slt_monitor_new() {
            lls_slt_monitor_t * lls_slt_monitor = calloc(1, sizeof(lls_slt_monitor_t));
            return lls_slt_monitor;
        }
        
### atsc3_listener_metrics_ncurses.cpp::pcap_loop_run_thread_with_file()
    ...
    pcap_open_offline(filename, errbuf); 
    ...
    pcap_loop(descr,-1,process_packet,NULL);

### atsc3_listener_metrics_ncurses.cpp::::process_packet()
    process_packet_from_pcap();  /* strip out udp packet */
    ...
    if(udp_packet->udp_flow.dst_ip_addr == LLS_DST_ADDR && 
       udp_packet->udp_flow.dst_port == LLS_DST_PORT) 
    {
        //#define LLS_DST_ADDR 3758102332 --> 0xE0_00_17_3C -> 224_0_23_60
        //#define LLS_DST_PORT 4937
        ...
        lls_table = lls_table_create_or_update_from_lls_slt_monitor_with_metrics();
        ...
        if(lls_table->lls_table_id == SLT) {
            ...
            lls_slt_table_perform_update(lls_table, lls_slt_monitor);
            ...
        }
    }
    ...
    //ALC(ROUTE) process
    matching_lls_slt_alc_session = lls_slt_alc_session_find_from_udp_packet(lls_slt_monitor, ...);
    if(matching_lls_slt_alc_session) {
        ...
        alc_packet = route_parse_from_udp_packet();
        route_process_from_alc_packet(xxx, &alc_packet);
        ...
    }
    
    //MMTP process
    matching_lls_slt_mmt_session = lls_slt_mmt_session_find_from_udp_packet(lls_slt_monitor, ...);
    if(matching_lls_slt_mmt_session) {
        update_global_mmtp_statistics_from_udp_packet_t(udp_packet);
    }
