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

    METHODS products_get_entity
        REDEFINITION .
  PRIVATE SECTION.
ENDCLASS.

CLASS ZCL_ZEPM_DEVELOPER_SCE_DPC_EXT IMPLEMENTATION.

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
