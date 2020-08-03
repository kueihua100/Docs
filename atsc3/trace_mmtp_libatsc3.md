# MMTP flow of libatsc3

#### atsc3_listener_metrics_ncurses.cpp::::process_packet()
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
        return udp_packet_free(&udp_packet);
    }
    ...
    
    //MMTP process
    matching_lls_slt_mmt_session = lls_slt_mmt_session_find_from_udp_packet(lls_slt_monitor, ...);
    if(matching_lls_slt_mmt_session) {
        update_global_mmtp_statistics_from_udp_packet_t(udp_packet);
    }
