# -
用emu8086实现的一个打字练习

这是汇编语言程序设计实验的课程设计，
即用汇编语言来设计一个打字练习，
有计时、正确率显示等功能以及中途退出选项

;This is a typing exercise programme.
data segment
    h_welcome   db 'Welcome here my friend, '
                db 'this is a typing exercise programme.', 0dh, 0ah
                db 'If you happen to be here, press ESC to quit.', 0dh, 0ah
                db 'If not, press any other key to start then.', 0dh, 0ah, '$'
    
    h_continue  db 'Press any key to try again but ESC to quit.', '$' 
                
    h_bye       db 'Bye! Hope you are having a great day!', 0dh, 0ah,'$'
    
    h_3 db  '********', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '      **', 0ah, 8 dup(08h)
        db  '      **', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '      **', 0ah, 8 dup(08h)
        db  '      **', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)     
        db  '********', 0ah, 8 dup(08h), '$'
    
    h_2 db  '********', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '      **', 0ah, 8 dup(08h)
        db  '      **', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '**      ', 0ah, 8 dup(08h)
        db  '**      ', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)     
        db  '********', 0ah, 8 dup(08h), '$'
        
    h_1 db  '   **   ', 0ah, 8 dup(08h)
        db  '  ***   ', 0ah, 8 dup(08h)
        db  ' ****   ', 0ah, 8 dup(08h)
        db  '   **   ', 0ah, 8 dup(08h)
        db  '   **   ', 0ah, 8 dup(08h)
        db  '   **   ', 0ah, 8 dup(08h)
        db  '   **   ', 0ah, 8 dup(08h)
        db  '   **   ', 0ah, 8 dup(08h)
        db  ' ****** ', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h), '$'
        
    h_0 db  '********', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)
        db  '**    **', 0ah, 8 dup(08h)
        db  '**    **', 0ah, 8 dup(08h)
        db  '**    **', 0ah, 8 dup(08h)
        db  '**    **', 0ah, 8 dup(08h)
        db  '**    **', 0ah, 8 dup(08h)
        db  '**    **', 0ah, 8 dup(08h)
        db  '********', 0ah, 8 dup(08h)     
        db  '********', 0ah, 8 dup(08h), '$' 
    
    les_miserables  db 0 dup(0)
        db  'Les Miserables, colloquially known as Les Mis or Les Miz, is a musical that was '
        db 80 dup(0)
        db  'composed in 1980 by the French composer Claude-Michel Schonberg with a libretto '
        db 80 dup(0)
        db  'by Alain Boublil, and lyrics by Herbert Kretzmer. Sung through, it is one of the'
        db 80 dup(0)
        db  ' most well-known and performed musicals worldwide. On October 8 2006, the show c'
        db 80 dup(0)
        db  'elebrated its 21st anniversary on London s West End and became the longest-runni'
        db 80 dup(0)
        db  'ng West End musical in history, reaching 9,500 performances. The production cont'
        db 80 dup(0)
        db  'inues to this day at Londons queens Theatre. The show is now in its 25th year.'
        db 80 dup(0), '$'
        ;db  'Based on Victor Hugo s 1862 novel of the same name, it is set in early 19th-cent'
        ;db 80 dup(0)
        ;db  'ury France and follows the intertwining stories of a cast of characters as they '
        ;db 80 dup(0)
        ;db  'struggle for redemption and revolution. The main characters are joined by an ens'
        ;db 80 dup(0)
        ;db  'emble that includes prostitutes, student revolutionaries, factory workers...', '$'

    time_begin  dw 0, 0
    time_end    dw 0, 0
    time_during dw 0, 0
    time_std    db 0, 0, 0 
    
    scale_tick  db 20
    scale_time  db 60
    
    h_time  db 'Time you have used is: '
    
    time    db 0, 0, ':', 0, 0, '.', 0, 0, 0dh, 0ah, '$'    
    
    char_typed      dw 0
    char_incorrect  dw 0
    char_correct    dw 0
    
    ratio_weight    db 100, 10, 1
    
    h_score db 'Score: '
    ratio   db 0, 0, 0, '/', 0, 0, 0, 0dh, 0ah, '$'
    
    h_quit_confirm  db 'No time and score left if quit, '
                    db 'press ESC to confirm, others to continue.', '$'

ends

stack segment
    dw   256  dup(0)
ends

code segment
start:
; set segment registers:
    mov ax, data
    mov ds, ax
    mov es, ax
;-----------------------------------------------------------------------------
scroll_m  macro   n, ulr, ulc, lrr, lrc, att
    push ax
    push bx
    push cx
    push dx
    
    mov ah, 6
    mov al, n
    mov ch, ulr
    mov cl, ulc
    mov dh, lrr
    mov dl, lrc
    mov bh, att
    int 10h
    
    pop dx
    pop cx
    pop bx
    pop ax              
endm
;-----------------------------------------------------------------------------
curse_m   macro   cury, curx
    push ax
    push bx
    push cx
    push dx
    
    mov ah, 2
    mov dh, cury
    mov dl, curx
    mov bh, 0
    int 10h
    
    pop dx
    pop cx
    pop bx
    pop ax  
endm
;-----------------------------------------------------------------------------
welcome_m macro
    push ax
    push dx
     
    mov ah, 09h
    mov dx, offset h_welcome
    int 21h
    
    pop dx
    pop ax
endm
;-----------------------------------------------------------------------------
get_key_m macro
    mov ah, 01h
    int 21h
endm
;-----------------------------------------------------------------------------
check_quit_m  macro 
    push ax
    
    cmp al, 1bh
    je quit
    jmp go
    
    pop ax
endm
;-----------------------------------------------------------------------------
goodbye_m macro
    scroll_m   0,    17,  0,  24, 79,   80h
    curse_m 18, 0
    mov ah, 09h
    mov dx, offset h_bye
    int 21h
endm
;-----------------------------------------------------------------------------
crlf_m    macro
    push ax
    push dx
    
    mov ah, 02h
    mov dl, 0dh
    int 21h
    mov dl, 0ah
    int 21h
    
    pop dx
    pop ax
endm
;-----------------------------------------------------------------------------
count_down_m    macro
    push ax
    push dx
    push cx
    
    curse_m 7, 34
    mov ah, 09h
    mov dx, offset h_3
    int 21h
    
    mov ah, 86h
    mov cx, 000ah
    mov dx, 0000h
    int 15h
     
    curse_m 7, 34
    mov ah, 09h
    mov dx, offset h_2
    int 21h
    
    mov ah, 86h
    mov cx, 000ah
    mov dx, 0000h
    int 15h
    
    curse_m 7, 34
    mov ah, 09h
    mov dx, offset h_1
    int 21h
    
    mov ah, 86h
    mov cx, 000ah
    mov dx, 0000h
    int 15h
          
    curse_m 7, 34
    mov ah, 09h
    mov dx, offset h_0
    int 21h 
    
    mov ah, 86h
    mov cx, 000ah
    mov dx, 0000h
    int 15h
    
    curse_m 0, 0
    
    pop cx
    pop dx
    pop ax
endm
;-----------------------------------------------------------------------------
continue_m  macro
    push ax
    push dx
    
    scroll_m   0,  17,  0,  24, 79,   80h
    curse_m 18, 0
    mov ah, 09h
    mov dx, offset h_continue
    int 21h
    
    pop dx
    pop ax
endm
;-----------------------------------------------------------------------------
exercise_m  macro
    mov bl, 1                   ; cursor index(second line)
    mov si, -80                 ; first round index = -80 + 80 = 0 
new_line:                       ;
    curse_m bl, 0               ; set cursor to the second line
    mov cx, 80                  ; loop 80 char per line
    add si, 80                  ; skip whole line
    mov bh, 0                   ; track column index
next_char:                      ;    
    mov ah, 00h
    int 16h                     ; get char into al
    
    cmp al, 1bh
    je stop                     ; pressed ESC
    inc char_typed              ; typed++

    
    cmp al, les_miserables[si]  ; check char correct?
    je correct                  
    scroll_m 0,   bl,  bh,  bl,  bh,  7ch       ; set next single char form
    inc char_incorrect          ; incorrect++
    
correct:                        ;    
    mov ah, 0eh
    int 10h                     ; show char (correct or incorrect)
    
    inc si                      ; line index++
    inc bh                      ; column index++
    
    cmp si, 040eh               ; 040eh, all char and skiped line  
    je text_end                 ; 040eh, also the end of the text
    
    loop next_char              ; not till 80 char, then next char
    add bl, 2                   ; row += 2 (line3, 5, 7, 9...)
    jmp new_line
endm
;-----------------------------------------------------------------------------
set_text    macro
    mov ah, 09h
    mov dx, offset les_miserables
    int 21h
endm
;-----------------------------------------------------------------------------
set_exercise_background macro
    mov bl, 0
next_set_BG:                                ;
    scroll_m 0,   bl,  0,  bl,  79,    8fh  ; white in gray++
    add bl, 2
    cmp bl, 14
    jne next_set_BG
endm   
;-----------------------------------------------------------------------------
time_run macro
    mov ah, 00h
    int 1ah
    mov time_begin[0], cx
    mov time_begin[2], dx       ; get begin tick
endm
;-----------------------------------------------------------------------------
time_stop   macro
    push ax
    push cx
    push dx
    
    mov ah, 00h
    int 1ah
    mov time_end[0], cx
    mov time_end[2], dx         ; get end tick
    
    sub cx, time_begin[0]
    mov time_during[0], cx
    sub dx, time_begin[2]
    mov time_during[2], dx      ; get during tick
    
    mov ax, time_during[2]
    div scale_tick              ; change tick to time
    
    mov time_std[2], ah         ; get msec 
    
    and ah, 0                   ; ready another div
    div scale_time
    mov time_std[1], ah         ; get sec
    
    mov time_std[0], al         ; get min
    
    pop dx
    pop cx
    pop bx
endm
;-----------------------------------------------------------------------------
show_time   macro
    push ax
    push bx
    push dx
    
    mov al, time_std[0]
    and ah, 0
    mov bl, 10
    div bl
    mov time[0], al         ; min 1
    add time[0], 30h
    mov time[1], ah         ; min 10
    add time[1], 30h
    
    mov al, time_std[1]
    and ah, 0
    mov bl, 10
    div bl
    mov time[3], al         ; sec 1
    add time[3], 30h
    mov time[4], ah         ; sec 10
    add time[4], 30h
    
    mov al, time_std[2]
    and ah, 0
    mov bl, 10
    div bl
    mov time[6], al         ; msec 1
    add time[6], 30h
    mov time[7], ah         ; msec 10
    add time[7], 30h
    
    scroll_m 0,   14, 0,  16, 79,    3ah    ; green in lazuli
    curse_m 15, 0
    
    mov ah, 09h
    mov dx, offset h_time
    int 21h
    
    pop dx
    pop bx
    pop ax
endm
;-----------------------------------------------------------------------------
correct_rate    macro
    push ax
    push dx
    
    mov ax, char_typed
    sub ax, char_incorrect
    mov char_correct, ax        ; get correct
    
    
    mov ax, char_correct
    div ratio_weight[0]
    mov ratio[0], al
    add ratio[0], 30h
    
    mov al, ah
    and ah, 0
    div ratio_weight[1]
    mov ratio[1], al
    add ratio[1], 30h
    
    mov al, ah
    and ah, 0
    div ratio_weight[2]
    mov ratio[2], al
    add ratio[2], 30h
    
    mov ax, char_typed
    div ratio_weight[0]
    mov ratio[4], al
    add ratio[4], 30h
    
    mov al, ah
    and ah, 0
    div ratio_weight[1]
    mov ratio[5], al
    add ratio[5], 30h
    
    mov al, ah
    and ah, 0
    div ratio_weight[2]
    mov ratio[6], al
    add ratio[6], 30h
    
    curse_m 15, 39
    mov ah, 09h
    mov dx, offset h_score
    int 21h
    
    pop dx
    pop ax
endm
;-----------------------------------------------------------------------------
clear_all_m     macro
    mov time_begin[0], 0
    mov time_begin[2], 0
    mov time_end[0], 0
    mov time_end[2], 0
    mov time_during[0], 0
    mov time_during[2], 0
    mov time_std[0], 0
    mov time_std[1], 0
    mov time_std[2], 0
    
    mov char_typed, 0
    mov char_incorrect, 0
    mov char_correct, 0
endm
;-----------------------------------------------------------------------------
store_cursor    macro
    push ax
    push bx
    push dx
    
    scroll_m 0, 14, 0, 16, 79, 6eh
    curse_m 15, 0
    mov ah, 09h
    mov dx, offset h_quit_confirm
    int 21h
    get_key_m
    cmp al, 1bh
    je quit
    
    pop dx
    pop bx
    pop ax
    scroll_m 0, 14, 0, 16, 79, 71h
    curse_m bl, bh
    jmp next_char
endm
;-----------------------------------------------------------------------------

;=============================================================================
    
    scroll_m    0, 0, 0, 24, 79, 71h        ; set BG gray, word blue
    welcome_m
    get_key_m
    check_quit_m     

go:                  ;
    clear_all_m
    scroll_m    0, 0, 0, 24, 79, 7bh        ; lazuli in gray
    count_down_m
    scroll_m    0, 0, 0, 24, 79, 71h        ; blue in gray
    set_exercise_background
    set_text
    time_run 
    exercise_m
    jmp text_end
stop:                   ;
    store_cursor        
text_end:               ;    
    time_stop
    show_time
    correct_rate                                                          
    continue_m
    get_key_m
    check_quit_m

quit:                   ;
    goodbye_m    
    
    mov ax, 4c00h ; exit to operating system.
    int 21h    
ends

end start ; set entry point and stop the assembler.
