# Hướng dẫn sửa lỗi DirectX Runtime khi mở Valorant

Script `check-directx.ps1` chạy trực tiếp từ GitHub, không cần tải về máy. Cập nhật: 16/10/2026.

## Dấu hiệu và nguyên nhân

Bấm PLAY trong Riot Client thì hiện hộp thoại "The following component(s) are required to run this program: DirectX Runtime", dù đã cài lại game và DirectX.

Nguyên nhân thường gặp là một file DLL của DirectX cũ trong `C:\Windows\System32` hoặc `SysWOW64` đã bị phần mềm khác ghi đè. File vẫn mang chữ ký Microsoft nhưng nội dung đã bị sửa, nên kiểm tra chữ ký báo `HashMismatch`. Game từ chối file này và báo thiếu DirectX.

Cài lại DirectX không sửa được, vì bộ cài thấy file "mới hơn" nên bỏ qua. Phải đổi tên file lỗi trước rồi mới cài lại.

## Chuẩn bị

1. Tải `directx_Jun2010_redist.exe` (DirectX End-User Runtimes June 2010) từ trang Microsoft. 
```
https://www.microsoft.com/en-us/download/details.aspx?id=8109
```
2. Chạy file vừa tải, ở hộp thoại hỏi nơi giải nén thì gõ `C:\DX`, bấm OK, chọn Yes nếu được hỏi tạo thư mục. Kết quả phải có file `C:\DX\DXSETUP.exe`.
3. Mở PowerShell bằng quyền admin: nhấn Start, gõ `powershell`, chuột phải chọn **Run as administrator**.

## Bước 1: Chạy kiểm tra

Dán lệnh sau vào PowerShell. Lệnh này chỉ kiểm tra, chưa thay đổi gì trên máy:

```
irm https://raw.githubusercontent.com/dat1799/Check-Valorant-DirectX-Runtime/main/check-directx.ps1 | iex
```

Script quét chữ ký mọi file DLL trong System32 và SysWOW64, mất khoảng 5 đến 15 phút. Thanh tiến trình hiện ở đầu cửa sổ. Khi xong, script lưu hai file CSV ra Desktop: `dll-check-all-<tên máy>.csv` (toàn bộ) và `dll-check-problems-<tên máy>.csv` (chỉ file có vấn đề).

## Bước 2: Đọc kết quả

Chỉ nhóm NGUY HIỂM cần sửa ngay; nếu nhóm này bằng 0 và hai file Valorant đều OK, lỗi không nằm ở file DLL.

| Mục trong kết quả | Ý nghĩa | Cần làm |
| --- | --- | --- |
| NGUY HIỂM (đỏ) | File bị sửa (`HashMismatch`) hoặc chữ ký không hợp lệ | Sửa ở Bước 3 |
| CẦN XEM LẠI (vàng) | File không có chữ ký | Xem tên và ngày sửa; thường là phần mềm cũ, không bắt buộc sửa |
| THÔNG TIN (xám) | File có chữ ký hợp lệ của hãng khác (NVIDIA, Intel, Realtek...) | Bình thường, bỏ qua |
| File Valorant cần: [THIEU] hoặc [LOI] | Thiếu hoặc hỏng `xinput1_3.dll` / `X3DAudio1_7.dll` | Sửa ở Bước 3 |
| VC++ x64 / x86: [THIEU] | Chưa cài Visual C++ 2015-2022 | Cài cả hai bản x64 và x86 từ trang Microsoft |

Ghi lại ngày sửa (cột Modified) của file bị sửa. Ngày đó giúp tìm phần mềm đã ghi đè file.

## Bước 3: Sửa tự động

1. Quét virus trước: Windows Security > Virus & threat protection > Scan options > **Microsoft Defender Antivirus (offline scan)** > Scan now. Máy tự khởi động lại và quét khoảng 15 phút.
2. Mở lại PowerShell quyền admin và dán lệnh:

```
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/dat1799/Check-Valorant-DirectX-Runtime/main/check-directx.ps1))) -Fix -DxSetup "C:\DX\DXSETUP.exe"
```

3. Với mỗi file DirectX bị sửa, script hỏi `Doi ten ...? (y/n)`. Gõ `y` rồi Enter.
4. Script tự đổi tên file thành `.bak`, cài lại DirectX ở chế độ ẩn, rồi kiểm tra lại. Mỗi file phải báo `[OK]`.
5. Nếu có file Windows bị sửa, script hỏi có chạy `sfc` và `DISM` không. Gõ `y`; bước này mất 10 đến 30 phút.
6. Khởi động lại máy.

Sửa nhiều máy một lúc thì thêm `-Force` để script làm hết không hỏi:

```
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/dat1799/Check-Valorant-DirectX-Runtime/main/check-directx.ps1))) -Fix -Force -DxSetup "C:\DX\DXSETUP.exe"
```

Script chỉ đổi tên file DirectX cũ. File của Windows được khôi phục bằng `sfc` và `DISM`; file không chữ ký hoặc của hãng khác không bị động đến.

## Bước 4: Sửa thủ công (khi script báo THAT BAI)

Ví dụ dưới đây dùng `X3DAudio1_7.dll`; thay bằng tên file script báo lỗi.

1. Mở Command Prompt quyền admin (Start, gõ `cmd`, chuột phải > Run as administrator).
2. Đổi tên file:

```
ren C:\Windows\System32\X3DAudio1_7.dll X3DAudio1_7.dll.bak
```

3. Nếu báo `Access is denied`, lấy quyền rồi chạy lại lệnh `ren`:

```
takeown /f C:\Windows\System32\X3DAudio1_7.dll /a
icacls C:\Windows\System32\X3DAudio1_7.dll /grant *S-1-5-32-544:F
```

4. Vào `C:\DX`, chuột phải `DXSETUP.exe` > Run as administrator, cài như bình thường.
5. Kiểm tra lại trong PowerShell, dòng `Status` phải là `Valid`:

```
Get-AuthenticodeSignature C:\Windows\System32\X3DAudio1_7.dll | fl
```

File nằm trong `SysWOW64` thì làm y hệt, chỉ đổi đường dẫn.

## Bước 5: Kiểm tra lại và dọn dẹp

1. Chạy lại lệnh ở Bước 1. Nhóm NGUY HIỂM phải bằng 0, hai file Valorant phải báo `[OK]`.
2. Mở Riot Client và bấm PLAY.
3. Game chạy được thì xóa các file `.bak` (danh sách trong `dll-renamed-<tên máy>.csv` trên Desktop).
4. Tìm phần mềm đã ghi đè file để gỡ, tránh lỗi quay lại. Lệnh dưới liệt kê phần mềm cài vào ngày sửa file (thay `20261002` bằng ngày ghi ở Bước 2, dạng năm-tháng-ngày):

```
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*, HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*, HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* -ErrorAction SilentlyContinue | Where-Object { $_.InstallDate -eq '20261002' } | Select-Object DisplayName, Publisher, InstallDate
```

Nếu không thấy gì đáng ngờ, nguyên nhân thường là chương trình chạy trực tiếp không cần cài (bản crack, mod, công cụ tải về). Xem thư mục Downloads sắp theo ngày.

**Khôi phục:** nếu sau khi sửa có phần mềm bị lỗi, đổi file `.bak` tương ứng về tên cũ.

## Xử lý sự cố thường gặp

| Hiện tượng | Cách xử lý |
| --- | --- |
| `irm` báo 404 hoặc không tải được | Kiểm tra repo đang Public, file `check-directx.ps1` nằm ở thư mục gốc, nhánh là `main` |
| `Che do -Fix can quyen admin` | Mở lại PowerShell bằng Run as administrator |
| Sau DXSETUP vẫn báo `[THIEU]` | Kiểm tra lại `E:\DX\DXSETUP.exe` có tồn tại; chạy DXSETUP thủ công bằng quyền admin |
| Sau sửa vẫn báo `HashMismatch` | Có phần mềm đang ghi đè lại file: quét virus, gỡ phần mềm nghi ngờ (Bước 5), rồi sửa lại |
| Không có file lỗi nhưng game vẫn báo DirectX Runtime | Cài Visual C++ x64 và x86, cập nhật driver card đồ họa, Repair game trong Riot Client |
| Vẫn không được | Dùng Process Monitor (lọc Process Name chứa VALORANT) để xem file game tìm trước khi báo lỗi, hoặc gửi hai file CSV cho hỗ trợ Riot |
