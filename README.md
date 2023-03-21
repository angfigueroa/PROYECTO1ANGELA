; Archivo: Proyecto.s
; Dispositivo: PIC16F887
; Autor: Angela Figueroa
; Compilador: pic-a (v2.30), MPLABX V5.40
;
; Programa: GENERADOR DE FRECUENCIAS 
; Hardware: LEDs, push buttons, displays
;
; Creado: 28 de febrero, 2023
; Ultima modificacion: 19 de marzo, 2023
PROCESSOR 16F887
#include <xc.inc>
;CONFIGURACIÓN 1 
    CONFIG FOSC=INTRC_NOCLKOUT //OSCILADOR INTERNO SIN SALIDAS
    CONFIG WDTE=OFF // WDT DISABLE (REINICIO REPETITIVO DE PIC)
    CONFIG PWRTE=OFF // PWRT ENABLE (ESPERA DE 72MS AL INICIAR)
    CONFIG MCLRE=ON //EL PIN DE MCLR SE USA COMO I/O
    CONFIG CP=OFF //SIN PROTECCIÓN DE CÓDIGO
    CONFIG CPD=OFF //SIN PROTECCIÓN DE DATOS    
    CONFIG BOREN=OFF //SIN REINICIO CUANDO EL V DE ALIMENTACIÓN BAJA A 4V
    CONFIG IESO=OFF // REINICIO SIN CAMBIO DE RELOJ INTERNO A EXTERNO
    CONFIG FCMEN=OFF // CAMBBIO DE RELOJ EXTERNO A INTERNO EN CASO DE FALLO
    CONFIG LVP=OFF //PROGRAMACIÓN EN BAJO VOLTAJE PERMITIDA
    
 ;CONFIGURACIÓN 2
    CONFIG WRT=OFF //PROTECCIÓN DE AUTOESCRITURA POR EL PROGRAMA DESACTIVADA
    CONFIG BOR4V=BOR40V //REINICIO ABAJO DE 4, (BOR21V=2.1V)
    PSECT udata_bank0 ;common memory
    ;----------MEMORIA DE LAS VARIABLES----------------------------------------
	w_tem: DS 1
	status_tem: DS 1
	contador: DS 1
    ;-----PARA LOS DISPLAYS----------------------------------------------------
	val_display10: DS 1
	val_display32: DS 1
	banderas: DS 1
	display0: DS 1
	display1: DS 1
	display2: DS 1
	display3: DS 1
	cont_tmr2: DS 1
    ;-----PARA DIGITOS DE LOS DISPLAYS-------------------------------------------
	unidades: DS 1
	decenas: DS 1
	centenas: DS 1
	miles: DS 1
	valor_tmr2: DS 1
    ;----------HZ------------------------------------------------------------
	banderas_ondas: DS 1
	indicadoreshz: DS 1
    ;-----SUBIR o BAJAR-------------------------------------------------------
    bsubir EQU 0
    bbajar EQU 1
   
	;cont_big: DS 1

    PSECT resVect, class=CODE, abs, delta=2 
    ;--------------RESET-------------------------
    ORG 00h	
    resetVec:
	PAGESEL main
	goto main
    ;------------Rutina de interupción-------------------
    PSECT code, delta=2, abs
    ORG 004h	;Poscición 
    
    push:
	movwf w_tem
	swapf STATUS,0
	movwf status_tem
    isr:
	banksel INTCON
	banksel PIR1
	banksel PORTB
	btfss PORTB,2
	call cambiar_ondas
	btfsc PIR1, 1
	call generar_ondas
	btfsc INTCON,2
	call display_var
	
    pop1:
	   swapf    status_tem,0
	   movwf    STATUS
	   swapf    w_tem,1
	   swapf    w_tem,0
	   retfie
    ;----NOS PERMITE CAMBIAR ONDAS REVISANDO LAS BANDERAS--------- 
    cambiar_ondas:
	btfss PORTB,2
	goto $-1
	btfsc banderas_ondas,2
	goto $+3
	bsf   banderas_ondas,2
	goto $+2
	bcf   banderas_ondas,2
	return
;----NOS PERMITE GENERAR LA ONDA CUADRADA A TRIANGULAR O DE TRI A CUA--------- 	
    generar_ondas:
	btfss banderas_ondas,2
	goto $+3
	call triangular
	return
	call cuadrada
	return	
  ;----OPERACION PARA LA CUADRADA--------	
    cuadrada:
	banksel TMR2
	clrf TMR2
	bcf  PIR1, 1
	banksel PORTA
	btfsc banderas_ondas,1
	goto apagar_cuadrada
	movlw 255
	movwf PORTA
	bsf banderas_ondas,1
	return
;.-----LA APAGAMOS----------	
	apagar_cuadrada:
	clrf PORTA
	bcf banderas_ondas,1
	return
;----OPERACION PARA LA TRIANGULAR--------	
    triangular:
	banksel TMR2
	clrf TMR2
	;bcf  PIR1, 1
	banksel PORTA
	btfsc banderas_ondas,0
	goto decremenrtar_triangular
	incf PORTA
	movf PORTA,0
	sublw 255
	btfss STATUS, 2
	return
	bsf banderas_ondas,0
	return
;----BAJAMOS LA TRIANGULAR--------
	decremenrtar_triangular:
	decfsz PORTA,1
	return
	bcf banderas_ondas,0
	return



    ;------------Main----------------
     PSECT code, delta=2, abs
    ORG 100h	;Poscición 
    ;-----------Tabla para display 7 segmentos, anodo común----------------
    ;----TABLA PARA NUESTRO VALOR DEL PR2--------
    tabla_PR2:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 250;50hz
	retlw 125;100hz
	retlw 100;125hz
	retlw 50;250hz
	retlw 25;500hz
	retlw 20;625hz
	retlw 50;1000hz
	retlw 40;1250hz
	retlw 25;2000hz
	retlw 20;2500hz
	retlw 16;3125hz
	retlw 10;5000hz
	retlw 8;6250hz
	retlw 25;8000hz
;----TABLA PARA LA CONFIGURACION DEL TMR2--------
     tabla_config_tmr2:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 01001111B
	retlw 01001111B
	retlw 01001111B
	retlw 01001111B
	retlw 01001111B
	retlw 01001111B
	retlw 01001101B
	retlw 01001101B
	retlw 01001101B
	retlw 01001101B
	retlw 01001101B
	retlw 01001101B
	retlw 01001101B
	retlw 01001100B
	
;----TABLA PARA MOSTAR LOS VALORES EN EL DISPLAY--------	
   valores_display10:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 50
	retlw 00
	retlw 25
	retlw 50
	retlw 00
	retlw 25
	retlw 00
	retlw 50
	retlw 00
	retlw 00
	retlw 25
	retlw 00
	retlw 50
	retlw 00
;----TABLA PARA MOSTAR LOS VALORES EN EL DISPLAY PARTE 2--------	
    valores_display32:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 00
	retlw 01
	retlw 01
	retlw 02
	retlw 05
	retlw 06
	retlw 10
	retlw 12
	retlw 20
	retlw 25
	retlw 31
	retlw 50
	retlw 62
	retlw 80
;----TABLA ORIGINAL DEL DISPLAY--------
    TABLA_DISPLAY:
	clrf PCLATH
	bsf  PCLATH,0
	andlw 0X0F
	addwf PCL
	retlw 01000000B;0
	retlw 11111001B;1
	retlw 00100100B;2
	retlw 00110000B;3
	retlw 00011001B;4
	retlw 00010010B;5
	retlw 00000010B;6
	retlw 01111000B;7
	retlw 00000000B;8
	retlw 00010000B;9
    main:
	call pines
	call ciocb
	call tmr0_config
	call tmr2_config
	call clock
	
    loop:
	call contador_frecuencia 
	call preparar_display
	call verificar_hz
	call obtener_miles
	call obtener_centenas
	call obtener_decenas
	call obtener_unidades
	call displays
	goto loop
;----SE VERIFICA CUANDO ESTAMOS EN HZ O EN KHZ--------	
    verificar_hz:
	
	movlw 10
	subwf val_display32,0
	btfsc STATUS,0
	goto $+3
	goto hz
	goto $+2
	goto khz
	return
	hz:;----SE ENCONTRARAN EN EL PORTD POR MEDIO DE LEDS--------
	    banksel PORTD
	    bsf PORTD,4
	    bcf PORTD,5
	    return
	khz:
	    banksel PORTD
	    bcf PORTD,4
	    bsf	PORTD,5
	    return
;----PREPARAMOS EL PR2--------	    
    preparar_pr2:
        movf contador,0
	call tabla_PR2
	call reset_tmr2
	return
;----PREPARAMOS LA CONFIGURACION DEL TMR2--------	
    preparar_config_tmr2:
        movf contador,0
	call tabla_config_tmr2
	banksel T2CON
	movwf T2CON
	return
;----PREPARAMOS EL DISPLAY--------	
    preparar_display:
	banksel PORTA
	clrf val_display10
	movf contador,0
	call valores_display10
	movwf val_display10
	
	clrf val_display32
	movf contador,0
	call valores_display32
	movwf val_display32
	return

;----PARA INCREMENTAR Y DECREMENTAR CREAMOS UN CONTADOR DE FRECUENCIAS--------	
    contador_frecuencia:
	banksel PORTB
	btfss PORTB,0
	call incrementar
	btfss PORTB, 1
	call decrementar
	return
	;----OPERACION PARA INCREMENTAR--------
	incrementar:
	    btfss PORTB,0
	    goto $-1
	    incf contador
	    call preparar_pr2
	    goto preparar_config_tmr2
	return
	;----OPERACION PARA DECREMENTAR--------
	decrementar:
	    btfss PORTB,1
	    goto $-1
	    decf contador
	    call preparar_pr2
	    goto preparar_config_tmr2
	return
;----RESETEAMOS EL TMR2--------	
    reset_tmr2:
	BANKSEL PR2
	clrf PR2
	movwf	PR2
	return
;------PARA COLOCAR LOS PUSHBUTTONS--------	
    pines:
	BANKSEL PORTD
	clrf	PORTD
	clrf	PORTA
	clrf	PORTC
	clrf	PORTB
	clrf	PORTE
	banksel TRISD
	bsf	TRISB,bsubir
	bsf	TRISB,bbajar
	clrf	TRISD
	clrf	TRISA
	clrf	TRISC
	clrf	TRISE
	banksel ANSEL
	clrf ANSEL
	clrf ANSELH
	
	;--------Confuración de PULL UP del PORTB------------------
	banksel	TRISA
	bcf	OPTION_REG,7 ;RBPU
	bsf	WPUB,bsubir
	bsf	WPUB,bbajar
	;----------------------------------------------------------
	
	
	return
	
     ciocb:
	banksel TRISA
	bsf	IOCB,bsubir
	bsf	IOCB,bbajar
	BSF	PIE1, 1
	banksel PORTA
	bsf	INTCON, 7
	bsf	INTCON, 6
    return
  ;------EL RELOJ--------  
    clock:
	banksel OSCCON;Oscillator Control Register 
	bsf IRCF2	 
	bsf IRCF1	  
	bsf IRCF0	  
	bsf SCS		  
	return
	
;----PARA CONFIGURAR EL TMR0------------------------------------	
     tmr0_config:
	banksel TRISA
	Movlw 01010111B	    ;Configuración
	movwf OPTION_REG    ;Cargar la configuración
	banksel PORTA
	banksel TMR0
	movlw	250;
	movwf	TMR0
	bcf	INTCON,2
	return
;----PARA CONFIGURAR EL TMR2------------------------------------	
    tmr2_config:
	banksel T2CON
	movlw 01001111B
	movwf T2CON
	banksel TRISA
	movlw	250
	movwf	PR2
	banksel	PORTA
	clrf	TMR2
	bcf	PIR1,1
	return
	
	
;-------OPERACIONES DEL POSTLAB 5------------------ 	
    obtener_miles:;-------MILES------------------ 
	clrf miles
	movlw 10
	subwf val_display32, 1
	incf miles
	btfsc STATUS, 0
	goto $-3
	decf miles
	movlw 10
	addwf val_display32, 1
    return
     obtener_centenas:;-------CENTENAS------------------ 
	clrf centenas
	movf val_display32,0
	movwf centenas
    return
    
    obtener_decenas:;-------DECENAS------------------ 
	clrf decenas
	movlw 10
	subwf val_display10, 1
	incf decenas
	btfsc STATUS, 0
	goto $-3
	decf decenas
	movlw 10
	addwf val_display10, 1
    return
    
    obtener_unidades:;-------UNIDADES------------------ 
	clrf unidades
	movf val_display10,0
	movwf unidades
	
    return
    
     ;-------DISPLAYS(4)-------------------------------------------  
    displays:
	 movf	miles, W
	call	TABLA_DISPLAY;-------LLAMAMOS A LA TABLA DE LOS DISPLAYS------- 
	movwf	display0
	
	movf	centenas, W
	call	TABLA_DISPLAY
	movwf	display1
	
	movf	decenas, W
	call	TABLA_DISPLAY
	movwf	display2
	
	movf	unidades, W
	call	TABLA_DISPLAY
	movwf	display3
	
	
	return
	
     display_var:
	clrf	PORTD
	btfsc	banderas, 0
	goto    display_centenas
	btfsc	banderas, 1
	goto    display_decenas
	btfsc	banderas, 2
	goto    display_unidades
;-------INIDICAMOS DONDE QUEREMOS COLOCAR LOS DISPLAYS------------------------ 	
    ;-------MILES-------------------------------------------  
    display_miles:
	movf	display0, 0
	movwf	PORTC
	BSF	PORTD, 0
	goto	toggle
    ;-------CENTENAS-------------------------------------------  	
    display_centenas:
	movf	display1, 0
	movwf	PORTC
	BSF	PORTD, 1
	goto	toggle
	;-------DECENAS-------------------------------------------  
    display_decenas:
	movf	display2, 0
	movwf	PORTC
	BSF	PORTD, 2
	goto	toggle
	;-------UNIDADES-------------------------------------------  
    display_unidades:
	movf	display3, 0
	movwf	PORTC
	BSF	PORTD, 3
;---REVISAMOS LAS BANDERAS------	
    toggle:
	incf	banderas
	return
END
