; BINARIO → DECIMAL → LCD
; Troy's Breadboard Computer

NUMBER = 135        ; número a mostrar
ZERO   = 48         ; ASCII '0'

DISPLAY_MODE = LCD_CMD_DISPLAY | LCD_CMD_DISPLAY_ON

; --- Inicializar LCD ---
lcc #LCD_INITIALIZE
lcc #DISPLAY_MODE
lcc #LCD_CMD_CLEAR

main:
    data SP, 255       ; inicializa stack
    data Rb, NUMBER    ; número a convertir
    call bin2dec       ; convierte y muestra
    hlt                ; detener ejecución


; ======================================================
; bin2dec
; Convierte el número en Rb a decimal ASCII y lo muestra
; ======================================================
bin2dec:
    tst Rb
    jz .printZero      ; si el número = 0, mostrar "0"

    data Rc, 0         ; contador de dígitos = 0

.loopDiv:
    mov Ra, Rb         ; dividendo
    data Rb, 10        ; divisor
    call div8          ; divide → Rc = cociente, Ra = resto
    push Ra            ; guardar resto en la pila
    inc Rc             ; contador de dígitos++
    mov Rb, Rc         ; pasa cociente a Rb
    tst Rb
    jnz .loopDiv       ; repetir mientras cociente ≠ 0

.printDigits:
    pop Ra             ; saca último resto
    data Rb, ZERO
    add Ra             ; convierte a ASCII
    lcd Ra             ; lo envía a LCD
    dec Rc             ; contador--
    jnz .printDigits   ; si aún quedan dígitos, repetir
    ret

.printZero:
    data Rd, ZERO
    lcd Rd
    ret


; ======================================================
; div8
; Divide Ra ÷ Rb
; Entrada: Ra = dividendo, Rb = divisor
; Salida : Rc = cociente, Ra = resto
; ======================================================
div8:
    data Rc, 0x00

.step:
    cmp Rb, Ra
    jz .add
    jc .return

.add:
    inc Rc
    sub Ra
    jnz .step

.return:
    ret
