# Demo for SAP Fiori Elements
Demonstrate SAP Fiori Elements (formerly known as: Smart Templates) without annotaded CDS View based OData Service on NetWeaver ABAP 7.40.

To make this example work the SAP delivered standard OData Service /IWBEP/EPM_DEVELOPER_SCENARIO must be redefined as ZEPM_DEVELOPER_SCENARIO and the products_get_entity method must be implemented with the following code:

```
CLASS zcl_zepm_developer_sce_dpc_ext DEFINITION
  PUBLIC
  INHERITING FROM zcl_zepm_developer_sce_dpc
  CREATE PUBLIC .

  PUBLIC SECTION.
  PROTECTED SECTION.

    METHODS businesspartners_get_entity
        REDEFINITION .
    METHODS businesspartners_get_entityset
        REDEFINITION .
    METHODS products_get_entity
        REDEFINITION .
  PRIVATE SECTION.
ENDCLASS.

CLASS ZCL_ZEPM_DEVELOPER_SCE_DPC_EXT IMPLEMENTATION.

  METHOD businesspartners_get_entity.
    DATA bp_id       TYPE bapi_epm_bp_id.
    DATA headerdata  TYPE bapi_epm_bp_header.
    DATA contactdata TYPE STANDARD TABLE OF bapi_epm_bp_contact.
    DATA return      TYPE STANDARD TABLE OF bapiret2.

    DATA(lt_keys) = io_tech_request_context->get_keys( ).
    " BP_ID  0100000000
    READ TABLE lt_keys ASSIGNING FIELD-SYMBOL(<fs_key>) WITH KEY name = 'BP_ID'.
    IF <fs_key> IS ASSIGNED.
      bp_id = <fs_key>-value.

      CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
        EXPORTING
          input  = bp_id
        IMPORTING
          output = bp_id.

      CALL FUNCTION 'BAPI_EPM_BP_GET_DETAIL'
        EXPORTING
          bp_id       = bp_id
        IMPORTING
          headerdata  = headerdata
        TABLES
          contactdata = contactdata
          return      = return.
      READ TABLE contactdata ASSIGNING FIELD-SYMBOL(<fs_contact>) INDEX 1.
      IF <fs_contact> IS ASSIGNED.
        MOVE-CORRESPONDING <fs_contact> TO er_entity.
      ENDIF.
      MOVE-CORRESPONDING headerdata TO er_entity.
      er_entity-date_of_birth = sy-datum.

    ENDIF.
  ENDMETHOD.
  
  METHOD businesspartners_get_entityset.

    DATA bpheaderdata        TYPE STANDARD TABLE OF bapi_epm_bp_header.
    DATA bpcontactdata       TYPE STANDARD TABLE OF bapi_epm_bp_contact.

    CALL FUNCTION 'BAPI_EPM_BP_GET_LIST'
      TABLES
        bpheaderdata  = bpheaderdata
        bpcontactdata = bpcontactdata.

    LOOP AT bpheaderdata ASSIGNING FIELD-SYMBOL(<fs_bp_head>).
      APPEND INITIAL LINE TO et_entityset ASSIGNING FIELD-SYMBOL(<fs_entity>).
      MOVE-CORRESPONDING <fs_bp_head> TO <fs_entity>.
      <fs_entity>-date_of_birth = sy-datum.
    ENDLOOP.

  ENDMETHOD.
  
  METHOD products_get_entity.
    DATA: lt_products         TYPE sepm_gws_product_header_t,
          lt_product_id_range TYPE sepm_gws_product_id_range_t.

    READ TABLE it_key_tab ASSIGNING FIELD-SYMBOL(<fs_key>) INDEX 1.
    IF <fs_key> IS ASSIGNED.
      APPEND INITIAL LINE TO lt_product_id_range
        ASSIGNING FIELD-SYMBOL(<fs_product_id_range>).

      <fs_product_id_range>-option = 'EQ'.
      <fs_product_id_range>-sign = 'I'.
      <fs_product_id_range>-low = <fs_key>-value.

      CALL FUNCTION 'SEPM_GWS_PRODUCTS_GET'
        EXPORTING
          iv_supply_picture   = abap_true
          it_product_id_range = lt_product_id_range
        IMPORTING
          et_product_list     = lt_products.
      READ TABLE lt_products ASSIGNING FIELD-SYMBOL(<fs_product>) INDEX 1.
      MOVE-CORRESPONDING <fs_product> TO  er_entity.
    ENDIF.
  ENDMETHOD.

ENDCLASS.
```
