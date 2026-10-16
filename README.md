# Fix Valorant DirectX Runtime

Sửa lỗi bấm PLAY trong Riot Client thì hiện thông báo "The following component(s) are required to run this program: DirectX Runtime". Cập nhật: 16/10/2026.

## Cách dùng (3 bước)

1. Nhấn **Start**, gõ `powershell`, chuột phải vào **Windows PowerShell** và chọn **Run as administrator**.
2. Dán lệnh sau vào cửa sổ PowerShell rồi bấm Enter:

```
irm "https://gist.githubusercontent.com/dat1799/150903857de1134ffc2b4e8ba6c75e2e/raw/check-directx.ps1?$(Get-Random)" | iex
```

3. Menu hiện ra. Gõ **1** rồi Enter để sửa lỗi tự động, gõ `y` khi được hỏi, chờ chạy xong rồi **khởi động lại máy** và mở Valorant.

Nếu quên chọn Run as administrator, công cụ sẽ hỏi để tự mở lại với quyền admin; bấm `y` rồi chọn **Yes** ở hộp thoại Windows.

## Các chức năng trong menu

| Số | Chức năng | Khi nào dùng |
| --- | --- | --- |
| 1 | Sửa lỗi tự động | Dùng đầu tiên. Kiểm tra file DirectX, đổi tên file lỗi, tự tải và cài lại DirectX gốc, cài Visual C++ nếu thiếu |
| 2 | Kiểm tra nhanh (~1 phút) | Chỉ muốn xem máy có lỗi không, chưa sửa gì |
| 3 | Kiểm tra toàn bộ DLL (5-15 phút) | Sửa xong vẫn lỗi, muốn quét mọi file DLL trong Windows |
| 4 | Sửa file DirectX bị lỗi | Giống bước sửa trong chức năng 1, có hỏi xác nhận |
| 5 | Cài / sửa Visual C++ 2015-2022 | Kết quả báo thiếu Visual C++ |
| 6 | Quét virus bằng Windows Defender | Có file bị sửa, nên quét để chắc máy sạch. Quét offline sẽ khởi động lại máy |
| 7 | Tìm phần mềm nghi đã ghi đè file | Muốn biết phần mềm nào gây lỗi để gỡ, tránh lỗi quay lại |
| 8 | Quản lý file .bak | Game chạy được rồi thì xóa file .bak; có phần mềm bị lỗi sau khi sửa thì khôi phục |
| 9 | Sửa file hệ thống Windows (10-30 phút) | Kiểm tra toàn bộ (3) báo có file Windows bị sửa |
| 0 | Thoát | Thoát và tự xóa file tải về tạm |

## Nguyên nhân lỗi

Một file DLL của DirectX cũ trong `C:\Windows\System32` hoặc `SysWOW64` (thường là `X3DAudio1_7.dll`) đã bị phần mềm khác ghi đè. File vẫn mang chữ ký Microsoft nhưng nội dung bị sửa, nên game từ chối và báo thiếu DirectX.

Cài lại DirectX bằng cách thông thường không sửa được, vì bộ cài thấy file "mới hơn" nên bỏ qua. Công cụ đổi tên file lỗi thành `.bak` trước rồi mới cài lại, nên file gốc được ghi vào đúng chỗ.

## Đọc kết quả

| Nhãn | Ý nghĩa |
| --- | --- |
| `[OK]` | File hợp lệ, có chữ ký Microsoft |
| `[LOI]` | File bị sửa hoặc chữ ký không hợp lệ, cần sửa |
| `[THIEU]` | File hoặc Visual C++ chưa được cài |
| `[NGUY HIEM]` | (Kiểm tra toàn bộ) file bị sửa, cần sửa ngay |
| `[CAN XEM LAI]` | (Kiểm tra toàn bộ) file không có chữ ký, thường là phần mềm cũ, không bắt buộc sửa |
| `[THONG TIN]` | (Kiểm tra toàn bộ) file của hãng khác như NVIDIA, Intel, bình thường |

## Xử lý sự cố

| Hiện tượng | Cách xử lý |
| --- | --- |
| `irm` báo 404 hoặc không tải được | Kiểm tra kết nối mạng; chép lại đúng nguyên lệnh ở bước 2 |
| Cửa sổ PowerShell tự tắt | Mở PowerShell trước rồi mới dán lệnh, không chạy lệnh qua Win + R |
| Không tự tải được DirectX | Tải `directx_Jun2010_redist.exe` từ trang Microsoft, giải nén ra một thư mục (vd `E:\DX`), rồi nhập đường dẫn `E:\DX\DXSETUP.exe` khi công cụ hỏi |
| Quét virus báo không dùng được Defender | Máy đang dùng phần mềm diệt virus khác; quét bằng phần mềm đó |
| Sửa xong vẫn báo `[LOI]` | Có phần mềm đang ghi đè lại file: quét virus (6), tìm và gỡ phần mềm nghi ngờ (7), rồi chạy lại chức năng 1 |
| Không có file lỗi nhưng game vẫn báo DirectX Runtime | Cập nhật driver card đồ họa, Repair game trong Riot Client, rồi chụp màn hình kết quả gửi hỗ trợ Riot |

## Lưu ý an toàn

Công cụ chạy bằng quyền admin và chỉ tải DirectX, Visual C++ từ trang chính thức của Microsoft. Nó chỉ đổi tên các file DirectX cũ (có thể khôi phục ở chức năng 8); file của Windows được sửa bằng công cụ có sẵn `sfc` và `DISM`, file của hãng khác không bị động đến.

Link công cụ nằm trong một secret gist: không ai tìm thấy nếu không có link, nhưng ai có link đều chạy và xem được, nên chỉ gửi cho người cần dùng.
