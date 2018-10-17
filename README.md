*&---------------------------------------------------------------------*
*& Report ZTESTE_PROG
*&---------------------------------------------------------------------*
*&INSTRUÇÕES
*&
*& Você está coreografando um show. Para um ato em especifico, você recebe
*& duas bolas em uma linha numérica, e essas bolas estão prontas para serem
*& chutadas na direção positiva (ou seja, em direção ao infinito positivo).
*& - A primeira bola é chutada da posição x1 e quica na proporção de v1 metros por quique.
*& - A segunda bola é chutada da posição x2 e quica na proporção v2 metros por quique.
*& Sabendo disso, você tem o desafio de desenvolver um programa que calcule uma
*& maneira de obter se existe a possibilidade das duas bolas quicarem no mesmo local,
*& ao mesmo tempo. Sendo isso possível, então retorne SIM É POSSÍVEL, caso contrário
*& retorne NÃO É POSSÍVEL.
*& Por exemplo:
*& Um bola parte do ponto x1 = 2 com um quique de distancia v1 = 1, a outra bola parte
*& do ponto x2 = 1 com um quique de distancia v2 = 2. Depois do primeiro quique,
*& ambas as bolas estarão na posição x3 = 3 (x1 + v1 = 2 + 1, x2 + v2 = 1 + 2),
*& logo dado esses inputs no programa, a resposta deveria ser: SIM É POSSÍVEL.
*& DESCRIÇÃO DO SISTEMA
*& - Um sistema que dado as entradas, ele retorne SIM É POSSÍVEL caso as bolas sejam
*& capazes de atingir em algum momento a mesma posição ao mesmo tempo, ou NÃO É POSSÍVEL
*& caso isso nunca aconteça.
*& - O input no sistema se dará por um arquivo TXT com uma linha quatro (4) números
*& inteiros separados por espaço denotando respectivamente x1, v1 e x2, v2.
*& - O output se dará por meio de um ALV, caso a resposta seja positiva a célula
*& do ALV deverá estar de uma cor, caso a resposta seja negativa, a célula do ALV
*& deverá estar de outra cor.
*& Exemplo:
*& Uma Tela de apresentação do sistema e um input para realizar o upload do
*& arquivo TXT, que contém o seguinte conteúdo:
*& 0 3 4 2
*& Após execução do programa a saída nesse caso deveria ser:
*& SIM É POSSÍVEL, em um ALV em outra tela seguindo o padrão de cor para a
*& célula em casos de sucesso.
*&
*&---------------------------------------------------------------------*
REPORT ZTESTE_PROG.

*&---------------------------------------------------------------------*
*& TIPOS
*&---------------------------------------------------------------------*
TYPES: BEGIN OF TY_FILE,
         LINE(128) TYPE C,
       END OF TY_FILE,

       BEGIN OF TY_INPUT,
         POSX1(3) TYPE C, "posição X1
         POSX2(3) TYPE C, "posição X2
         PROV1(3) TYPE C, "proporção V1
         PROV2(3) TYPE C, "proporção V2
       END OF TY_INPUT,

       BEGIN OF TY_ALV,
         POSQ(3)  TYPE C, "posição do quique
         RESP(15) TYPE C, "resposta SIM/NÃO
         COLOR    TYPE LVC_T_SCOL,
       END OF TY_ALV.

*&---------------------------------------------------------------------*
*& TABELAS INTERNAS
*&---------------------------------------------------------------------*
DATA: T_FILE  TYPE TABLE OF TY_FILE,
      T_INPUT TYPE TABLE OF TY_INPUT,
      T_ALV   TYPE TABLE OF TY_ALV.

*&---------------------------------------------------------------------*
*& VARIÁVEIS GLOBAIS
*&---------------------------------------------------------------------*
DATA: V_COR TYPE C.

*&---------------------------------------------------------------------*
*& TELA DE SELEÇÃO
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK B1 WITH FRAME TITLE TEXT-001.
PARAMETERS: P_FILE(128) TYPE C OBLIGATORY.
SELECTION-SCREEN END OF BLOCK B1.

*&---------------------------------------------------------------------*
*& AT SELECTION-SCREEN
*&---------------------------------------------------------------------*
AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE .

  CALL FUNCTION 'KD_GET_FILENAME_ON_F4'
    EXPORTING
      FIELD_NAME    = 'p_file'
    CHANGING
      FILE_NAME     = P_FILE
    EXCEPTIONS
      MASK_TOO_LONG = 1
      OTHERS        = 2.

*&---------------------------------------------------------------------*
*& PROGRAMA PRINCIPAL
*&---------------------------------------------------------------------*
START-OF-SELECTION.

  PERFORM F_CARREGA_ARQUIVO.

  PERFORM F_PROCESSA_DADOS.

  PERFORM F_GERA_ALV.

*&---------------------------------------------------------------------*
*&      Form  F_CARREGA_ARQUIVO
*&---------------------------------------------------------------------*
*       Carrega arquivo
*----------------------------------------------------------------------*
FORM F_CARREGA_ARQUIVO .

  DATA: LW_FILE  TYPE TY_FILE,
        LW_INPUT TYPE TY_INPUT.

  DATA: LV_FILE TYPE STRING.

  LV_FILE = P_FILE.

  CALL FUNCTION 'GUI_UPLOAD'
    EXPORTING
      FILENAME                = LV_FILE
    TABLES
      DATA_TAB                = T_FILE
    EXCEPTIONS
      FILE_OPEN_ERROR         = 1
      FILE_READ_ERROR         = 2
      NO_BATCH                = 3
      GUI_REFUSE_FILETRANSFER = 4
      INVALID_TYPE            = 5
      NO_AUTHORITY            = 6
      UNKNOWN_ERROR           = 7
      BAD_DATA_FORMAT         = 8
      HEADER_NOT_ALLOWED      = 9
      SEPARATOR_NOT_ALLOWED   = 10
      HEADER_TOO_LONG         = 11
      UNKNOWN_DP_ERROR        = 12
      ACCESS_DENIED           = 13
      DP_OUT_OF_MEMORY        = 14
      DISK_FULL               = 15
      DP_TIMEOUT              = 16
      OTHERS                  = 17.

  IF T_FILE[] IS NOT INITIAL.
    LOOP AT T_FILE INTO LW_FILE.
      SPLIT LW_FILE AT ' ' INTO LW_INPUT-POSX1
                                LW_INPUT-POSX2
                                LW_INPUT-PROV1
                                LW_INPUT-PROV2.

      APPEND LW_INPUT TO T_INPUT.
      CLEAR: LW_FILE,LW_INPUT.

    ENDLOOP.
  ENDIF.

ENDFORM.

*&---------------------------------------------------------------------*
*&      Form  F_PROCESSA_DADOS
*&---------------------------------------------------------------------*
*       Processa dados
*----------------------------------------------------------------------*
FORM F_PROCESSA_DADOS .

  DATA: LW_INPUT TYPE TY_INPUT,
        LW_ALV   TYPE TY_ALV.

  DATA: LV_BOLA1(3) TYPE C,
        LV_BOLA2(3) TYPE C.

  CLEAR: V_COR.

  LOOP AT T_INPUT INTO LW_INPUT.
    CLEAR: LV_BOLA1,LV_BOLA2.
    LV_BOLA1 = LW_INPUT-POSX1 + LW_INPUT-PROV1.
    LV_BOLA2 = LW_INPUT-POSX2 + LW_INPUT-PROV2.

    IF LV_BOLA1 = LV_BOLA2.
      LW_ALV-POSQ = LV_BOLA1.
      LW_ALV-RESP = 'SIM É POSSÍVEL'.

    ELSE.
      LW_ALV-RESP = 'NÃO É POSSÍVEL'.
      V_COR = 'X'.

    ENDIF.

    APPEND LW_ALV TO T_ALV.
    CLEAR: LW_INPUT,LW_ALV.

  ENDLOOP.

ENDFORM.

*&---------------------------------------------------------------------*
*&      Form  F_GERA_ALV
*&---------------------------------------------------------------------*
*       Gera ALV
*----------------------------------------------------------------------*
FORM F_GERA_ALV .

  DATA: LR_TABLE     TYPE REF TO CL_SALV_TABLE,
        LR_FUNCTIONS TYPE REF TO CL_SALV_FUNCTIONS,
        LR_COLUMNS   TYPE REF TO CL_SALV_COLUMNS_TABLE,
        LR_COLUMN    TYPE REF TO CL_SALV_COLUMN_TABLE.

  DATA: LS_COLOR TYPE LVC_S_COLO.
  DATA: LC_TRUE  TYPE SAP_BOOL VALUE 'X'.

  CALL METHOD CL_SALV_TABLE=>FACTORY
    IMPORTING
      R_SALV_TABLE = LR_TABLE
    CHANGING
      T_TABLE      = T_ALV.

  IF V_COR IS NOT INITIAL.
    LR_COLUMNS = LR_TABLE->GET_COLUMNS( ).
    LR_COLUMNS->SET_OPTIMIZE( LC_TRUE ).
    LR_COLUMN ?= LR_COLUMNS->GET_COLUMN( 'RESP' ).

    LS_COLOR-COL = COL_NEGATIVE.
    LS_COLOR-INT = 0.
    LS_COLOR-INV = 0.

    LR_COLUMN->SET_COLOR( LS_COLOR ).
    LR_COLUMNS->SET_COLOR_COLUMN( 'COLOR' ).

  ENDIF.

  CALL METHOD LR_TABLE->DISPLAY.

ENDFORM.
