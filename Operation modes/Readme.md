![1](/Operation%20modes/1.png)

#### Thread mode
- Khi khởi động, vi xử lý đều bắt đầu ở thread mode, ở chế độ này code của chương trình được thực hiện tuần tự trong hàm main(). Bất cứ khi nào xảy ra Ngắt hay Exception, vi xử lí sẽ chuyển sang chế độ Handler Mode. Ở chế độ này thanh ghi SP được sử dụng có thể là MSP hoặc PSP.
#### Handler Mode.
Bất cứ khi nào xảy ra Ngắt hay Exception, vi xử lí nhảy vào ISR tương ứng, tức là chuyển sang chế độ Handler Mode. Ở chế độ này chỉ có MSP được sử dụng.
Để phân biệt 2 chế độ này, vi xử lí chỉ thị bằng 9 bit LSB của thanh ghi IPSR.

