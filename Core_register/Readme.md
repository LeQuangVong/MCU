![1](/Core_register/1.png)

- Các thanh ghi R0-R12 là các thanh ghi mục đích chung, có tác dụng xử lí data, tính toán. Tức là các phép tính toán, toán tử, toán hạng, kết quả,... sẽ được ALU tối ưu để đặt trên các thanh ghi này.
- R13 (SP - Stack Pointer) trỏ đến một vùng nhớ đặt biệt gọi là Stack. Thanh ghi này có thể là MSP (Main Stack Pointer) hoặc PSP (Process Stack Pointer) dùng cho các task RTOS. Khi reset nó luôn = 0x00000000.
- R14 (LR - Link Register) là thanh ghi chứa địa chỉ trả về của các lệnh rẽ nhánh (ví dụ như khi gọi 1 hàm thì LR sẽ chứa địa chỉ của lệnh tiếp theo sau khi hàm đó trả về).
- R15 (PC - Program Counter) sử dụng để điều khiển quá trình thực thi các lệnh trong chương trình, PC sẽ chứa địa chỉ của lệnh tiếp thoe mà Processor sẽ xử lí.
- PSR (Program status register) boa gồm 3 thanh ghi:
![4](/Core_register/4.png)
    - Application Program Status Register (APSR): chưa các cờ phục vụ cho quá trình tính toán của chương trình.
    ![5](/Core_register/5.png)
    - Interrupt Program Status Register (IPSR):
    Sử dụng 8 bit LSB để chứa exception của chương trình hiện tại.
    ![2](/Core_register/2.png)
        - Khi ở main ở chế độ Thread Mode thì 8 bit cuối luôn bằng 0.
    ![3](/Core_register/3.png)
        - Khi vào chương trình ngắt ở chế độ Handler Mode thì 8 bit cuối thay đổi theo giá trị của IRQ Number.
    - Execution Program Status Register (EPSR):
        - PC sẽ trỏ đến địa chỉ của lệnh tiếp theo mà Processor sẽ xử lí
        ![6](/Core_register/6.png)
        - khi vào trong hàm, thanh ghi LR sẽ chứa địa chỉ của lệnh tiếp theo sau khi hàm đó trả về.
        ![7](/Core_register/7.png)
        - Sau khi thực hiện xong giá trị của thanh ghi LR sẽ được gán cho PC
        ![8](/Core_register/8.png)
        Nhưng vẫn bị lệch 1 bit do T bit (Bit[24]) dùng để xác định tập lệnh mà Processor đang sử dụng.
        - T = 1 : Tập lệnh Thumb ISA, các câu lệnh phổ biến đều dài 16 bit và một số dài 32 bit.
        - T = 0 : Tập lệnh ARM ISA, Các câu lệnh dài 32 bit.
        Arm Cortex M hổ trợ cho bộ xử lí Thumb nên T bit bằng 1.
#### ASSEMBLY
- Các core register không có địa chỉ, nên không thể truy cập bằng ngôn ngữ C, chỉ có thể truy cập bằng ngôn ngữ Assembly.
Ex: assembly: LDR R0,R1
Trong C:      _asm volatile("LDR R0,[R1]");

```
//Tải địa chỉ 0x20001000 vào thanh ghi R1
__asm volatile(" LDR R1, =#0x20001000");

//Tải địa chỉ 0x20001004 vào thanh ghi R2
__asm volatile(" LDR R2, =#0x20001004");

//Tải giá trị tại địa chỉ được chứa trong R1 vào thanh ghi R0
__asm volatile(" LDR R0, [R1]");

//Tải giá trị tại địa chỉ được chứa trong R2 vào thanh ghi R1
__asm volatile(" LDR R1, [R2]");

//Cộng giá trị R0 và R1 lưu vào thanh ghi R0
__asm volatile(" ADD R0,R0,R1");

//Lưu giá trị trong R0 (vừa tính được) vào địa chỉ mà R2 trỏ đến (0x20001004)
__asm volatile(" STR R0, [R2]");
```
