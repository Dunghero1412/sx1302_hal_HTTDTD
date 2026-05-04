# LoRaWan gate way hal SX1303
### dự án được import từ sx1302_hal của Lora-net


/ _____)             _              | |
	( (____  _____ ____ _| |_ _____  ____| |__
	 \____ \| ___ |    (_   _) ___ |/ ___)  _ \
	 _____) ) ____| | | || |_| ____( (___| | | |
	(______/|_____)_|_|_| \__)_____)\____)_| |_|
	  (C)2020 Semtech


```markdown
# Dự án SX1302 LoRa Gateway

## 1. Thư viện lõi: libloragw

Thư mục này chứa mã nguồn của thư viện để xây dựng một gateway dựa trên chip tập trung LoRa SX1302 của Semtech (hay còn gọi là bộ tập trung). Sau khi biên dịch, toàn bộ mã nguồn được chứa trong file `libloragw.a` và sẽ được liên kết tĩnh (tức là nhúng trực tiếp vào file thực thi cuối cùng).

Thư viện cũng đi kèm với một số chương trình kiểm tra cơ bản, được dùng để kiểm tra các module con khác nhau của thư viện.

Vui lòng tham khảo file `readme.md` trong thư mục `libloragw` để biết thêm chi tiết.

## 2. Chương trình trợ giúp

Các chương trình này được đưa vào dự án để cung cấp ví dụ về cách sử dụng thư viện HAL, đồng thời giúp người xây dựng hệ thống kiểm tra các phần khác nhau của nó.

### 2.1. Bộ chuyển tiếp gói (packet_forwarder)

Bộ chuyển tiếp gói là một chương trình chạy trên máy chủ của gateway LoRa, có nhiệm vụ chuyển tiếp các gói RF nhận được từ bộ tập trung đến máy chủ qua đường liên kết IP/UDP và phát ra các gói RF do máy chủ gửi xuống.



**Đường lên (Uplink):** Các gói vô tuyến được gateway nhận, cùng với siêu dữ liệu (metadata) do gateway thêm vào, được chuyển tiếp đến máy chủ. Có thể bao gồm cả trạng thái của gateway.

**Đường xuống (Downlink):** Các gói do máy chủ tạo ra, kèm theo siêu dữ liệu bổ sung, để gateway truyền trên kênh vô tuyến. Có thể bao gồm dữ liệu cấu hình cho gateway.

Vui lòng tham khảo file `readme.md` trong thư mục `packet_forwarder` để biết thêm chi tiết.

### 2.2. util_net_downlink

Bộ gửi gói đường xuống là một chương trình trợ giúp đơn giản, lắng nghe trên một cổng UDP duy nhất, phản hồi các datagram `PUSH_DATA` và `PULL_DATA` bằng ACK thích hợp, đồng thời gửi các gói JSON đường xuống đến socket với các tham số khung đã cho, theo khoảng thời gian đều đặn. Đây là một bộ gửi gói mạng.

Nó cũng có thể được sử dụng như một bộ ghi log gói UDP để lưu trữ các đường lên nhận được trong file CSV cục bộ.

Vui lòng tham khảo file `readme.md` trong thư mục `util_net_downlink` để biết thêm chi tiết.

### 2.3. util_chip_id

Tiện ích này cấu hình SX1302 để có thể truy xuất EUI của nó. Sau đó, EUI có thể được sử dụng làm ID Gateway.

### 2.4. util_boot

Chỉ dùng cho gateway USB, phần mềm này chuyển bộ tập trung sang chế độ DFU để lập trình vi điều khiển STM32 bên trong nó.

### 2.5. util_spectral_scan

Phần mềm này cho phép quét băng tần quang phổ bằng radio sx1261 bổ sung trên thiết kế tham khảo Corecell của Semtech.

## 3. Script trợ giúp

### 3.1. tools/reset_lgw.sh

Script này dùng để thực hiện khởi tạo cơ bản cho SX1302 thông qua các GPIO được định nghĩa bởi thiết kế tham khảo CoreCell. Nó đưa SX1302 ra khỏi trạng thái reset và thiết lập chân Power Enable. Script này được gọi bởi mọi chương trình cung cấp ở đây có truy cập vào SX1302. Nó **PHẢI** nằm trong cùng thư mục với file thực thi của chương trình.

## 4. Hướng dẫn biên dịch, cài đặt và chạy

Tất cả thư viện và chương trình kiểm tra có thể được biên dịch và cài đặt từ thư mục gốc của dự án này.

### 4.1. Dọn dẹp và biên dịch mọi thứ

`make clean all`

### 4.2. Cài đặt các file thực thi và file liên quan vào một thư mục

Trước tiên, hãy sửa file `target.cfg` nằm trong thư mục gốc của dự án để cấu hình nơi sẽ cài đặt các file thực thi.

- `TARGET_IP`: địa chỉ IP của máy chủ gateway. Trong trường hợp dự án được biên dịch ngay trên máy chủ gateway (Raspberry Pi...), có thể để là `localhost`.
- `TARGET_DIR`: thư mục trên hệ thống file của máy chủ gateway, nơi các file thực thi sẽ được sao chép đến. Lưu ý rằng thư mục **PHẢI** tồn tại khi gọi lệnh cài đặt.
- `TARGET_USR`: người dùng Linux sẽ được dùng để thực hiện lệnh SSH/SCP để sao chép các file thực thi.

Để tránh phải nhập mật khẩu người dùng khi cài đặt, hãy thực hiện các bước sau:

Giả sử bạn muốn sao chép giữa hai máy chủ `host_src` và `host_dest` (có thể là cùng một máy). `host_src` là máy bạn sẽ chạy lệnh `scp`, bất kể chiều sao chép file là gì!

* Trên `host_src`, chạy lệnh này với tư cách người dùng sẽ chạy `scp`:
  `ssh-keygen -t rsa`
  Lệnh này sẽ yêu cầu nhập mật khẩu (passphrase). Chỉ cần nhấn phím Enter. Nó sẽ tạo một khóa nhận dạng (khóa riêng tư) và một khóa công khai. Đừng bao giờ chia sẻ khóa riêng tư với bất kỳ ai! `ssh-keygen` sẽ hiển thị vị trí lưu khóa công khai. Mặc định là `~/.ssh/id_rsa.pub`.
* Chuyển file `id_rsa.pub` đến `host_dest`:
  `ssh-copy-id -i ~/.ssh/id_rsa.pub user@host_dest`
  Bạn sẽ có thể đăng nhập vào `host_dest` mà không cần nhập mật khẩu.

Sau khi mọi thứ được thiết lập, có thể gọi lệnh sau:
`make install`

Để cũng cài đặt các file cấu hình JSON của bộ chuyển tiếp gói:
`make install_conf`

### 4.3. Biên dịch chéo từ PC

- Thêm đường dẫn đến các file nhị phân của trình biên dịch tương ứng với nền tảng đích vào biến môi trường `PATH`.
- Đặt biến môi trường `ARCH` thành `arm`.
- Đặt biến môi trường `CROSS_COMPILE` thành tiền tố tương ứng với trình biên dịch cho nền tảng đích.

```

Ví dụ cho Raspberry Pi:

```bash
export PATH=[path]/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
```

Sau đó, từ cùng cửa sổ dòng lệnh nơi các biến môi trường trên đã được đặt, thực hiện:

make clean all

5. USB

Dự án này hỗ trợ cả giao diện SPI và USB. Đối với giao diện USB, board bộ tập trung có một vi điều khiển STM32 mà máy chủ Linux sẽ giao tiếp để cấu hình sx1302 và các radio liên quan. STM32 đóng vai trò như một cầu nối USB <-> SPI.

Vi điều khiển STM32 phải được lập trình với file nhị phân được cung cấp trong thư mục mcu_bin của dự án này. Để biết thêm chi tiết về cách nạp, vui lòng tham khảo hướng dẫn trong util_boot/readme.md.

Mọi tiện ích kiểm tra của dự án có thể được sử dụng với tùy chọn dòng lệnh -u -d /dev/ttyACMx, hoặc với cấu hình phù hợp trong file global_conf.json của bộ chuyển tiếp gói.

6. Thư viện bên thứ ba

Dự án này dựa trên một số thư viện mã nguồn mở bên thứ ba, có thể tìm thấy trong thư mục libtools:

· parson: trình phân tích cú pháp JSON (http://kgabis.github.com/parson/)
· tinymt32: bộ sinh số ngẫu nhiên giả (chỉ dùng cho debug/kiểm tra)

7. Nhật ký thay đổi (Changelog)

2.1.0-experimental-poc-hybrid-testing

Cập nhật

Phiên bản này là một đề xuất thử nghiệm cập nhật cho tiện ích kiểm tra test_loragw_hal_tx từ thư viện libloragw, cho phép thực hiện chuyển kênh ngẫu nhiên khi kiểm tra truyền gói của gateway.

Thay đổi

· TST: Thêm tùy chọn mới --hop cho tiện ích test_loragw_hal_tx để chỉ định rằng nó phải chọn ngẫu nhiên một kênh từ danh sách được cung cấp cho mỗi gói tin truyền. Tùy chọn này nhận một file JSON làm tham số để định nghĩa tần số kênh. Một file cấu hình JSON ví dụ được cung cấp, tên là channels_conf.json. Tiện ích cũng sẽ tránh lặp lại trên cùng một kênh hai lần. Tùy chọn này có thể được sử dụng để kiểm tra ở các khu vực cụ thể yêu cầu việc truyền phải được phân bố đều trên X kênh.

v2.1.0

Cập nhật

Bản phát hành này chỉ nhắm mục tiêu USB Corecell (không có thay đổi cho loại kết nối SPI). Firmware cầu nối USB-SPI, chạy trên vi điều khiển STM32 của USB Corecell, đã được cập nhật để dọn dẹp API và cải thiện độ mạnh mẽ.

Thay đổi

· MCU: Firmware cầu nối USB-SPI phiên bản v1.0.0.
  · Loại bỏ các lệnh lỗi thời (ORDER_ID__REQ_SPI, ORDER_ID__ACK_SPI)
  · Dịch chuyển chỉ số lệnh sau khi loại bỏ các lệnh lỗi thời (ORDER_ID__REQ_MULTIPLE_SPI, ORDER_ID__ACK_MULTIPLE_SPI)
  · Bộ phân tích cú pháp lệnh gửi ORDER_ID__UNKNOW_CMD trong trường hợp kích thước lệnh sai.
  · Dọn dẹp mã nguồn (sửa lỗi chính tả, thêm bình luận...)
  · Triển khai hàm Error_Handler() để đặt lại MCU trong trường hợp lỗi nghiêm trọng.
  · Sửa lỗi tràn tiềm ẩn trong hàm read_write_spi().
  · Tăng dung sai độ trễ cho phản hồi của máy chủ đối với truyền USB.
· HAL: Cập nhật giao diện lệnh cho firmware MCU v1.0.0.
  · Loại bỏ các lệnh lỗi thời khỏi enum order_id_e
  · Dịch chuyển chỉ số enum theo bản cập nhật cầu nối USB-SPI.
  · Loại bỏ hàm decode_ack_spi_access() không sử dụng.
· HAL: Thêm thông tin gỡ lỗi thời gian dưới DEBUG_MCU.

v2.0.2

Cập nhật

Sửa lỗi kiểm tra phiên bản firmware AGC cho các nền tảng dựa trên sx1255/sx1257 (gateway full-duplex...).

Thay đổi

· HAL: Phiên bản firmware AGC cho gateway dựa trên sx1255/sx1257 là v6.
· HAL: Thay đổi hình thức nhỏ và sửa lỗi chính tả.

v2.0.1

Cập nhật

Tính năng đóng dấu thời gian chính xác (fine timestamping) đã được xác nhận đầy đủ với bản phát hành này.

Thay đổi

· HAL: Điều chỉnh trường freq_offset của các gói nhận được, để tính đến lỗi phân giải IF của kênh.
· HAL: Tinh chỉnh độ lệch dấu thời gian chính xác so với Gateway v2, bằng cách tính đến độ lệch tần số của gói nhận được.
· HAL: Sửa lỗi độ dài preamble cho đường xuống FSK.
· MCU: Loại bỏ file nhị phân được biên dịch ở chế độ debug.
· util_spectral_scan: Thực sự sử dụng đối số đầu vào nb_scan mà trước đây đã bị bỏ qua.

v2.0.0

Tính năng mới

· Thêm hỗ trợ giao diện USB giữa HOST và bộ tập trung, chỉ dành cho bộ tập trung dựa trên sx1250.
· Thêm hỗ trợ Nghe-Trước-Khi-Nói (Listen-Before-Talk) cho vùng AS923, sử dụng radio sx1261 bổ sung từ thiết kế tham khảo Corecell v3 của Semtech.
· Thêm hỗ trợ Quét phổ (Spectral Scan) với radio sx1261 bổ sung từ thiết kế tham khảo Corecell v3 của Semtech.
· Thêm hỗ trợ cho chip SX1303, để hỗ trợ thêm tính năng Đóng dấu thời gian chính xác (Fine Timestamping).
· Hợp nhất nhánh master-fdd-cn490 để hỗ trợ thiết kế tham khảo Full-Duplex CN490. Đây là tích hợp của các bản phát hành v1.1.0, v1.1.1, v1.1.2 được mô tả bên dưới.

Thay đổi

· HAL: Thiết kế lại toàn bộ lớp giao tiếp. Một module loragw_com mới được giới thiệu để xử lý việc chuyển đổi giữa giao diện USB hoặc SPI, căn chỉnh nguyên mẫu hàm cho radio sx125x, sx1250 và sx1261. Đối với USB, một chế độ đã được thêm vào để nhóm các yêu cầu ghi SPI đến STM32 MCU, nhằm tối ưu hóa độ trễ trong các giai đoạn cấu hình quan trọng về thời gian.
· HAL: Thêm hỗ trợ sơ bộ cho Đóng dấu thời gian chính xác (Fine Timestamping) cho định vị TDOA.
· HAL: Cập nhật firmware AGC lên v6: thêm độ trễ cấu hình cho PA để khởi động, thêm hỗ trợ Nghe-Trước-Khi-Nói.
· HAL: Thêm hàm API mới lgw_demod_setconf() để thiết lập cài đặt bộ giải điều chế toàn cục.
· HAL: Thêm hàm API mới cho Quét phổ.
· Bộ chuyển tiếp gói: Loại giao diện có thể cấu hình trong file global_conf.json với com_type có thể là "USB" hoặc "SPI".
· Bộ chuyển tiếp gói: Thay đổi các tham số để cấu hình tính năng đóng dấu thời gian chính xác trong global_conf.json.
· Bộ chuyển tiếp gói: Thêm các phần để cấu hình tính năng quét phổ và Nghe-Trước-Khi-Nói.
· Bộ chuyển tiếp gói: Thêm một luồng mới cho ví dụ quét phổ nền, để chỉ ra cách sử dụng API quét phổ do HAL cung cấp, mà không can thiệp vào các tác vụ chính của gateway (nhận đường lên và truyền đường xuống).
· Bộ chuyển tiếp gói: Thêm phân tích cú pháp trường nhdr từ yêu cầu JSON đường xuống txpk để có thể gửi yêu cầu beacon từ Network Server.
· Bộ chuyển tiếp gói: Thêm chan_multiSF_All trong global_conf.json để chọn hệ số trải phổ nào cần kích hoạt cho các bộ giải điều chế đa SF.
· Bộ chuyển tiếp gói: Cập nhật PROTOCOL.md lên v1.6.
· Công cụ: thêm util_spectral_scan, một tiện ích quét phổ độc lập.

Ghi chú

· Bản phát hành này đã được xác nhận trên thiết kế tham khảo Corecell v3 của Semtech với giao diện USB.

v1.1.2 (Đã tích hợp vào v2.0.0 từ nhánh master-fdd-cn490)

· Bộ chuyển tiếp gói: cập nhật global_conf.json.sx1255.CN490.full-duplex với các hệ số bù nhiệt độ RSSI và cập nhật độ lệch RSSI cho radio 1.

v1.1.1 (Đã tích hợp vào v2.0.0 từ nhánh master-fdd-cn490)

· HAL: Cập nhật cấu hình bảng LUT LNA/PA SX1302 cho thiết kế tham khảo Full Duplex CN490.
· test_loragw_hal_rx/tx: thêm tùy chọn --fdd để kích hoạt Full Duplex.
· Bộ chuyển tiếp gói: cập nhật global_conf.json.sx1255.CN490.full-duplex cho thiết kế tham khảo CN490.

v1.1.0 (Đã tích hợp vào v2.0.0 từ nhánh master-fdd-cn490)

· HAL: Thêm hỗ trợ cho thiết kế tham khảo full duplex CN490.

v1.0.5

· HAL: Sửa lỗi dấu thời gian gói tin bị "nhảy cóc" trong các điều kiện đặc biệt.
· HAL: Giải pháp tạm thời cho vấn đề phần cứng khi đọc thanh ghi 32-bit (dấu thời gian, số byte trong bộ đệm RX...).
· HAL: Sửa lỗi vòng lặp vô hạn tiềm ẩn trong sx1302_tx_abort() nếu truy cập SPI thất bại.
· Bộ chuyển tiếp gói: Thêm global_conf.json.sx1250.US915 cho băng tần US915.
· test_hal_rx: thêm dòng lệnh để chỉ định độ lệch RSSI sẽ được áp dụng.

v1.0.4

· Thêm file LICENSE.TXT bị thiếu.
· HAL & Bộ chuyển tiếp gói: thêm hỗ trợ cho thiết kế tham khảo dựa trên sx1250 cho khu vực CN490.
· Bộ chuyển tiếp gói: vô hiệu hóa phát beacon theo mặc định.

v1.0.3

· HAL: Sửa lỗi độ chính xác thời gian đường xuống theo lịch trình bằng cách tính đến độ trễ bắt đầu truyền.
· HAL: Sửa lỗi tính toán hiệu chỉnh dấu thời gian cho BW250 & BW500.
· HAL: Sửa lỗi tràn bộ đệm tiềm ẩn trong hàm lgw_receive().
· HAL: Giữ lại gói nhận được trong bộ đệm RX khi bộ đệm được cấp để nhận gói quá nhỏ. Các gói còn lại sẽ được lấy trong các lần gọi lgw_receive tiếp theo (căn chỉnh theo hành vi của SX1301).

v1.0.2

· Sửa các cảnh báo biên dịch do các phiên bản GCC mới nhất báo cáo.
· Làm lại cách xử lý cảm biến nhiệt độ.
· Dọn dẹp các tệp không sử dụng.
· Thêm hướng dẫn và file cấu hình để tự động khởi động bộ chuyển tiếp gói với systemd.
· Thêm hiệu chuẩn radio SX1250 khi khởi động.

v1.0.1

· Bộ chuyển tiếp gói: Cập nhật bảng LUT độ lợi TX trong global_conf.json.sx1250 với hiệu chuẩn phù hợp.

v1.0.0

· HAL: Bản phát hành chính thức đầu tiên cho Thiết kế tham khảo CoreCell SX1302.

v0.0.1

· HAL: Bản phát hành nội bộ đầu tiên cho chương trình TAP.

8. Thông báo pháp lý

Thông tin được trình bày trong tài liệu dự án này không cấu thành bất kỳ một phần nào của bảng chào hàng hay hợp đồng, được tin là chính xác và đáng tin cậy và có thể thay đổi mà không cần thông báo trước. Nhà xuất bản sẽ không chịu bất kỳ trách nhiệm pháp lý nào đối với hậu quả của việc sử dụng nó. Việc xuất bản tài liệu này không bao hàm hay cấp bất kỳ giấy phép nào theo bằng sáng chế hoặc quyền sở hữu công nghiệp hoặc trí tuệ khác. Semtech không chịu bất kỳ trách nhiệm hoặc nghĩa vụ nào đối với bất kỳ sự cố hoặc hoạt động bất ngờ nào do lạm dụng, sơ suất, lắp đặt sai, sửa chữa hoặc xử lý không đúng cách, hoặc áp lực vật lý hoặc điện bất thường, bao gồm nhưng không giới hạn ở việc tiếp xúc với các thông số vượt quá định mức tối đa được chỉ định hoặc hoạt động ngoài phạm vi định mức.

SẢN PHẨM SEMTECH KHÔNG ĐƯỢC THIẾT KẾ, DỰ ĐỊNH, ỦY QUYỀN HOẶC BẢO HÀNH ĐỂ PHÙ HỢP CHO CÁC ỨNG DỤNG HỖ TRỢ SỰ SỐNG, THIẾT BỊ HOẶC HỆ THỐNG HOẶC CÁC ỨNG DỤNG QUAN TRỌNG KHÁC. VIỆC BAO GỒM SẢN PHẨM SEMTECH TRONG CÁC ỨNG DỤNG NHƯ VẬY ĐƯỢC HIỂU LÀ DO KHÁCH HÀNG TỰ CHỊU RỦI RO. Nếu khách hàng mua hoặc sử dụng sản phẩm Semtech cho bất kỳ ứng dụng trái phép nào như vậy, khách hàng sẽ bồi thường và giữ Semtech cùng các cán bộ, nhân viên, công ty con, chi nhánh và nhà phân phối của mình vô hại trước mọi khiếu nại, thiệt hại, chi phí và phí tổn của luật sư có thể phát sinh.

EOF

```

---

## 📝 Tổng kết nội dung chính bạn cần biết

| Thành phần | Chức năng |
|------------|-----------|
| **libloragw** | Thư viện lõi để giao tiếp với chip SX1302/SX1303 |
| **packet_forwarder** | Chương trình chính chuyển tiếp gói LoRa giữa gateway và server |
| **util_spectral_scan** | Quét phổ bằng radio sx1261 |
| **util_chip_id** | Đọc EUI của gateway |
| **util_boot** | Nạp firmware cho STM32 trên board USB |
| **util_net_downlink** | Mô phỏng server cho mục đích kiểm tra |

