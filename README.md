# BÀI TẬP SỐ 3: HỆ QUẢN TRỊ CSDL QUẢN LÝ CẦM ĐỒ

## THÔNG TIN SINH VIÊN:


**Họ Tên**: Nguyễn Văn Sang

**Lớp**: K59.KMT.K01

**Mssv**: K235480106060

--- 

# Phần 1: Thiết Kế CSDL 

**1**. Sơ đồ thực thể liên kết ERD.

<p><img width="1920" height="1019" alt="image" src="https://github.com/user-attachments/assets/43bfd3c0-4239-4231-abc1-63f5cd245812" />

<I>Hình 1: Sơ đồ thực thể liên kết ERD
</I></I></p>

**2**.bảng chuẩn hóa 3NF.

**chuẩn hóa thực thể hợp đồng**

* Mục tiêu: tách thông tin hợp đồng liên kết khách hàng qua khóa ngoại để tránh lặp lại thông tin khách hàng trên mỗi hợp đồng mới.

<p><img width="632" height="282" alt="image" src="https://github.com/user-attachments/assets/9edc1808-50f5-4bf3-ad0b-5f37b47693a8" />

Hình 2:chuẩn hóa thực thể hợp đồng

**chuẩn hóa thực thể tài sản**

* Mục tiêu: tách thông tin tài sản ra khỏi hợp đồng để giải quyết triệt để bài toán "1 hợp đồng có nhiều tài sản thế chấp" đảm bảo tính nguyên tử của dữ liệu (1NF).

<p><img width="466" height="211" alt="image" src="https://github.com/user-attachments/assets/1b98653c-315e-4fb8-b893-70c3b494162d" />
Hình 3:chuẩn hóa thực thể tài sản
</p>

**chuẩn hóa thực thể lịch sử biến động**

* Mục tiêu: tách thực thể giao dịch thành 1 thực thể riêng để tránh việc cập nhật đè trên bảng hợp đồng, đồng thời liên kết với người giao dịch.

<P><img width="481" height="243" alt="image" src="https://github.com/user-attachments/assets/3c98dd27-67ed-466d-b0d3-397eee78caaf" />
Hình 4: chuẩn hóa thực thể lịch sử biến động 
</P>

**chuẩn hóa thực thể nhân viên**

* Mục tiêu: quản lý thông tin người thu tiền độc lập, không ghi trực tiếp tên nhân viên vào bảng Log để tránh dư thừa thông tin và sai sót chính tả.

<p><img width="459" height="141" alt="image" src="https://github.com/user-attachments/assets/b4ef1b79-0521-4798-b1bd-82b83b4924cb" />
hình 5:chuẩn hóa thực thể nhân viên
</p>

**chuẩn hóa CSDL quản lý cầm đồ**

<p><img width="233" height="590" alt="image" src="https://github.com/user-attachments/assets/283f2f4d-32dc-4d8a-bd7f-c74d7085db50" />
Hình 6: chuẩn hóa CSDL quản lý cầm đồ
</p>

# phần 2: cài đặt SQL

**Event 1**:đăng kí hợp đồng mới(vay tiền)

```

USE QuanLyCamDo;
GO

CREATE PROCEDURE sp_DangKyHopDong
    @HoTen NVARCHAR(100),
    @SoDienThoai VARCHAR(15),
    @CCCD VARCHAR(12),
    @SoTienVay DECIMAL(18,2),
    @TenTS NVARCHAR(100),
    @DinhGia DECIMAL(18,2)
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        DECLARE @MaKH INT;
        
        -- 1. XỬ LÝ KHÁCH HÀNG (Kiểm tra xem đã tồn tại chưa)
        SELECT @MaKH = MaKH FROM KhachHang WHERE CCCD = @CCCD;
        IF @MaKH IS NULL
        BEGIN
            INSERT INTO KhachHang (HoTen, SoDienThoai, CCCD) 
            VALUES (@HoTen, @SoDienThoai, @CCCD);
            
            -- Lấy ID khách hàng vừa được tạo tự động
            SET @MaKH = SCOPE_IDENTITY();
        END

        -- 2. TÍNH TOÁN DEADLINE (Giả định Deadline 1 là 30 ngày, Deadline 2 là 60 ngày)
        DECLARE @Deadline1 DATETIME = DATEADD(day, 30, GETDATE());
        DECLARE @Deadline2 DATETIME = DATEADD(day, 60, GETDATE());

        -- 3. XỬ LÝ HỢP ĐỒNG
        DECLARE @MaHD INT;
        INSERT INTO HopDong (MaKH, NgayVay, SoTienVayGoc, Deadline1, Deadline2, TrangThai)
        VALUES (@MaKH, GETDATE(), @SoTienVay, @Deadline1, @Deadline2, N'Đang vay');
        
        -- Lấy ID hợp đồng vừa được tạo tự động
        SET @MaHD = SCOPE_IDENTITY();

        -- 4. XỬ LÝ TÀI SẢN
        INSERT INTO TaiSan (MaHD, TenTaiSan, GiaTriDinhGia, TrangThaiTS)
        VALUES (@MaHD, @TenTS, @DinhGia, N'Đang cầm cố');

        -- Xác nhận lưu dữ liệu vĩnh viễn
        COMMIT TRANSACTION;
        PRINT N'Đăng ký hợp đồng thành công!';
        
    END TRY
    BEGIN CATCH
        -- Hoàn tác nếu có bất kỳ lỗi nào xảy ra
        ROLLBACK TRANSACTION;
        PRINT N'Lỗi giao dịch: ' + ERROR_MESSAGE();
    END CATCH
END;
GO

```
