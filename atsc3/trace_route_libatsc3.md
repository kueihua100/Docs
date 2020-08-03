# ROUTE flow of libatsc3
#### atsc3_listener_metrics_ncurses.cpp::process_packet()
    //strip out udp packet
    process_packet_from_pcap();
    
    //prcoess lls (low level signaling)
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
        alc_packet = route_parse_from_udp_packet(matching_lls_slt_alc_session, udp_packet);
        if (alc_packet)
            route_process_from_alc_packet(xxx, &alc_packet);
        ...
        return udp_packet_free(&udp_packet);
    }

#### atsc3_lls_alc_utils.c::lls_slt_alc_session_find_from_udp_packet(lls_slt_monitor, src_ip_addr, dst_ip_addr, dst_port)
    for(int i=0; i < lls_slt_monitor->lls_sls_alc_session_flows_v.count; i++)
    {
        lls_sls_alc_session_flows = lls_slt_monitor->lls_sls_alc_session_flows_v.data[i];
        for(int j=0; j < lls_sls_alc_session_flows->lls_sls_alc_session_v.count; j++ )
        {
            lls_slt_alc_session = lls_sls_alc_session_flows->lls_sls_alc_session_v.data[j];
            if ((lls_slt_alc_session->sls_relax_source_ip_check || 
                (!lls_slt_alc_session->sls_relax_source_ip_check && 
                  lls_slt_alc_session->sls_source_ip_address == src_ip_addr)) &&
                  lls_slt_alc_session->sls_destination_ip_address == dst_ip_addr && 
                  lls_slt_alc_session->sls_destination_udp_port == dst_port)
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
    if (matching_lls_slt_alc_session->alc_session)
    {
        //process ALC streams
        alc_rx_analyze_packet_a331_compliant(..., &alc_packet);
        ...
    }
    
    return alc_packet;


***
![0001](/atsc3/res/route_alc.png)
***
#### The TOI value for SLS packages:
![0002](/atsc3/res/alc_toi.png)
***

#### atsc3_listener_metrics_ncurses.cpp::route_process_from_alc_packet(xxx, &alc_packet)
    //[note]
    // *) dump ROUTE sls signaling: route/ip.port.tsi-toi 
    // *) dump ROUTE media data (A/V/T) per ALC packet base to: route/ip.port.tsi-toi.recovering 
    // *) call alc_process_done callback function if set
    atsc3_alc_packet_persist_to_toi_resource_process_sls_mbms_and_emit_callback(..., alc_packet, ...);
    ...
    //[note]
    // pack media data into a file
    // *) below API: pack distinct recovering files into one file
    //dump_media_from_recover_file(udp_flow, *alc_packet, lls_slt_monitor->lls_sls_alc_monitor);
    // *) below API: pack media alc packet into one file
    dump_media_from_alc_packet(udp_flow, *alc_packet, lls_slt_monitor->lls_sls_alc_monitor);

***
![](/atsc3/res/route_packet.png)
***

#### atsc3_alc_utils.c::atsc3_alc_packet_persist_to_toi_resource_process_sls_mbms_and_emit_callback(..., alc_packet, ...)
    ...
    // file name: route/ip.port.tsi-toi.recovering 
    temporary_filename = alc_packet_dump_to_object_get_temporary_recovering_filename();
    ...
    //[note]
    //alc_packet->transfer_len: the total data length will be transfer at one or multiple alc_packets
    //alc_packet->alc_len: the total data length that remain in this alc_packet
    
    if (alc_packet->use_sbn_esi) 
    {
        //write fec alc data to: ip.port.tsi-toi.recovering 
        if (!alc_packet->esi)
            //write 0 alc_packet->transfer_len size to f
            f = alc_object_pre_allocate();
        else 
            f = alc_object_open_or_pre_allocate();
            
        //write alc_packet->alc_len size of alc_packet->alc_payload to f
        alc_packet_write_fragment(f, ..., alc_packet->esi, ...);
    } 
    else if (alc_packet->use_start_offset) 
    {
        //write non-fec alc data to:  ip.port.tsi-toi.recovering 
        if (!alc_packet->start_offset)
            //write 0 alc_packet->transfer_len size to f
            f = alc_object_pre_allocate();
        else
            f = alc_object_open_or_pre_allocate();
        
        //write alc_packet->alc_len size of alc_packet->alc_payload to f
        alc_packet_write_fragment(f, ..., alc_packet->start_offset, ...);
    }
    ...
    
    //if current alc is the last fragement of ROUTE sls or video/audio/subtitle data
    if (alc_packet->close_object_flag)
    {
        //sls signaling (tsi == 0)
        if (lls_sls_alc_monitor && lls_sls_alc_monitor->atsc3_lls_slt_service &&  alc_packet->def_lct_hdr->tsi == 0)
        {
            // file name: route/ip.port.0-toi
            final_mbms_toi_filename = alc_packet_dump_to_object_get_filename_tsi_toi();
            
            //rename "ip.port.0-toi.recovering" to "ip.port.0-toi"
            rename(temporary_filename, final_mbms_toi_filename);
            ...
            atsc3_route_sls_process_from_alc_packet_and_file(..., lls_sls_alc_monitor);
        }
        else
        {
            //get file name from STSID.RS.LS.SrcFlow.EFDT.FDT-Instance.File@Content-Location
            //or if not found will return "ip.port.tsi-toi"
            s_tsid_content_location = alc_packet_dump_to_object_get_s_tsid_filename();
            if (strncmp(temporary_filename, s_tsid_content_location, ...) != 0)
            {
                //create new_file_name as file name: route/service_id/{audio,video,text}/xxx.yyy
                ...
                //rename "ip.port.tsi-toi.recovering" to "route/service_id/{audio,video,text}/xxx.yyy"
                rename(temporary_filename, new_file_name);
                ...
                //call alc_process_done callback function if set
                lls_sls_alc_monitor->atsc3_lls_sls_alc_on_object_close_flag_s_tsid_content_location();
            }
        }
    }

***
![0003](/atsc3/res/route_fdt.png)
***    

#### atsc3_route_sls_processor.c::atsc3_route_sls_process_from_alc_packet_and_file(..., lls_sls_alc_monitor)
    ...
    //An Extended FDT transported in the LCT channel referencing broadcast SLS fragments 
    //with TOI=0 shall be present
    if (alc_packet->def_lct_hdr->toi == 0)
    {
        //file name: ip.port.0-0
        file_name = alc_packet_dump_to_object_get_filename_tsi_toi(udp_flow, 0, 0);
        ...
        fp = fopen(file_name, "r");
        ...
        //open efdt (from ip.port.0-0) for later parsing
        fdt_xml = xml_open_document(fp);
        if (fdt_xml)
        {
            //parsing efdt (from ip.port.0-0) to atsc3_fdt_instance
            atsc3_fdt_instance = atsc3_fdt_instance_parse_from_xml_document(fdt_xml);
            -> atsc3_fdt_parser.c::atsc3_fdt_instance_parse_from_xml_document()
                ...
                atsc3_fdt_instance = atsc3_efdt_instance_parse_from_xml_node();
                -> atsc3_fdt_parser.c::atsc3_efdt_instance_parse_from_xml_node()
                    {
                        ...
                        if (xml_node_equals_ignore_case(root_child, "FDT-Instance"))
                        {
                            //parsing FDT-Instance node
                            atsc3_fdt_parse_from_xml_fdt_instance(atsc3_fdt_instance, ...);
                            ...
                            for (num_fdt_children)
                            {
                                if (xml_node_equals_ignore_case(fdt_child, "File"))
                                {
                                    atsc3_fdt_file = atsc3_fdt_file_parse_from_xml_fdt_instance();
                                    ...
                                    atsc3_fdt_instance_add_atsc3_fdt_file(atsc3_fdt_instance, atsc3_fdt_file);
                                }
                            }
                            
                        }
                        return atsc3_fdt_instance;
                    }
            ...
            lls_sls_alc_monitor->atsc3_fdt_instance = atsc3_fdt_instance;
        }
    }
    else
    {
        if (lls_sls_alc_monitor && lls_sls_alc_monitor->atsc3_fdt_instance)
        {
            //get toi from lls_sls_alc_monitor->atsc3_fdt_instance
            mbms_toi = atsc3_mbms_envelope_find_toi_from_fdt(lls_sls_alc_monitor->atsc3_fdt_instance);
            ...
            //file name: ip.port.0-mbms_toi
            mbms_toi_filename = alc_packet_dump_to_object_get_filename_tsi_toi(udp_flow, 0, *mbms_toi);
            
            //open mbms envelope xml for later parsing
            fp_mbms = fopen(mbms_toi_filename, "r");
            ...
            //parsing mbms envelope xml
            atsc3_sls_metadata_fragments = atsc3_mbms_envelope_to_sls_metadata_fragments_parse_from_fdt_fp(fp_mbms);
            if (atsc3_sls_metadata_fragments)
            {
                if (atsc3_sls_metadata_fragments->atsc3_route_s_tsid)
                {
                    ...
                    //update our audio and video tsi and toi_init
                    lls_sls_alc_update_tsi_toi_from_route_s_tsid();
                }
                ...
                lls_sls_alc_monitor->atsc3_sls_metadata_fragments = atsc3_sls_metadata_fragments;
                ...
                //loop to find mpd.xml
                for (int i=0; i < lls_sls_alc_monitor->atsc3_sls_metadata_fragments->atsc3_mime_multipart_related_instance->atsc3_mime_multipart_related_payload_v.count; i++) 
                {
                    ...
                    //ATSC3_ROUTE_MPD_TYPE = "application/dash+xml"
                    if (strncmp(atsc3_mime_multipart_related_payload->content_type, ATSC3_ROUTE_MPD_TYPE, 
                        __MIN(strlen(atsc3_mime_multipart_related_payload->content_type), strlen(ATSC3_ROUTE_MPD_TYPE))) == 0)
                    {
                        //[note] 
                        // patch mpd.xml's MPD type="dynamic" with "availabilitystarttime=" to 
                        // current NOW and "startnumber=" to the most recent A/V flows for TOI delivery
                        atsc3_route_sls_patch_mpd_availability_start_time_and_start_number();
                    }
                }
            }
            ...
        }
    }


***
![004](/atsc3/res/route_mbms_envelope.png)
***

#### atsc3_mime_multipart_related_parser.c::atsc3_mbms_envelope_to_sls_metadata_fragments_parse_from_fdt_fp()
    ...
    //[note]
    //*1) check "Content-Type: multipart/related" and then "boundary="
    //*2) check "Content-Type" and "Content-Location" amid "boundary=" to find envelope.xml, usbd.xml, stsid.xml, ...
    //      and then copy related payload for later parsing below
    atsc3_mime_multipart_related_instance = atsc3_mime_multipart_related_parser()
    ...
    if (atsc3_mime_multipart_related_instance)
    {
        ...
        atsc3_sls_metadata_fragments = 
            atsc3_sls_metadata_fragment_types_parse_from_mime_multipart_related_instance(atsc3_mime_multipart_related_instance);
            -> atsc3_sls_metadata_fragment_types_parser.c::xxx(atsc3_mime_multipart_related_instance)
                {
                    ...
                    //parsing envelope.xml
                    atsc3_mbms_envelope_parse_from_payload();
                    ...
                    //parsing usbd.xml
                    atsc3_route_user_service_bundle_description_parse_from_payload()
                    ...
                    //parsing stsid.xml
                    atsc3_route_s_tsid_parse_from_payload();
                    ...
                    //parsing mpd.xml
                    atsc3_route_mpd_parse_from_payload();
                    ...
                    //parsing held.xml
                    atsc3_sls_held_fragment_parse_from_payload();
                    ...
                }
    }
    ...
    return atsc3_sls_metadata_fragments;

***
![](/atsc3/res/route_usbd.png)
***
![](/atsc3/res/route_stsid.png)
***
![](/atsc3/res/route_held.png)
***

#### atsc3_lls_alc_utils.c::lls_sls_alc_update_tsi_toi_from_route_s_tsid()
    ...
    if (strncasecmp("audio", src_flow_content_info_content_type, 5) == 0 && 
        !lls_sls_alc_monitor->audio_tsi_manual_override) 
    {
        if (!lls_sls_alc_monitor->audio_tsi_manual_override)
        {
            //update our audio tsi and toi accordingly
            lls_sls_alc_monitor->audio_toi_init = atsc3_fdt_file->toi;
            lls_sls_alc_monitor->audio_tsi = atsc3_route_s_tsid_RS_LS->tsi;
        }
    } 
    else if (strncasecmp("video", src_flow_content_info_content_type, 5) == 0)
    {
        if (!lls_sls_alc_monitor->video_tsi_manual_override) 
        {
            lls_sls_alc_monitor->video_toi_init = atsc3_fdt_file->toi;
            lls_sls_alc_monitor->video_tsi = atsc3_route_s_tsid_RS_LS->tsi;
        }
    }
    else if(strncasecmp("text", src_flow_content_info_content_type, 4) == 0) 
    {
        if (!lls_sls_alc_monitor->text_tsi_manual_override) 
        {
            lls_sls_alc_monitor->text_toi_init = atsc3_fdt_file->toi;
            lls_sls_alc_monitor->text_tsi = atsc3_route_s_tsid_RS_LS->tsi;
        }
    }
    ...

