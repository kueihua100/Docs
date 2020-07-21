# ROUTE flow of libatsc3
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
    //ALC(ROUTE) process
    matching_lls_slt_alc_session = lls_slt_alc_session_find_from_udp_packet(lls_slt_monitor, ...);
    if(matching_lls_slt_alc_session) {
        ...
        alc_packet = route_parse_from_udp_packet(matching_lls_slt_alc_session, ...);
        if (alc_packet)
            route_process_from_alc_packet(xxx, &alc_packet);
        ...
    }
    
#### atsc3_lls_alc_utils.c::lls_slt_alc_session_find_from_udp_packet(lls_slt_monitor, src_ip_addr, dst_ip_addr, dst_port)
    for(int i=0; i < lls_slt_monitor->lls_sls_alc_session_flows_v.count; i++)
    {
        lls_sls_alc_session_flows = lls_slt_monitor->lls_sls_alc_session_flows_v.data[i];
        for(int j=0; j < lls_sls_alc_session_flows->lls_sls_alc_session_v.count; j++ )
        {
            lls_slt_alc_session = lls_sls_alc_session_flows->lls_sls_alc_session_v.data[j];
            if (
                (lls_slt_alc_session->sls_relax_source_ip_check || 
                (!lls_slt_alc_session->sls_relax_source_ip_check && 
                  lls_slt_alc_session->sls_source_ip_address == src_ip_addr)) &&
				  lls_slt_alc_session->sls_destination_ip_address == dst_ip_addr && 
                  lls_slt_alc_session->sls_destination_udp_port == dst_port
              )
            {
                //update alc_session for lls_slt_monitor->lls_sls_alc_monitor
                for(int k=0; k < lls_slt_monitor->lls_sls_alc_monitor_v.count; k++) 
                {
                    lls_sls_alc_monitor_t* lls_sls_alc_monitor = lls_slt_monitor->lls_sls_alc_monitor_v.data[k];
                    if(lls_slt_alc_session->service_id == lls_sls_alc_monitor->atsc3_lls_slt_service->service_id) {
                        lls_slt_monitor->lls_sls_alc_monitor = lls_sls_alc_monitor;
                    }
                }
		
		return lls_slt_alc_session;
            }
        }
        
    }

#### atsc3_listener_metrics_ncurses.cpp::route_parse_from_udp_packet(matching_lls_slt_alc_session, ...)
    ...
    //process ALC streams
    alc_rx_analyze_packet_a331_compliant(..., &alc_packet);
    ...
    return alc_packet;

![0001](/atsc3/res/route_alc.png)

***

#### The TOI value for SLS packages:
![0002](/atsc3/res/alc_toi.png)





