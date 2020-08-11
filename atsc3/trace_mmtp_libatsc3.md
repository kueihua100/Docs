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
        //Sync code from atsc3_phy_mmt_player_bridge.cpp::atsc3_phy_mmt_player_bridge_process_packet_phy()
        mmtp_parse_from_udp_packet(udp_packet);
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

#### atsc3_listener_metrics_ncurses.cpp::mmtp_parse_from_udp_packet(udp_packet)
    ...
    //[note]
    //parsing mmtp packet header
    mmtp_packet_header = mmtp_packet_header_parse_from_block_t();
    ...
    //[note] mmtp_payload_type
    //0x0: a media-aware fragment of the MPU
    //0x2: one or more signalling messages or a fragment of a signalling message
    if (mmtp_packet_header->mmtp_payload_type == 0x0)
    {
        //mmtp_mpu_packet = mmtp_mpu_packet_parse_from_block_t(mmtp_packet_header, ...);
        mmtp_mpu_packet = mmtp_mpu_packet_parse_and_free_packet_header_from_block_t(&mmtp_packet_header, udp_packet->data);
        -> atsc3_mmt_mpu_parser.c::mmtp_mpu_packet_parse_and_free_packet_header_from_block_t()
            {
                ...
                mmtp_mpu_packet = mmtp_mpu_packet_parse_from_block_t(mmtp_packet_header, udp_packet);
                ...
                return mmtp_mpu_packet;
            }
        ...
        if (mmtp_mpu_packet->mpu_timed_flag == 1)
        {
            atsc3_packet_statistics_mmt_stats_populate();
        }
    }
    else if (mmtp_packet_header->mmtp_payload_type == 0x2)
    {
        mmtp_signalling_packet = mmtp_signalling_packet_parse_and_free_packet_header_from_block_t();
        -> atsc3_mmt_signalling_message.c::mmtp_signalling_packet_parse_and_free_packet_header_from_block_t()
            {
                ...
                mmtp_signalling_packet = mmtp_signalling_packet_parse_from_block_t(mmtp_packet_header, udp_packet);
                ...
                return mmtp_signalling_packet;
            }
             
        mmt_signalling_message_parse_packet(mmtp_signalling_packet, ...);
        ...
    }
    ...

***
![](/atsc3/res/mmtp_1.png)
***
![](/atsc3/res/mmtp_2.png)
***
![](/atsc3/res/mmtp_3.png)
***

#### atsc3_mmt_signalling_message.c::mmtp_signalling_packet_parse_from_block_t()
    //parsing header for signalling message mode
    //[note]
    /* 
     * f_i: bits 0-1 fragmentation indicator:
     * 0x00 = payload contains one or more complete signalling messages
     * 0x01 = payload contains the first fragment of a signalling message
     * 0x10 = payload contains a fragment of a signalling message that is neither first/last
     * 0x11 = payload contains the last fragment of a signalling message
     */
     
     //next 4 bits is reserved field with 0x0 value
     //bit 6 is additional Header
     //bit 7 is Aggregation
     //bits 8~15: count of for how many fragments follow this message, e.g si_fragmentation_indiciator != 0

***
![](/atsc3/res/mmtp_4.png)
***

#### atsc3_mmt_signalling_message.c::mmt_signalling_message_parse_packet(mmtp_signalling_packet, ...)
    ...
    if (mmtp_signalling_packet->si_aggregation_flag)
    {
        while(block_Remaining_size(udp_packet))
        {
            if (mmtp_signalling_packet->si_additional_length_header)
            {
                //read the full 32 bits for MSG_length = 16+16*H
                buf = extract(buf, (uint8_t*)&mmtp_aggregation_msg_length, 4);
            }
            else
            {
                //only read 16 bits for MSG_length
                buf = extract(buf, (uint8_t*)&aggregation_msg_length_short, 2);
            }
            ...
            //build a msg from buf to buf+mmtp_aggregation_msg_length
            mmt_signalling_message_parse_id_type(mmtp_signalling_packet, udp_packet);
        }
    }
    else if (udp_packet_size)
    {
        //fragmentation indicator
        //00: Payload contains one or more complete signalling messages.
        //01: Payload contains the first fragment of a signalling message.
        //10: Payload contains a fragment of a signalling message that is 
        //    neither the first nor the last fragment.
        //11: Payload contains the last fragment of a signalling message.
        
        //[note] skip f_i = `10` | `11` cases: 
        if ((mmtp_signalling_packet->si_fragmentation_indiciator == 2) ||
            (mmtp_signalling_packet->si_fragmentation_indiciator == 3))
        {
            __MMSM_ERROR("%s: IS SUPPORTED for f_i = %d cases ??? SKIP !!!", 
                __FUNCTION__, mmtp_signalling_packet->si_fragmentation_indiciator);
            return (buf != udp_raw_buf);
        }

        //parse a single message
        mmt_signalling_message_parse_id_type(mmtp_signalling_packet, udp_packet);
    }
    
    
#### atsc3_mmt_signalling_message.c::mmt_signalling_message_parse_id_type(mmtp_signalling_packet, udp_packet)
    ...
    //read "message_id" from general signalling message format
    buf = extract(buf, (uint8_t*)&message_id, 2);
    message_id = ntohs(message_id);
    ...
    //read "version" from general signalling message format
    buf = extract(buf, &version, 1);
    ...
    //create general signalling message structure
    mmt_signalling_message_header_and_payload = 
        mmt_signalling_message_header_and_payload_create(message_id, version);
    ...
    mmt_signalling_message_header = &mmt_signalling_message_header_and_payload->message_header;
    ...
    if (mmt_signalling_message_header->message_id == PA_message)
    {
        pa_message_parse(mmt_signalling_message_header_and_payload, udp_packet);
        -> //NOT SUPPORTED!!
    } 
    else if (mmt_signalling_message_header->message_id >= MPI_message_start && 
               mmt_signalling_message_header->message_id < MPI_message_end)
    {
        mpi_message_parse(mmt_signalling_message_header_and_payload, udp_packet);
        -> //NOT SUPPORTED!!
    }
    else if (mmt_signalling_message_header->message_id >= MPT_message_start && 
               mmt_signalling_message_header->message_id <= MPT_message_end)
    {
    	// 0x11 <= message_id <= 0x20
        mpt_message_parse(mmt_signalling_message_header_and_payload, udp_packet);
    }
    else if (mmt_signalling_message_header->message_id == MMT_ATSC3_MESSAGE_ID)
    {
    	// message_id=0x8100
        mmt_atsc3_message_payload_parse(mmt_signalling_message_header_and_payload, udp_packet);
    }
    else if (mmt_signalling_message_header->message_id == MMT_SCTE35_Signal_Message)
    {
    	// message_id=0xF337
        mmt_scte35_message_payload_parse(mmt_signalling_message_header_and_payload, udp_packet);
    }
    else
    {
        //NOT SUPPORTED!!
    }

#### atsc3_mmt_mpu_parser.c::mmtp_mpu_packet_parse_from_block_t(mmtp_packet_header, udp_packet)
    //Parsing MPU packet:
    /*   MMTP payload header for MPU mode
    *     -----------------------------------------------------
    *     | length                |  FT |T|f_i|A| frag_counter |
    *     -----------------------------------------------------
    *     |                  MPU_sequence_number               |
    *     -----------------------------------------------------
    *     |    DU_length        |      DU_Header               |
    *     -----------------------------------------------------
    *     |                       DU payload  ....              
    *     -----------------------------------------------------
    */
    
    ...
    //loop to parse DU if exists, or just copy data
    do
    {
        ...
        if (mmtp_mpu_packet->mpu_aggregation_flag) {
            //only read DU length if mpu_aggregation_flag=1
            to_read_packet_length = mmtp_mpu_packet->data_unit_length;
            ...
        } else {
            //set remain bytes of packet to be the read length later: to_read_packet_length
            to_read_packet_length = mmtp_mpu_payload_length - (buf - udp_raw_buf);
            ...
        }
        
        if (mmtp_mpu_packet->mpu_fragment_type == 0x0) {
            ...
            //just copy MPU metadata
            block_Write(mmtp_mpu_packet->du_mpu_metadata_block, buf, to_read_packet_length);
        } else if (mmtp_mpu_packet->mpu_fragment_type == 0x2) {
            //MFU parsing
            ...
            //parsing media sample, then assign to mmtp_mpu_packet->du_mfu_block
            atsc3_mmt_mpu_sample_format_parse(mmtp_mpu_packet, temp_timed_buffer);
            ...
        } else if (mmtp_mpu_packet->mpu_fragment_type == 0x1) {
            ...
            //just copy movie fragment metadta
            block_Write(mmtp_mpu_packet->du_movie_fragment_block, buf, to_read_packet_length);
        }
        ...
        //move buf ptr
        buf += to_read_packet_length;
        //compute the remaining bytes of this packet 
        remaining_du_packet_len = mmtp_mpu_payload_length - (buf - udp_raw_buf);
        ...
    } while(mmtp_mpu_packet->mpu_aggregation_flag && remaining_du_packet_len > 0);


#### atsc3_mmt_mpu_sample_format_parser.c::atsc3_mmt_mpu_sample_format_parse(mmtp_mpu_packet, udp_packet)
    //Parsing DU packet:
    /*   DU header for timed-media MFU
    *     -----------------------------------------------------
    *     |  movie_fragement_sequence_number                  |
    *     -----------------------------------------------------
    *     |     sample_number                                 |
    *     -----------------------------------------------------
    *     |      offset                                       |
    *     -----------------------------------------------------
    *     | priority | offset      |
    *     -----------------------------------------------------
    */

    /*   DU header for non-timed media MFU
    *     -----------------------------------------------------
    *     |                         item_ID                   |
    *     -----------------------------------------------------
    */

    /* From ISO 23800-1:
    * MFU mpu_fragmentation_indicator==1's are prefixed by the following box, need to remove and process
    *
    *   aligned(8) class MMTHSample {
    *      unsigned int(32) sequence_number;
    *      if (is_timed) {
    *
    *       //interior block is 152 bits, or 19 bytes
    *         signed int(8) trackrefindex;
    *         unsigned int(32) movie_fragment_sequence_number
    *         unsigned int(32) samplenumber;
    *          unsigned int(8)  priority;
    *         unsigned int(8)  dependency_counter;
    *         unsigned int(32) offset;
    *         unsigned int(32) length;
    *       //end interior block
    *
    *         multiLayerInfo();
    *   } else {
    *           //additional 2 bytes to chomp for non timed delivery
    *         unsigned int(16) item_ID;
    *      }
    *   }
    *
    *   aligned(8) class multiLayerInfo extends Box("muli") {
    *      bit(1) multilayer_flag;
    *      bit(7) reserved0;
    *      if (multilayer_flag==1) {
    *          //32 bits
    *         bit(3) dependency_id;
    *         bit(1) depth_flag;
    *         bit(4) reserved1;
    *         bit(3) temporal_id;
    *         bit(1) reserved2;
    *         bit(4) quality_id;
    *         bit(6) priority_id;
    *      }  bit(10) view_id;
    *      else{
    *           //16bits
    *          bit(6) layer_id;
    *          bit(3) temporal_id;
    *         bit(7) reserved3;
    *   } }
    */
    
    ...
    if (mmtp_mpu_packet->mpu_timed_flag) {
        //parse out DU header for timed-media MFU, 14 bytes:
            //[movie_fragment_sequence_number]
            //[sample_number]
            //[offset]
            //[priority]
            //[dep_counter]
        if ( mmtp_mpu_packet->mpu_fragment_type == 2 &&
            (mmtp_mpu_packet->mpu_fragmentation_indicator == 0 || 
             mmtp_mpu_packet->mpu_fragmentation_indicator  == 1) )
        {
            //parse out mmthsample block, (4+19) bytes:
                //[sequence_number]
                //[trackrefindex]
                //[movie_fragment_sequence_number]
                //[samplenumber]
                //[priority]
                //[dependency_counter]
                //[offset]
                //[length]
            //parse out multiLayerInfo box
                //??
        }
    } else {
        //DU header for non-timed media MFU
    }





