.MODEL SMALL
 
.STACK 100H

.DATA
    parking     DW 10 DUP(0)
    start_hours DB 10 DUP(0)
    
    menu        DB 13,10,'===== PARKING LOT MENU =====',13,10
                DB '1. Vehicle Entry',13,10
                DB '2. Slot Status Display',13,10
                DB '3. Exit Program',13,10
                DB 'Choose Option (1-3): $'
    
    prompt_veh  DB 13,10,'Enter vehicle number (max 3 digits): $'
    prompt_hour DB 13,10,'Enter start hour (0-23): $'
    prompt_slot DB 13,10,'Enter slot number (0-9): $'
    
    msg_invalid_veh DB 13,10,'Invalid vehicle number! Must be numeric and max 3 digits.$'
    msg_invalid_hour DB 13,10,'Invalid hour! Must be between 0-23.$'
    msg_invalid_slot_input DB 13,10,'Invalid slot number! Must be a single digit 0-9.$'
    msg_already_parked DB 13,10,'This vehicle is already parked in another slot!$'
    msg_invalid_slot DB 13,10,'Invalid slot number!$'
    msg_slot_occupied DB 13,10,'Slot already occupied!$'
    msg_success DB 13,10,'Vehicle parked successfully in slot $'
    msg_dot     DB '.$'
    
    slot_header DB 13,10,'Slot Status:',13,10,'$'
    zero_char   DB '0 $'
    one_char    DB '1 $'
    newline     DB 13,10,'$'
    
    msg_exit    DB 13,10,'Program Ended.$'
    msg_invalid_choice DB 13,10,'Invalid choice.$'
    
    buffer      DB 6 DUP('$')
    temp_num    DW 0
    vehicle_no  DW 0
    slot_no     DB 0
    start_hour  DB 0
    
.CODE
MAIN PROC
    MOV AX,@DATA
    MOV DS,AX
 
main_loop:
    LEA DX, menu
    MOV AH, 09H
    INT 21H
    
    MOV AH, 01H
    INT 21H
    
    CMP AL, '1'
    JE choice1
    
    CMP AL, '2'
    JE choice2
    
    CMP AL, '3'
    JE choice3
    
    LEA DX, msg_invalid_choice
    MOV AH, 09H
    INT 21H
    JMP main_loop

choice1:
    CALL vehicle_entry
    JMP main_loop

choice2:
    CALL slot_status
    JMP main_loop

choice3:
    LEA DX, msg_exit
    MOV AH, 09H
    INT 21H
    JMP exit_program

exit_program:
    MOV AX,4C00H
    INT 21H

MAIN ENDP

vehicle_entry PROC
    LEA DX, prompt_veh
    MOV AH, 09H
    INT 21H
    
    MOV CX, 3
    MOV SI, offset buffer
    MOV vehicle_no, 0
    
read_veh_loop:
    MOV AH, 01H
    INT 21H
    CMP AL, 13
    JE veh_read_done
    
    CMP AL, '0'
    JB invalid_vehicle
    CMP AL, '9'
    JA invalid_vehicle
    
    MOV [SI], AL
    INC SI
    
    SUB AL, '0'
    MOV AH, 0
    MOV BX, vehicle_no
    MOV DX, 10
    MOV AX, BX
    MUL DX
    MOV BX, AX
    MOV AL, [SI-1]
    SUB AL, '0'
    MOV AH, 0
    ADD BX, AX
    MOV vehicle_no, BX
    
    LOOP read_veh_loop
    
veh_read_done:
    CMP vehicle_no, 0
    JE invalid_vehicle
    
    MOV CX, 10
    MOV SI, 0
check_existing:
    MOV AX, parking[SI]
    CMP AX, vehicle_no
    JE already_parked
    ADD SI, 2
    LOOP check_existing
    
    LEA DX, prompt_hour
    MOV AH, 09H
    INT 21H
    
    CALL read_two_digit_number
    
    CMP AX, 0
    JL invalid_hour_input
    CMP AX, 23
    JG invalid_hour_input
    
    MOV start_hour, AL
    
    LEA DX, prompt_slot
    MOV AH, 09H
    INT 21H
    
    CALL read_one_digit_number
    
    CMP AX, 0
    JL invalid_slot_input
    CMP AX, 9
    JG invalid_slot_input
    
    MOV slot_no, AL
    
    MOV BL, slot_no
    MOV BH, 0
    MOV SI, BX
    SHL SI, 1
    MOV AX, parking[SI]
    CMP AX, 0
    JNE slot_occupied
    
    MOV AX, vehicle_no
    MOV parking[SI], AX
    
    MOV SI, BX
    MOV AL, start_hour
    MOV start_hours[SI], AL
    
    LEA DX, msg_success
    MOV AH, 09H
    INT 21H
    
    MOV DL, slot_no
    ADD DL, '0'
    MOV AH, 02H
    INT 21H
    
    LEA DX, msg_dot
    MOV AH, 09H
    INT 21H
    
    RET
    
invalid_vehicle:
    LEA DX, msg_invalid_veh
    MOV AH, 09H
    INT 21H
    RET
    
already_parked:
    LEA DX, msg_already_parked
    MOV AH, 09H
    INT 21H
    RET
    
invalid_hour_input:
    LEA DX, msg_invalid_hour
    MOV AH, 09H
    INT 21H
    RET
    
invalid_slot_input:
    LEA DX, msg_invalid_slot_input
    MOV AH, 09H
    INT 21H
    RET
    
slot_occupied:
    LEA DX, msg_slot_occupied
    MOV AH, 09H
    INT 21H
    RET

vehicle_entry ENDP

read_number PROC
    PUSH BX
    PUSH CX
    PUSH DX
    
    MOV CX, 0
    MOV BX, 0
    
read_digit:
    MOV AH, 01H
    INT 21H
    CMP AL, 13
    JE done_reading
    
    CMP AL, '0'
    JB done_reading
    CMP AL, '9'
    JA done_reading
    
    SUB AL, '0'
    MOV AH, 0
    
    PUSH AX
    MOV AX, BX
    MOV DX, 10
    MUL DX
    MOV BX, AX
    POP AX
    ADD BX, AX
    
    INC CX
    CMP CX, 3
    JB read_digit
    
done_reading:
    MOV AX, BX
    POP DX
    POP CX
    POP BX
    RET
read_number ENDP

read_two_digit_number PROC
    PUSH BX
    PUSH CX
    PUSH DX
    
    MOV CX, 0
    MOV BX, 0
    
read_hour_digit:
    MOV AH, 01H
    INT 21H
    CMP AL, 13
    JE hour_done_reading
    
    CMP AL, '0'
    JB hour_done_reading
    CMP AL, '9'
    JA hour_done_reading
    
    SUB AL, '0'
    MOV AH, 0
    
    PUSH AX
    MOV AX, BX
    MOV DX, 10
    MUL DX
    MOV BX, AX
    POP AX
    ADD BX, AX
    
    INC CX
    CMP CX, 2
    JB read_hour_digit
    
hour_done_reading:
    MOV AX, BX
    POP DX
    POP CX
    POP BX
    RET
read_two_digit_number ENDP

read_one_digit_number PROC
    PUSH BX
    PUSH CX
    PUSH DX
    
    MOV CX, 0
    MOV BX, 0
    
read_slot_digit:
    MOV AH, 01H
    INT 21H
    CMP AL, 13
    JE slot_done_reading
    
    CMP AL, '0'
    JB slot_done_reading
    CMP AL, '9'
    JA slot_done_reading
    
    SUB AL, '0'
    MOV AH, 0
    MOV BX, AX
    
    INC CX
    CMP CX, 1
    JE slot_done_reading
    
    JMP read_slot_digit
    
slot_done_reading:
    MOV AX, BX
    POP DX
    POP CX
    POP BX
    RET
read_one_digit_number ENDP

slot_status PROC
    LEA DX, slot_header
    MOV AH, 09H
    INT 21H
    
    MOV CX, 10
    MOV SI, 0
    
slot_loop:
    MOV AX, parking[SI]
    CMP AX, 0
    JE print_zero
    
    LEA DX, one_char
    MOV AH, 09H
    INT 21H
    JMP next_slot
    
print_zero:
    LEA DX, zero_char
    MOV AH, 09H
    INT 21H
    
next_slot:
    ADD SI, 2
    LOOP slot_loop
    
    LEA DX, newline
    MOV AH, 09H
    INT 21H
    
    RET
slot_status ENDP

    END MAIN
