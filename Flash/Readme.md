#### Chế độ Boot Mode
- 3 chế độ khởi động chính:
    - Main Flash Memory: Khởi động từ bộ nhớ Flash chính, nơi chương trình được nạp (0x08000000).
    - System Memory (Bootloader): Khởi động bootloader để nạp code thông qua UART, USB (0x1FFF000).
    - Embedded SRAM: Khởi động từ SRAM dùng để thử nghiệm hoặc debug code (0x20000000).
#### Volatile vs Non-Volatile Memory
- Volatile Memory là loại bộ nhớ mà chỉ lưu trữ được thông tin dữ liệu khi duy trì nguồn điện, khi cắt nguồn điện Volatile Memory sẽ mất hết dữ liệu và RAM là một loại Volatile Memory. Volatile Memory thường được dùng để lưu trữ dữ liệu trong quá trình run-time.
- Non-Volatile Memory là loại bộ nhớ lưu trữ thông tin khi duy trì nguồn điện và vẫn giữ thông tin khi bị ngắt nguồn điện. Điển hình của Non-Volatile Memory là Flash, EEPROM,... Non-Volatile Memory thường dùng để lưu trữ chương trình, các cấu hình của hệ thống.

#### Flash
![1](/Nạp%20code/1.png)
- Các vùng trong Flash:
    - Main Stack Pointer (MSP).
    - Vector Table: Lưu trữ các địa chỉ của trình xử lí ngắt (ISR), Reset_Handler.
    - .text: Chứa mã chương trình đã được biên dịch.
    - .rodata: Lưu trữ các dữ liệu chỉ đọc.
    - .data (Initialized Data): Lưu các giá trị khởi tạo của các biến toàn cục hoặc tĩnh có khởi tạo. Những giá trị này sẽ được sao chép vào RAM khi khởi động.
- Quy trình ghi dữ liệu vào Flash:
    - Mở khóa Flash.
    - Kiểm tra trạng thái Flash.
    - Xóa trang Flash (Nếu cần thiết).
    - Ghi dữ liệu vào Flash.

Địa chỉ bộ nhớ Flash bắt đầu từ 0x00000000, nhưng trong Vi điều khiển STM32, vùng nhớ code được lưu trữ từ địa chỉ 0x08000000, khi chúng ta nạp xuống, chương trình mặc định nạp từ địa chỉ này, với MSP ở địa chỉ 0x08000000 và Vector Table bắt đầu từ địa chỉ 0x08000004 (Reset_Handler).

- Tổ chức vùng nhớ Flash
    Vi điều khiển STM32F103C8:
    - Dung lượng 64Kbytes.
    - Main Memory Block chia thành 32 trang mỗi trang 1Kbytes, bắt đầu từ 0x08000000 đến 0x080013FF.
    - System Memory 2Kbytes sử dụng cho Device Boots ở System Memory Boot Mode.
    - 16Bytes Options Bytes để configure việc đọc ghi, reset khi thiết bị ở mode Standby/Stop.
    ![3](/Nạp%20code/3.png)

- Lock/Unlock Flash
    - Mặc định thanh ghi Control của Flash sẽ bị Lock để tránh truy cập bất thường.
    - Trạng thái Lock/Unlock của thanh ghi FLASH_CR được quy đinh bởi bit LOCK của thanh ghi FLASH_CR (LOCK = 1 thanh ghi FLASH_CR ở trạng thái Lock và ngược lại được Unlock khi bit LOCK = 0).
    - Ghi giá trị KEY1 = 0x45670123, KEY2 = 0xCDEF89AB vào thanh ghi FLASH_KEYR thì bit LOCK trên thanh ghi FLASH_CR sẽ được reset về bằng 0.

- Xóa Flash
    Bộ nhớ Flash hỗ trợ xóa từng trang hoặc xóa toàn bộ. Để thực hiện xóa một trang:
    - Đảm bảo không có thao tác nào đang diễn ra trước khi thực hiện xóa bằng cách check bit BSY trong thanh ghi FLASH_SR.
    - Set bit PER trong thanh ghi FLASH_CR.
    - Ghi địa chỉ đầu trang cần xóa vào thanh ghi FLASH_AR.
    - Set bit STRT của thanh ghi FLASH_CR.
    - Kiểm tra bit BSY trong thanh ghi FLASH_SR để xác định khi nào thao tác hoàn tất. Khi BSY = 0 thì thao tác hoàn thành.
    ![4](/Nạp%20code/4.png)
    ![5](/Nạp%20code/5.png)

    Xóa toàn bộ Flash:
    - Đảm bảo không có thao tác nào đang diễn ra trước khi thực hiện xóa bằng cách check bit BSY trong thanh ghi FLASH_SR.
    - Set bit MER trong thanh ghi FLASH_CR.
    - Set bit STRT của thanh ghi FLASH_CR.
    - Kiểm tra bit BSY trong thanh ghi FLASH_SR để xác định khi nào thao tác hoàn tất. Khi BSY = 0 thì thao tác hoàn thành.
    ![6](/Nạp%20code/6.png)
    ![7](/Nạp%20code/7.png)
- Ghi vào Flash
    Flash hỗ trợ ghi theo half-word. Để thực hiện ghi:
    - Đảm bảo không có thao tác nào đang diễn ra trước khi thực hiện xóa bằng cách check bit BSY trong thanh ghi FLASH_SR.
    - Set bit PG trong thanh ghi FLASH_CR.
    - Ghi dự liệu half-word vào địa chỉ cần ghi.
    - Kiểm tra bit BSY trong thanh ghi FLASH_SR để xác định khi nào thao tác hoàn tất. Khi BSY = 0 thì thao tác hoàn thành.

    ![8](/Nạp%20code/8.png)
    ![9](/Nạp%20code/9.png)
#### RAM
![2](/Nạp%20code/2.png)
