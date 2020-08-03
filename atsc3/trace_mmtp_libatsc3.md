# MMTP flow of libatsc3

#### atsc3_listener_metrics_ncurses.cpp::process_packet()
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

#### atsc3_lls_mmt_utils.c::lls_slt_mmt_session_find_from_udp_packet(lls_slt_monitor, src_ip_addr, dst_ip_addr, dst_port)
    for(int i=0; i < lls_slt_monitor->lls_sls_mmt_session_flows_v.count; i++)
    {
        lls_sls_mmt_session_flows = lls_slt_monitor->lls_sls_mmt_session_flows_v.data[i];
        for(int j=0; j < lls_sls_mmt_session_flows->lls_sls_mmt_session_v.count; j++ )
        {
            lls_slt_mmt_session = lls_sls_mmt_session_flows->lls_sls_mmt_session_v.data[j];
            if ((lls_slt_mmt_session->sls_relax_source_ip_check || 
                (!lls_slt_mmt_session->sls_relax_source_ip_check && 
                  lls_slt_mmt_session->sls_source_ip_address == src_ip_addr)) &&
				  lls_slt_mmt_session->sls_destination_ip_address == dst_ip_addr &&
				  lls_slt_mmt_session->sls_destination_udp_port == dst_port)
            {
                return lls_slt_mmt_session;
            }
        }
    }

#### atsc3_listener_metrics_ncurses.cpp::update_global_mmtp_statistics_from_udp_packet_t(udp_packet)
    ...
    mmtp_packet_header = mmtp_packet_header_parse_from_block_t();
    ...
    if (mmtp_packet_header->mmtp_payload_type == 0x0)
    {
        mmtp_mpu_packet = mmtp_mpu_packet_parse_from_block_t(mmtp_packet_header, ...);
        ...
        if (mmtp_mpu_packet->mpu_timed_flag == 1)
        {
            atsc3_packet_statistics_mmt_stats_populate();
        }
    }
    else if (mmtp_packet_header->mmtp_payload_type == 0x2)
    {
        mmtp_signalling_packet = mmtp_signalling_packet_parse_and_free_packet_header_from_block_t();
        mmt_signalling_message_parse_packet(mmtp_signalling_packet, ...);
        ...
    }
    ...

***
![](/atsc3/res/mmtp_1.png)
***






