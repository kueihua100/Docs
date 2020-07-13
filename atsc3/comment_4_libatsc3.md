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
