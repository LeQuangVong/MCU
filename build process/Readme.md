#### Chương trình C trên máy tính
##### Preprocessor 
![1](/build%20process/1.png)

Quá trình Preprocessor - tiền xử lý là quá trình chuẩn bị  cho quá trình Compile, ở giai đoạn này các file source code (.c, .cpp) sẽ được gộp với các file header (.h) để tạo thành các Extended Files (.i). Mặt khác trong quá trình này, các chỉ thị tiền xử lý như #define, #ifdef, ... sẽ được xử lý để thay thế.
##### Compilation
- Các file source được Compile bởi Compiler tạo file Assembly (.asm).
- Compiler được sử dụng là các compiler thông dụng như GCC.
##### Assemble
- Các file .asm (từ bước trên / source file) tiếp tục được dịch bằng trình Assembler
- Đầu ra là các file object (.o)
##### Linking
- Các file .o được link với các file thư viện tĩnh (.a / .lib) bởi trình linker.
- Đầu ra là file .exe cuối cùng để chạy.
#### Chương trình C trên Vi điều khiển
![2](/build%20process/2.png)
- Sử dụng Cross Compiler, có nghĩa là mỗi con chip sẽ cần có những cách build chương trình ra file nhị phân khác nhau, sử dụng Cross Compiler, nó cho phép mình build chương trình trên một máy tính (chip khác - x86 Intel chẳng hạn), rồi nạp và chạy trên một chip khác (ARM).
    Máy tính dùng để build chương trình được gọi là host, môi trường chip chạy gọi là target.

- Các file Object vẫn được link với thư viện tĩnh, nhưng cần trải qua một bước nữa đó là Locator, bước này dùng 1 file Linker Script để phân bố vùng nhớ cho vi điều khiển - Khác với máy tính thì IDE và Hệ điều hành đã tự động Locator và ẩn đi file Linker Script này.
- File đích tạo ra không còn là .exe mà là các file như .elf (dùng để execute & debug), các file định dạng khác như .hex / .bin / .srec / .axf / ... để nạp code và debug.
![3](/build%20process/3.png)