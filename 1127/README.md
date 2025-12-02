1. DF = 1
2. ±2 (2 증가 또는 감소)
3. 비교 방향이 모호함 (DF에 의해 결정됨)
4. 일치한 문자 이후 위치(다음 바이트)를 가리킴
5. REPNE
6. DF = 0
7. 공백이 아닐 때 루프 종료하려고 (JNE 사용)
8. 아무 변화 없음 (그대로 유지됨)
9. REPNE
10. EDI - 시작주소 - 1
11. 10
12. 문자열 연산이 앞으로 진행되게 하려고 (DF=0 필요)
13. 중복된 비교라 결과에 영향 없음 → 제거 가능
14. 조건 분기를 재구성하여 점프 없이 처리 가능
ㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡ
1. DF = 1
2. ±2

3. DF 상태에 따라 증가/감소가 달라져서

4. 일치한 문자 바로 다음 위치

5. REPNE (REPNZ)

6. DF = 0 (CLD)

7. 일치하지 않을 때 반복을 계속하기 위해 (JNE)

8. 아무 변화 없음

9. REPNE (REPNZ)

10. (EDI_final – EDI_initial) – 1

11. 10번

12. 배열을 앞에서 뒤로 채우기 위해(DF=0 필요)

13. 중복된 비교라서 없어도 결과 동일

14.분기 구조를 정리해 흐름을 자연스럽게 만들어 제거 가능
ㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡ
1.
INCLUDE Irvine32.inc

.data
msg1 BYTE "Result: ", 0
result DWORD ?

.code

main PROC

    mov eax, 5
    add eax, 7
    mov result, eax

    mov edx, OFFSET msg1
    call WriteString

    mov eax, result
    call WriteInt
    call Crlf

    exit
main ENDP

MyProc PROC
    ret
MyProc ENDP

END
2.
INCLUDE Irvine32.inc

.data
msg1 BYTE "Result: ", 0
result DWORD ?

.code

main PROC

    mov eax, 5
    add eax, 7
    mov result, eax

    mov edx, OFFSET msg1
    call WriteString

    mov eax, result
    call WriteInt
    call Crlf

    exit
main ENDP


Str_concat PROC uses esi edi src:PTR BYTE, dest:PTR BYTE
    mov edi, dest
L1:
    mov al, [edi]
    cmp al, 0
    je L2
    inc edi
    jmp L1

L2:
    mov esi, src
L3:
    mov al, [esi]
    mov [edi], al
    cmp al, 0
    je L4
    inc esi
    inc edi
    jmp L3

L4:
    ret
Str_concat ENDP

END
3.
INCLUDE Irvine32.inc

.data
target BYTE "abcxxxxdefghijklmop", 0

.code

Str_remove PROC USES esi edi ecx edx, pStart:PTR BYTE, count:DWORD

    mov edx, pStart        ; 시작 주소
    mov ecx, count         ; 제거할 문자 개수

    mov esi, edx
    add esi, ecx           ; esi = pStart + count
    mov edi, edx           ; edi = pStart

copy_loop:
    mov al, [esi]
    mov [edi], al
    inc esi
    inc edi
    cmp al, 0
    jne copy_loop

    ret
Str_remove ENDP


main PROC

    INVOKE Str_remove, ADDR target+3, 4

    mov edx, OFFSET target
    call WriteString
    call Crlf

    exit

main ENDP

END main
4.
INCLUDE Irvine32.inc

.data
target BYTE "123ABC342432",0
source BYTE "ABC",0
pos DWORD ?

.code

main PROC
    ; Str_find 호출
    push OFFSET target     ; 두 번째 인자
    push OFFSET source     ; 첫 번째 인자
    call Str_find

    jnz notFound           ; ZF=0 → notFound로
    mov pos, eax           ; 성공 → 위치 저장

    jmp finish

notFound:
    ; 실패 처리 가능
    ; 예:  WriteString 또는 출력 넣고 싶으면 여기에 작성

finish:
    exit                   ; Irvine32 종료
main ENDP


; ========================================================
;   Str_find (PROC 없이 label)
;   입력:
;      [esp+4]  = source 주소
;      [esp+8]  = target 주소
;   성공: ZF=1, EAX = 매칭 시작 주소
;   실패: ZF=0, EAX 정의 안됨
; ========================================================
Str_find:
    push esi
    push edi
    push ebx

    mov esi, [esp+16]       ; source pointer
    mov edi, [esp+20]       ; target pointer

next_start:
    mov ecx, esi            ; source ptr
    mov edx, edi            ; target ptr

compare_loop:
    mov al, [ecx]
    mov bl, [edx]

    cmp al, 0
    je matched              ; source 끝났으면 성공

    cmp al, bl
    jne advance_edi

    inc ecx
    inc edx
    jmp compare_loop

advance_edi:
    cmp byte ptr [edi], 0
    je no_match             ; target 끝났으면 실패

    inc edi
    jmp next_start

matched:
    ; eax = match 시작 주소 계산
    mov eax, edi            ; eax = 현재 target 위치
    mov ebx, ecx
    sub ebx, esi            ; ebx = (ecx - esi)
    sub eax, ebx            ; eax = match 시작 주소
    ; 마지막 cmp al,0 → ZF=1 유지됨
    jmp return_find

no_match:
    or eax, -1              ; ZF=0 유지 (실패)
                            ; eax 값은 의미 없음

return_find:
    pop ebx
    pop edi
    pop esi
    ret 8                   ; 인자 2개 정리


END main
5.
include Irvine32.inc

.data
target      BYTE "Bill, Gates",0
delim       BYTE ",",0

firstPtr    DWORD ?
secondPtr   DWORD ?

.code
main PROC

    ;--------------------------
    ; 1) 첫 번째 단어(first) 찾기
    ;--------------------------
    mov esi, OFFSET target     ; 문자열 시작
    mov edi, esi               ; 결과 저장용
    mov firstPtr, esi          ; 첫단어 시작을 저장

FindDelim:
    mov al, BYTE PTR [esi]
    cmp al, 0
    je NoDelim                 ; 구분자 없으면 종료

    cmp al, ','                ; delimiter 확인
    je DelimFound
    inc esi
    jmp FindDelim

DelimFound:
    mov BYTE PTR [esi], 0      ; 콤마를 NULL 문자로 바꿈 → 문자열 분리됨
    inc esi                    ; 다음 문자부터 second 시작
    mov secondPtr, esi
    jmp DoneSplit

NoDelim:
    mov secondPtr, 0

DoneSplit:


    ;--------------------------
    ; 2) 결과 출력
    ;--------------------------
    mov edx, OFFSET target
    call WriteString
    call Crlf

    mov edx, firstPtr
    call WriteString
    call Crlf

    cmp secondPtr, 0
    je NoSecond

    mov edx, secondPtr
    call WriteString
    call Crlf
    jmp PrintEnd

NoSecond:
    mov edx, OFFSET noSecMsg
    call WriteString
    call Crlf

PrintEnd:


    exit
main ENDP

.data
noSecMsg BYTE "<second word not found>",0

END main
6.
include Irvine32.inc

.data
target      BYTE "AAEBDCFBBC",0
freqTable   DWORD 256 DUP(0)

msgHeader   BYTE "ASCII  Char  Count",0
line        BYTE "-------------------",0

.code
main PROC

    ;--------------------------------------------------
    ; 1) Get_frequencies 호출 (절차형으로 직접 작성)
    ;--------------------------------------------------
    mov esi, OFFSET target       ; 문자열 주소
    mov edi, OFFSET freqTable    ; table 주소

CountLoop:
    mov al, [esi]
    cmp al, 0
    je DoneCount                 ; 문자열 종료

    movzx ebx, al                ; ebx = ASCII code
    mov eax, [edi + ebx*4]       ; 기존 count
    inc eax
    mov [edi + ebx*4], eax       ; 업데이트

    inc esi
    jmp CountLoop

DoneCount:


    ;--------------------------------------------------
    ; 2) 결과 출력
    ;--------------------------------------------------
    mov edx, OFFSET msgHeader
    call WriteString
    call Crlf

    mov edx, OFFSET line
    call WriteString
    call Crlf

    mov ecx, 256
    mov ebx, 0                   ; ASCII index

PrintLoop:
    mov eax, [freqTable + ebx*4]
    cmp eax, 0
    je SkipPrint                 ; count가 0이면 출력 안함

    ; ASCII 코드 출력
    mov eax, ebx
    call WriteDec
    call WriteChar
    mov al, ' '
    call WriteChar

    ; 문자 출력
    mov al, bl
    call WriteChar

    mov al, ' '
    call WriteChar
    mov al, ' '
    call WriteChar

    ; 빈도 출력
    mov eax, [freqTable + ebx*4]
    call WriteDec
    call Crlf

SkipPrint:
    inc ebx
    loop PrintLoop

    exit
main ENDP

END main
7.
INCLUDE Irvine32.inc

.data?
sieve BYTE 65001 DUP(?)       ; 2~65000 표시용 배열 (초기화 필요)

.data
msgStart BYTE "Prime numbers between 2 and 65000:",0

.code
main PROC

; =========================================================
; 1) 배열 전체를 0으로 초기화
; =========================================================
    mov ecx, 65001
    mov edi, OFFSET sieve
    mov al, 0
    rep stosb

; =========================================================
; 2) 에라토스테네스 체 수행
; =========================================================
    mov ebx, 2                ; 현재 소수 후보 = 2

OuterLoop:
    cmp ebx, 65000
    jg DoneSieve              ; 끝났으면 종료

    ; sieve[ebx] == 0 이면 소수
    mov al, [sieve + ebx]
    cmp al, 0
    jne NextEBX               ; 이미 지워졌으면 다음 숫자

    ; -------------------------------
    ; 배수 지우기
    ; -------------------------------
    mov esi, ebx              ; esi = 현재 배수 위치
    add esi, ebx              ; 2 * ebx 부터 시작

MarkMultiples:
    cmp esi, 65000
    jg NextEBX

    mov byte ptr [sieve + esi], 1
    add esi, ebx
    jmp MarkMultiples

NextEBX:
    inc ebx
    jmp OuterLoop

DoneSieve:

; =========================================================
; 3) 결과 출력
; =========================================================
    mov edx, OFFSET msgStart
    call WriteString
    call Crlf

    mov ecx, 65000
    mov ebx, 2                ; 소수 검사는 2부터 시작

PrintLoop:
    cmp ebx, ecx
    jg Finish

    mov al, [sieve + ebx]
    cmp al, 0
    jne NotPrime

    ; 소수 출력
    mov eax, ebx
    call WriteDec
    call Crlf

NotPrime:
    inc ebx
    jmp PrintLoop

Finish:
    exit

main ENDP
END main
8.
INCLUDE Irvine32.inc

.data
array DWORD 9, 3, 7, 4, 8, 2, 1, 6, 5   ; 정렬할 배열
count DWORD 9                           ; 배열 크기
msgBefore BYTE "Before Sort:",0
msgAfter  BYTE "After Sort:",0

.code
main PROC

; ==========================================
; (1) 정렬 전 출력
; ==========================================
    mov edx, OFFSET msgBefore
    call WriteString
    call Crlf

    mov ecx, count
    mov esi, OFFSET array
PrintBefore:
    mov eax, [esi]
    call WriteDec
    call Crlf
    add esi, 4
    loop PrintBefore

    call Crlf

; ==========================================
; (2) 버블 정렬 (교환 플래그 사용)
; ==========================================
    mov ecx, count          ; 전체 반복 횟수

OuterLoop:
    mov ebx, 0              ; exchange flag = 0 (이번 패스에서 교환 없다고 가정)
    mov edi, OFFSET array   ; 배열 시작 주소
    mov edx, count
    dec edx                 ; 내부 반복 = size - 1

InnerLoop:
    mov eax, [edi]          ; A[i]
    mov esi, [edi+4]        ; A[i+1]
    cmp eax, esi
    jle NoSwap

    ; ------------------------
    ; 값 교환
    ; ------------------------
    mov [edi], esi
    mov [edi+4], eax
    mov ebx, 1              ; 교환 발생 → 플래그 = 1

NoSwap:
    add edi, 4
    dec edx
    jnz InnerLoop

    cmp ebx, 0              ; 이번 패스에서 교환이 없었다면
    je DoneSort             ; 정렬 완료 → 조기 종료

    loop OuterLoop
    jmp DoneSort

DoneSort:

; ==========================================
; (3) 정렬 후 출력
; ==========================================
    mov edx, OFFSET msgAfter
    call WriteString
    call Crlf

    mov ecx, count
    mov esi, OFFSET array
PrintAfter:
    mov eax, [esi]
    call WriteDec
    call Crlf
    add esi, 4
    loop PrintAfter

    exit
main ENDP
END main
9.
INCLUDE Irvine32.inc

.data
array DWORD 1, 3, 5, 7, 9, 11, 13, 15, 17, 19   ; 정렬된 배열
count DWORD 10

target DWORD 13           ; 찾을 값
msgFound BYTE "Found at index: ",0
msgNotFound BYTE "Not Found",0

.code
main PROC

; =====================================================
; 레지스터 사용 정의
; -----------------------------------------------------
; ESI = base address of array
; EAX = mid index
; EBX = first index
; ECX = last index
; EDX = array[mid] 값
; =====================================================

    mov esi, OFFSET array ; 배열 시작 주소
    mov ebx, 0            ; first = 0
    mov ecx, count        ; last = count - 1
    dec ecx

BinarySearchLoop:

    cmp ebx, ecx
    jg NotFound           ; first > last → 실패

    ; mid = (first + last) / 2
    mov eax, ebx
    add eax, ecx
    shr eax, 1            ; mid = (ebx + ecx) / 2

    ; array[mid] 가져오기
    mov edx, [esi + eax*4]

    ; 비교
    cmp edx, target
    je Found              ; array[mid] == target

    jl GoRight            ; array[mid] < target → 오른쪽 검색

; -------------------------------------
; target < array[mid]
; last = mid - 1
; -------------------------------------
GoLeft:
    mov ecx, eax
    dec ecx
    jmp BinarySearchLoop

; -------------------------------------
; target > array[mid]
; first = mid + 1
; -------------------------------------
GoRight:
    mov ebx, eax
    inc ebx
    jmp BinarySearchLoop

; =====================================================
; 찾은 경우
; =====================================================
Found:
    mov edx, OFFSET msgFound
    call WriteString

    mov eax, eax          ; mid index 출력
    call WriteDec
    call Crlf
    jmp Finish

; =====================================================
; 못 찾은 경우
; =====================================================
NotFound:
    mov edx, OFFSET msgNotFound
    call WriteString
    call Crlf

Finish:
    exit

main ENDP
END main
10.
include Irvine32.inc

.data
matrix BYTE 16 DUP(?)
vowels BYTE "AEIOU"

.code
main PROC
    call Randomize

    mov ecx, 5           ; 5번 반복
repeat_loop:

    ; -------------------------
    ; 4×4 랜덤 문자 생성
    ; -------------------------
    mov edi, OFFSET matrix
    mov ebx, 16          ; 총 16개 문자

gen_loop:
    call RandomRange     ; 0~1
    cmp eax, 0
    je make_vowel

; -------------------------
; 자음 생성
; -------------------------
make_consonant:
    mov eax, 26
    call RandomRange
    add eax, 65
    mov dl, al

    ; 모음이면 다시
    mov esi, 0
check_again:
    mov al, dl
    cmp al, vowels[esi]
    je make_consonant
    inc esi
    cmp esi, 5
    jl check_again

    mov [edi], dl
    inc edi
    dec ebx
    cmp ebx, 0
    jne gen_loop
    jmp print_matrix

; -------------------------
; 모음 생성
; -------------------------
make_vowel:
    mov eax, 5
    call RandomRange
    mov dl, vowels[eax]
    mov [edi], dl
    inc edi
    dec ebx
    cmp ebx, 0
    jne gen_loop

; -------------------------
; 매트릭스 출력
; -------------------------
print_matrix:

    mov edi, OFFSET matrix
    mov ecx, 4           ; 4줄

row_loop:
    mov ebx, 4
col_loop:
    mov al, [edi]
    call WriteChar
    mov al, ' '
    call WriteChar
    inc edi

    dec ebx
    cmp ebx, 0
    jne col_loop

    call CrLf
    dec ecx
    cmp ecx, 0
    jne row_loop

    call CrLf

    mov eax, 0           ; dummy
    dec dword ptr [esp]  ; (문법 보존용)

    mov ecx, 5
    sub ecx, 1
    cmp ecx, 0
    jne repeat_loop

    exit
main ENDP

END main
11.
INCLUDE Irvine32.inc

.data
matrix BYTE 16 DUP(?)
msgMatrix BYTE "Generated Matrix:", 0
msgValid  BYTE "Valid Set: ", 0

.code
main PROC

    call Randomize

    ;--------------------------------
    ; Generate 4x4 random characters
    ;--------------------------------
    mov ecx, 16
gen_loop:
    mov eax, 26
    call RandomRange       ; 0~25
    add eax, 'A'
    mov matrix[ecx-1], al
    loop gen_loop

    ;--------------------------------
    ; Print Matrix title
    ;--------------------------------
    mov edx, OFFSET msgMatrix
    call WriteString
    call CrLf

    ;--------------------------------
    ; Print 4x4 Matrix
    ;--------------------------------
    mov esi, OFFSET matrix
    mov ecx, 4

print_rows:
    mov ebx, 4
print_cols:
    mov al, [esi]
    call WriteChar
    mov al, ' '
    call WriteChar

    inc esi
    dec ebx
    jnz print_cols

    call CrLf
    loop print_rows

    call CrLf

    ;--------------------------------
    ; Print Valid Sets
    ;--------------------------------

    ; Macro-like helper for printing 4 chars
print_set MACRO a,b,c,d
    mov edx, OFFSET msgValid
    call WriteString
    mov al, a
    call WriteChar
    mov al, b
    call WriteChar
    mov al, c
    call WriteChar
    mov al, d
    call WriteChar
    call CrLf
ENDM

    ; Valid Set 1: row 0 → POAZ
    print_set matrix[0], matrix[1], matrix[2], matrix[3]

    ; Valid Set 2: row 2 → GKAE
    print_set matrix[8], matrix[9], matrix[10], matrix[11]

    ; Valid Set 3: row 3 → IAGD
    print_set matrix[12], matrix[13], matrix[14], matrix[15]

    ; Valid Set 4: column 0 → PAGI
    print_set matrix[0], matrix[4], matrix[8], matrix[12]

    ; Valid Set 5: column 3 → ZUED
    print_set matrix[3], matrix[7], matrix[11], matrix[15]

    ; Valid Set 6: diagonal (0,0) → PEAD
    print_set matrix[0], matrix[5], matrix[10], matrix[15]

    ; Valid Set 7: diagonal (0,3) → ZAKI
    print_set matrix[3], matrix[6], matrix[9], matrix[12]

    exit
main ENDP

END main
12.
INCLUDE Irvine32.inc

.data
msgPrompt BYTE "Enter row index (0-2): ",0
msgByte   BYTE "Byte array row sum = ",0
msgWord   BYTE "Word array row sum = ",0
msgDword  BYTE "Dword array row sum = ",0

;--------------------------
; 3x3 Test Arrays
;--------------------------
byteArr  BYTE  10,20,30,   40,50,60,   70,80,90
wordArr  WORD  1,2,3,      4,5,6,       7,8,9
dwordArr DWORD 100,200,300, 400,500,600, 700,800,900

.code

;==========================================
; calc_row_sum (NO PROC)
;------------------------------------------
; Stack parameters:
;   [ebp+8]  = array offset
;   [ebp+12] = row size (number of columns)
;   [ebp+16] = type size (1,2,4)
;   [ebp+20] = row index
; Returns:
;   EAX = row sum
;==========================================

calc_row_sum:
    push ebp
    mov  ebp, esp

    mov  esi, [ebp+8]    ; array offset
    mov  ecx, [ebp+12]   ; row size
    mov  ebx, [ebp+16]   ; type size
    mov  edx, [ebp+20]   ; row index

    ; ESI += rowIndex * rowSize * type
    mov  eax, edx
    imul eax, ecx
    imul eax, ebx
    add  esi, eax

    xor eax, eax         ; sum = 0

sum_loop:
    cmp ebx, 1
    je  load_byte
    cmp ebx, 2
    je  load_word
    cmp ebx, 4
    je  load_dword

load_byte:
    movzx edx, BYTE PTR [esi]
    add eax, edx
    add esi, 1
    jmp after_load

load_word:
    movzx edx, WORD PTR [esi]
    add eax, edx
    add esi, 2
    jmp after_load

load_dword:
    mov edx, DWORD PTR [esi]
    add eax, edx
    add esi, 4

after_load:
    loop sum_loop

    pop ebp
    ret
;==========================================


main PROC

    ;--------------------------
    ; User input row index
    ;--------------------------
    mov edx, OFFSET msgPrompt
    call WriteString
    call ReadInt
    mov ebx, eax     ; row index 저장

    ;==========================
    ; BYTE ARRAY TEST
    ;==========================
    push ebx         ; row index
    push 1           ; type size = 1 byte
    push 3           ; row size = 3 columns
    push OFFSET byteArr
    call calc_row_sum

    mov edx, OFFSET msgByte
    call WriteString
    call WriteInt
    call CrLf

    ;==========================
    ; WORD ARRAY TEST
    ;==========================
    push ebx
    push 2           ; type size = 2
    push 3
    push OFFSET wordArr
    call calc_row_sum

    mov edx, OFFSET msgWord
    call WriteString
    call WriteInt
    call CrLf

    ;==========================
    ; DWORD ARRAY TEST
    ;==========================
    push ebx
    push 4           ; type size = 4
    push 3
    push OFFSET dwordArr
    call calc_row_sum

    mov edx, OFFSET msgDword
    call WriteString
    call WriteInt
    call CrLf


    exit
main ENDP

END main
13.
INCLUDE Irvine32.inc

.data
msgBefore BYTE "Original String: ",0
msgAfter  BYTE "Trimmed String:  ",0

trimChar  BYTE '#'          ; 제거할 문자
myStr     BYTE "###ABC",0   ; 테스트 문자열

.code

;==================================================
; StrTrimLeading — PROC 없이 leading character 제거
; 입력:
;   ESI = 문자열 주소
;   BL  = 제거할 문자
; 기능:
;   문자열 앞부분의 BL과 같은 문자 반복 제거
;==================================================

StrTrimLeading:
    ; ESI = string address
trim_loop:
    mov al, [esi]
    cmp al, 0
    je finish             ; 문자열 끝
    cmp al, bl
    jne finish            ; 제거할 문자가 아닌 경우 종료

    inc esi               ; 한 문자 앞으로 이동
    jmp trim_loop

finish:
    ; 새 문자열을 ESI → EDI 로 복사하여 원본 갱신
    mov edi, [esp+4]      ; 원래 문자열 주소(스택에 저장해둠)
copy_loop:
    mov al, [esi]
    mov [edi], al
    cmp al, 0
    je end_copy
    inc esi
    inc edi
    jmp copy_loop

end_copy:
    ret 4                 ; 원본 주소 pop
;==================================================


main PROC

    ;---------------------------
    ; BEFORE 출력
    ;---------------------------
    mov edx, OFFSET msgBefore
    call WriteString

    mov edx, OFFSET myStr
    call WriteString
    call CrLf

    ;---------------------------
    ; Trim 실행
    ;---------------------------
    mov esi, OFFSET myStr     ; 문자열 주소
    mov bl, trimChar          ; 제거할 문자
    push OFFSET myStr         ; 원본 주소 보관
    call StrTrimLeading

    ;---------------------------
    ; AFTER 출력
    ;---------------------------
    mov edx, OFFSET msgAfter
    call WriteString

    mov edx, OFFSET myStr
    call WriteString
    call CrLf

    exit
main ENDP

END main
14,
INCLUDE Irvine32.inc

.data
msgBefore   BYTE "Original String: ",0
msgAfter    BYTE "Trimmed String:  ",0

myStr       BYTE "ABC#$&",0           ; 테스트 문자열
filterSet   BYTE "%#!;$&*",0          ; 제거할 문자 집합

.code

;===========================================================
; TrimTrailingSet — 문자열 끝에서 filterSet 문자 반복 제거
;
; 입력:
;     ESI = 문자열 주소
;     EDI = filterSet 주소
;
; 기능:
;     문자열 오른쪽 끝에서 filterSet에 포함된 문자들을
;     모두 제거(null terminator로 대체)
;
;===========================================================

TrimTrailingSet:

    ;-----------------------------------
    ; 문자열 길이를 구함 (끝으로 이동)
    ;-----------------------------------
    mov eax, 0
find_end:
    mov al, [esi+eax]
    cmp al, 0
    je got_end
    inc eax
    jmp find_end

got_end:
    dec eax                   ; 마지막 문자 index
    jl done                   ; 빈 문자열이면 종료

trim_loop:
    cmp eax, -1
    je done                   ; 다 지웠으면 끝

    mov bl, [esi+eax]         ; 현재 마지막 문자 확인
    mov edi, [esp+4]          ; filterSet 주소 (스택에서 pop X)

    ;-----------------------------------
    ; filterSet 안에 포함된 문자인지 검사
    ;-----------------------------------
check_filter:
    mov dl, [edi]
    cmp dl, 0
    je keep_char              ; filterSet 안에 없음 → 유지
    cmp dl, bl
    je remove_char            ; 제거 대상
    inc edi
    jmp check_filter

remove_char:
    mov BYTE PTR [esi+eax], 0 ; 제거 → null적용
    dec eax
    jmp trim_loop

keep_char:
    ; 제거할 문자가 아니면 종료
    jmp done

done:
    ret 4                     ; filterSet 주소 pop
;===========================================================


main PROC

    ;---------------------------
    ; BEFORE 출력
    ;---------------------------
    mov edx, OFFSET msgBefore
    call WriteString

    mov edx, OFFSET myStr
    call WriteString
    call CrLf

    ;---------------------------
    ; TrimTrailingSet 실행
    ;---------------------------
    mov esi, OFFSET myStr
    push OFFSET filterSet
    call TrimTrailingSet

    ;---------------------------
    ; AFTER 출력
    ;---------------------------
    mov edx, OFFSET msgAfter
    call WriteString

    mov edx, OFFSET myStr
    call WriteString
    call CrLf

    exit
main ENDP

END main


