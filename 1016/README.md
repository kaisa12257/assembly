Data Transfers, Addressing, and Arithmetic
레지스터는 컴퓨터에서 cpu안에서 사용된다. 그리고 레지스터8,16,12는 각각 메모리에서 8비트 16비트 12비트로서 사용이 된다.
명령어에서 피연산자들은 같은 크기 들이며 또한 명령어에서 등장하는 ECX란 범용레지스터 써로 cpu내부의 데이터 저장 임시공간이다.
또한 inc는 +1을 dec는 -1을 준다. 소스 피연산자는 연산에 의해 바뀌지 않고그 합은 목적 피연산자에 저장된다
data
var1 DWORD 10000h
var2 DWORD 20000h
.code
mov eax, varl; EAX = 10000h
add eax, var2; EAX = 30000h
그리고 flag에서 산술 연산의 결과가 0이 되면, 제로 플래그(Zero flag)가 설정된다.
mov ecx, 1
sub ecx, 1
mov eax, 0FFFFFFFFh
inc eax
inc eax
dec eax
또한 정수 연산에서 오버플로우가 발생할 경우에는 두 양수의 합이 음수가 되며 두 음수의 합이 양수가 된다

; Addition and Subtraction (AddSubTest.asm)
; Chapter 4 example. Demonstration of ADD, SUB,
; INC, DEC, and NEG instructions, and how
; they affect the CPU status flags.
.386
.model flat,stdcall
.stack 4096
ExitProcess proto,dwExitCode:dword
.data
Rval SDWORD ?
Xval SDWORD 26
Yval SDWORD 30
Zval SDWORD 40
.code
main proc
; INC and DEC
mov ax,1000h
inc ax ; 1001h
dec ax ; 1000h

또한 어셈블리에서 OFFSET 연산자는 데이터 레이블(label)의 오프셋(offset)을 반환한다., 그리고 인덱스 주소 지정(indexed operand)은 레지스터의 값에 상수(constant)를 더해서 유효 주소(effective address)를 생성한다.
어셈블리에서 TYPEDEF 연산자는 사용자가 정의한 타입(user-defined type)을 만들 수 있게 해주며, 이 타입은 변수를 정의할 때 내장 타입(built-in type)과 동일한 특성을 가진다.
