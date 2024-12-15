#### Startup file

##### Start Procedure
Địa chỉ bắt đầu của bộ nhớ (STM32 là FLASH) là 0x00000000. Phần đầu tiên của bộ nhớ chứa đó là Vector Table - Chứa các vector ngắt điển hình nhất chính là Vector Reset.
![1](/Biên%20dịch/1.png)

- CPU sẽ tìm kiếm bảng Vector ngắt của chương trình trong bộ nhớ (Đọc thanh ghi SCB_VTOR). Ở trạng thái reset, SCB_VTOR = 0, CPU sẽ tìm đến địa chỉ 0x00000000.
- Bộ xử lí tìm nạp tại ô nhớ đầu tiên của Vector Table (0x00000000), vào thanh ghi Main Stack Pointer (MSP).
- Bộ xử lí sẽ lấy địa chỉ của Reset Handler được đặt trong ô nhớ thứ 2 của Vector Table, tại địa chỉ 0x00000004 và load vào PC để thực hiện hàm Reset Handler, Vector Reset cũng là 1 Exception của hệ thống, nó sẽ làm những công việc setup hệ thống ban đầu trước khi gọi hàm main.
- Sau khi nhảy đến Reset Handler, bộ xử lí bắt đầu thực hiện các lệnh trong Reset Handler:
    - Khởi tạo vùng nhớ bss, data
    - Khởi tạo thư viện C standard.
    - Gọi đến hàm main() trong file main.c

##### Vector Table
ARM-CortexM3 có 15 exceptions và 43 interrupt vector

- Định nghĩa các địa chỉ bắt đầu SRAM: 0x20000000, size của SRAM 20Kb, địa chỉ cuối cùng của SRAM = Địa chỉ bắt đầu + size, địa chỉ khởi đầu của Stack Pointer = Địa chỉ cuối cùng của SRAM
```
#define SRAM_START 0x20000000U
#define SRAM_SIZE  (20U * 1024U)//20KB
#define SRAM_END   ((SRAM_START) + (SRAM_SIZE))
#define STACK_START SRAM_END
```
- Địa chỉ bảng vecto
- Xác định mã số ngoại lệ/ngắt
- Tính toán vị trí trong bảng vector
- Lấy địa chỉ handler
- CPU nhảy đến địa chỉ handler
#####
```
void NMI_Handler(void) __attribute__((weak, alias("Default_Handler")));
```
```
uint32_t vectors[] __attribute__((section(".isr_vector"))) = {
    STACK_START,//địa chỉ khởi tạo stack pointer (SP)
    (uint32_t)Reset_Handler,
    (uint32_t)NMI_Handler,
    ...
};
```
- __ __attribute__ __: là cú pháp mở rộng của GCC để cung cấp thông tin bổ sung về hàm, biến hoặc kiểu dữ liệu. Nó cho phép điều chỉnh các đặc tính của đối tượng trong code.

    ![2](/Biên%20dịch/2.png)

    - __section__: là một thuộc tính của attribute đặt biến hoặc hàm vào một vùng nhớ cụ thể.
    - __alias__: là một thuộc tính của attribute để tạo ra một tên khác cho một hàm, có nghĩa là 2 tên khác nhau sẽ cùng trỏ đến cùng 1 định nghĩa hàm.
    ![3](/Biên%20dịch/3.png)
    - __weak__: là một thuộc tính của attribute để đánh dấu 1 hàm hoặc biến là yếu, cho phép nó bị ghi đè bởi một định nghĩa mạnh hơn ở nơi khác trong chương trình.
    ![4](/Biên%20dịch/4.png)
    ![5](/Biên%20dịch/5.png)
##### Reset handler
- Định nghĩa các vùng bộ nhớ .data,.bss trong RAM
```
extern uint32_t _etext;//kết thúc vùng nhớ .text
extern uint32_t _sdata;//bắt đầu vùng nhớ .data
extern uint32_t _edata;//kết thúc vùng nhớ .data
extern uint32_t _sbss;//bắt đầu vùng nhớ .bss
extern uint32_t _ebss;//kết thúc vùng nhớ .bss
```
- Khởi tạo 2 vùng nhớ .bss .data:
    - Copy .data từ FLASH qua RAM
    ```
    uint32_t size = (uint32_t)&_edata - (uint32_t)&_sdata;
    uint8_t *pDst = (uint8_t *)&_sdata;//sram
    uint8_t *pSrc = (uint8_t *)&_etext;//flash

    for (uint32_t i = 0; i < size; i++)
    {
        *pDst++ = *pSrc++;
    }
    ```
    - Khởi tạo vùng nhớ .bss với giá trị bằng 0
    ```
    size = (uint32_t)&_ebss - (uint32_t)&_sbss;
    pDst = (uint8_t *)&_sbss;
    for (uint32_t i = 0; i < size; i++)
    {
        *pDst = 0;
    }
    ```

    ![6](/Biên%20dịch/6.png)
- Khởi tạo standard library (nếu có).
- Gọi hàm main().

#### Linker file
Quá trình build thì các file .o sẽ được tạo ra sau quá trình Compile và Assemble. Quá trình Compile này sẽ tạo ra địa chỉ tương đối cho mỗi file object, bao gồm text, data, bss, rodata, tức là mỗi file object đang bắt đầu từ địa chỉ 0x0000.0000.

Như vậy, các file objects có thể bị conflict lẫn nhau. và địa chỉ này sẽ không phù hợp với địa chỉ mong muốn mà ta muốn nạp xuống vi điều khiển. 

Vì vậy, cần merge các file object trên với nhau, và phân bổ vào bộ nhớ vi điều khiển như người dùng mong muốn.

Và file giúp làm điều đó chính là file Linker Script (.ld). 
##### ENTRY
```
ENTRY(Reset_Handler)
```
Dùng để set thông tin "Entry point address" ở đầu file .elf. Debugger sẽ dùng entry point này để biết hàm nào được chạy đầu tiên.
##### MEMORY Command
```
MEMORY
{
    FLASH(rx) : ORIGIN = 0x08000000, LENGTH = 64K
    SRAM(rwx) : ORIGIN = 0x20000000, LENGTH = 20K
}
```
Lệnh này dùng để set các phân vùng bộ nhớ (FLASH SRAM), bao gồm thông tin địa chỉ bắt đầu và kích thước của các vùng nhớ tương ứng, cùng với đó là các Attribute (cho phép đọc/ghi).

![7](/Biên%20dịch/7.png)

##### Section Command

```
.text :
    {
        *(.isr_vector)//đưa bảng vecto ngắt vào vùng .text
        *(.text) //đưa mã chương trình vào vùng .text
        *(.rodata) //đưa mã dữ liệu chỉ đọc vào vùng .text 
        _etext = .;//Lưu địa chỉ kết thúc vào vùng .text
        . = ALIGN(4);
    }>FLASH // đặt toàn bộ vùng nhớ .text vào FLASH
```
```
.data :
    {
        _sdata = .;
        *(.data)
        _edata = .;
        . = ALIGN(4);
    }>SRAM AT> FLASH
```

```
.bss :
    {
        _sbss = .;
        *(.bss)
        . = ALIGN(4);
        _ebss = .;
    } >SRAM
```

AT>: Lệnh này được sử dụng sau mỗi Section, được dùng để định nghĩa hai vùng nhớ trước và sau runtime.
- Vùng nhớ data trước khi run-time được phân bổ ở VMA (virtual memory address) và được phân bổ trên LMA (load memory address) khi run time
