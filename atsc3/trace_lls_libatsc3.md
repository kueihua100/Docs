# lls flow of libatsc3
#### atsc3_lls_slt_parser.c::lls_slt_monitor_create()
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

#### atsc3_block
    typedef struct atsc3_block {
        uint8_t* p_buffer;  // currently write point
        uint32_t p_size;     // allocated buffer size
        uint32_t i_pos;      // currently read position
        uint8_t  _bitpos;
        uint8_t  _refcnt;
        uint8_t  _is_alloc;
    } block_t;

#### atsc3_listener_metrics_ncurses.cpp::pcap_loop_run_thread_with_file()
    ...
    pcap_open_offline(filename, errbuf); 
    ...
    pcap_loop(descr,-1,process_packet,NULL);

![11](/atsc3/res/lls.png)

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
    //ALC(ROUTE) process
    matching_lls_slt_alc_session = lls_slt_alc_session_find_from_udp_packet(lls_slt_monitor, ...);
    if(matching_lls_slt_alc_session) {
        ...
        alc_packet = route_parse_from_udp_packet();
        if (alc_packet)
            route_process_from_alc_packet(xxx, &alc_packet);
        ...
    }
    
    //MMTP process
    matching_lls_slt_mmt_session = lls_slt_mmt_session_find_from_udp_packet(lls_slt_monitor, ...);
    if(matching_lls_slt_mmt_session) {
        update_global_mmtp_statistics_from_udp_packet_t(udp_packet);
    }

#### atsc3_lls.c::lls_table_create_or_update_from_lls_slt_monitor_with_metrics(lls_slt_monitor, ...)
    ...
    __lls_table_create();
    ...
    if(lls_table_new->lls_table_id != SignedMultiTable)
        atsc3_lls_table_create_or_update_from_lls_slt_monitor_with_metrics_single_table();
    else
        for (LLS_payload_count)
            atsc3_lls_table_create_or_update_from_lls_slt_monitor_with_metrics_single_table();

#### atsc3_lls.c::__lls_table_create()
    lls_create_xml_table();
    ...
    if(lls_table->lls_table_id != SignedMultiTable)
        atsc3_lls_table_parse_raw_xml();
    else
        for (lls_table->signed_multi_table.atsc3_signed_multi_table_lls_payload_v.count)
            atsc3_lls_table_parse_raw_xml();

#### atsc3_lls.c::lls_create_xml_table()
    lls_table = __lls_create_base_table_raw();
    ...
    if(lls_table->lls_table_id == SignedMultiTable) {
        return lls_table;
    }
    ...
    ret = atsc3_unzip_gzip_payload(); //unzip gziped data
    ...
    lls_table->raw_xml.xml_payload = decompressed_payload;
    lls_table->raw_xml.xml_payload_size = ret;
    return lls_table;

#### atsc3_lls.c::__lls_create_base_table_raw()
    ...
    if(base_table->lls_table_id == SignedMultiTable) {
        base_table->signed_multi_table.lls_payload_count = block_Read_uint8_bitlen();
        
        for (base_table->signed_multi_table.lls_payload_count) 
        {
            //extract lls_table of multi-tables
            lls_payload = atsc3_signed_multi_table_lls_payload_new();
            lls_payload->lls_payload_id = block_Read_uint8_bitlen();
            lls_payload->lls_payload_version = block_Read_uint8_bitlen();
            lls_payload->lls_payload_length = block_Read_uint16_ntohs();
            ...
            lls_payload->lls_payload = block_Duplicate_from_ptr();
            ...
            ret = tsc3_unzip_gzip_payload(..., decompressed_payload); //unzip gziped data
            ...
            //create lls_table for each one of multi-tables
            lls_payload->lls_table = (lls_table_t*)calloc(1, sizeof(lls_table_t));
            lls_payload->lls_table->lls_table_id = lls_payload->lls_payload_id;
            lls_payload->lls_table->lls_group_id = base_table->lls_group_id;
            lls_payload->lls_table->group_count_minus1 = base_table->group_count_minus1;
            lls_payload->lls_table->lls_table_version = lls_payload->lls_payload_version;

            lls_payload->lls_table->raw_xml.xml_payload = decompressed_payload;
            lls_payload->lls_table->raw_xml.xml_payload_size = ret;
            ...
            atsc3_signed_multi_table_add_atsc3_signed_multi_table_lls_payload(&base_table->signed_multi_table, lls_payload);
            ...
        }
        ...
        //extract signature length and signature
        base_table->signed_multi_table.signature_length = block_Read_uint16_ntohs();
        ...
        base_table->signed_multi_table.signature = block_Duplicate_from_position();
        ...
    }
    else
    {
        uint8_t *temp_gzip_payload = (uint8_t*)calloc();
        memcpy(temp_gzip_payload, block_Get(lls_packet_block), ...);
        ...
        base_table->raw_xml.xml_payload_compressed = temp_gzip_payload;
        base_table->raw_xml.xml_payload_compressed_size = remaining_payload_size;
    }
    return base_table;

#### atsc3_lls.c::atsc3_lls_table_parse_raw_xml()
    ...
    //parsing xml file
    xml_payload_document_parse();
    ...
    //according lls table type to create related table structure, ie SLT, ...
    lls_create_table_type_instance();
    -> atsc3_lls.c::lls_create_table_type_instance()
        ...
        if(lls_table->lls_table_id == SLT)
            lls_slt_table_build();
        else if(lls_table->lls_table_id == RRT)
            build_rrt_table();  ** //NOT IMPLEMENTED **
        else if(lls_table->lls_table_id == SystemTime)
            build_system_time_table();
        else if(lls_table->lls_table_id == AEAT)
            atsc3_aeat_table_populate_from_xml();  ** //looks like NOT IMPLEMENTED DONE **
        else if(lls_table->lls_table_id == OnscreenMessageNotification)
            build_onscreen_message_notification_table();  ** //NOT IMPLEMENTED **

![22](/atsc3/res/lls_slt.png)


***
#### atsc3_lls.c::atsc3_lls_table_create_or_update_from_lls_slt_monitor_with_metrics_single_table(lls_slt_monitor, lls_table_new, ...)
    ...
    if(lls_table_new->lls_table_id == AEAT)
    {
        //update last successfully processed AEAT table
        lls_slt_monitor->lls_latest_aeat_table = lls_table_new;
        ...
        return NULL;
    }
    
    if(lls_table_new->lls_table_id == OnscreenMessageNotification)
    {
        //update last successfully processed on screen message notification table
        lls_slt_monitor->lls_latest_on_screen_message_notification_table = lls_table_new;
        ...
        return NULL;
    }
    
    if(lls_table_new->lls_table_id != SLT)
    {
        ...
        return NULL;
    }
    
    //upate last successfully processed SLT table:
    //a) if lls_table_new is same with lls_slt_monitor->lls_latest_slt_table, 
    //      just free lls_table_new and return NULL
    //b) if lls_table_new is different with lls_slt_monitor->lls_latest_slt_table, 
    //      free lls_slt_monitor->lls_latest_slt_table and assign lls_table_new to it
    lls_slt_monitor->lls_latest_slt_table = lls_table_new;
    lls_slt_table_perform_update(lls_table_new, lls_slt_monitor);
    ...
    return lls_slt_monitor->lls_latest_slt_table;

#### atsc3_lls_slt_parser.c::lls_slt_table_perform_update(lls_table, lls_slt_monitor)
    ...
    lls_slt_monitor_add_or_update_lls_slt_service_id_group_id_cache_entry();
    if(atsc3_lls_slt_service->atsc3_slt_broadcast_svc_signalling_v.count) {
        ...
        if(atsc3_lls_slt_service->atsc3_slt_broadcast_svc_signalling_v.data[0]->sls_protocol == SLS_PROTOCOL_ROUTE) 
        {
            ...
            lls_sls_alc_session = lls_slt_alc_session_find_or_create(lls_slt_monitor, atsc3_lls_slt_service);
            ...
        } 
        else if (atsc3_lls_slt_service->atsc3_slt_broadcast_svc_signalling_v.data[0]->sls_protocol == SLS_PROTOCOL_MMTP)
        {
            ...
            lls_sls_mmt_session = lls_slt_mmt_session_find_or_create(lls_slt_monitor, atsc3_lls_slt_service);
            ...
        }
    } 
