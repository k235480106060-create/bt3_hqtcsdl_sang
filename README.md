# BÀI TẬP SỐ 3: HỆ QUẢN TRỊ CSDL QUẢN LÝ CẦM ĐỒ

## THÔNG TIN SINH VIÊN:


**Họ Tên**: Nguyễn Văn Sang

**Lớp**: K59.KMT.K01

**Mssv**: K235480106060

**Giảng viên hướng dẫn**: Đỗ Duy Cốp

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

<p><img width="1920" height="1022" alt="image" src="https://github.com/user-attachments/assets/bc76204c-3fdc-456b-84ef-f1495a27f89e" />
hình 7: thi hành lệnh thành công
</p>

* Mô tả:

Tính toàn vẹn dữ liệu: Dùng BEGIN TRANSACTION và TRY...CATCH để hoàn tác (rollback) toàn bộ nếu có lỗi, ngăn chặn dữ liệu "nửa vời".

Quản lý khách hàng: Tự động nhận diện khách quen qua CCCD để tái sử dụng ID, tránh trùng lặp bản ghi trong cơ sở dữ liệu.

Thiết lập thời hạn: Tự động tính toán Deadline1 và Deadline2 từ ngày lập hợp đồng để làm căn cứ chuẩn xác cho việc tính lãi sau này.

Liên kết thực thể: Dùng SCOPE_IDENTITY() lấy ngay Khóa chính vừa tạo để nạp vào Khóa ngoại, móc xích chặt chẽ chuỗi Khách hàng – Hợp đồng – Tài sản. 

**Event 2**: Tính toán công nợ thời gian thực.

**Function 1**: fn_CalcMoneyTransaction

```

USE QuanLyCamDo;
GO

CREATE FUNCTION fn_CalcMoneyTransaction
(
    @TransactionID INT, -- Tương đương với Mã Hợp Đồng (MaHD)
    @TargetDate DATETIME
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @TongTien DECIMAL(18,2) = 0;
    DECLARE @TienGoc DECIMAL(18,2);
    DECLARE @NgayVay DATETIME;
    DECLARE @Deadline1 DATETIME;
    
    DECLARE @SoNgayLaiDon INT = 0;
    DECLARE @SoNgayLaiKep INT = 0;

    -- BƯỚC 1: TRUY XUẤT DỮ LIỆU KHOẢN VAY
    SELECT 
        @TienGoc = SoTienVayGoc,
        @NgayVay = NgayVay,
        @Deadline1 = Deadline1
    FROM HopDong 
    WHERE MaHD = @TransactionID;

    -- Bắt lỗi nếu mã hợp đồng không tồn tại
    IF @TienGoc IS NULL RETURN 0;

    -- BƯỚC 2: TÍNH TOÁN CÔNG NỢ DỰA TRÊN THỜI GIAN
    IF (@TargetDate <= @Deadline1)
    BEGIN
        -- TRƯỜNG HỢP 1: Lãi đơn (Trước hoặc đúng Deadline 1)
        SET @SoNgayLaiDon = DATEDIFF(day, @NgayVay, @TargetDate);
        
        -- Tránh trường hợp TargetDate nhập vào nhỏ hơn NgayVay
        IF @SoNgayLaiDon < 0 SET @SoNgayLaiDon = 0; 
        
        -- Tính tổng tiền = Gốc + Lãi đơn
        SET @TongTien = @TienGoc + (@TienGoc * 0.005 * @SoNgayLaiDon);
    END
    ELSE 
    BEGIN
        -- TRƯỜNG HỢP 2: Lãi kép (Sau Deadline 1)
        -- Chốt số ngày tính lãi đơn (Từ ngày vay đến Deadline 1)
        SET @SoNgayLaiDon = DATEDIFF(day, @NgayVay, @Deadline1);
        
        -- Số ngày tính lãi kép (Từ Deadline 1 đến TargetDate)
        SET @SoNgayLaiKep = DATEDIFF(day, @Deadline1, @TargetDate);

        -- Tính số tiền chốt tại Deadline 1 (Bao gồm Gốc và Lãi đơn)
        DECLARE @TienChotDeadline1 DECIMAL(18,2);
        SET @TienChotDeadline1 = @TienGoc + (@TienGoc * 0.005 * @SoNgayLaiDon);

        -- Tính Lãi kép cộng dồn dựa trên số tiền chốt Deadline 1
        SET @TongTien = @TienChotDeadline1 * POWER(1.005, @SoNgayLaiKep);
    END

    -- BƯỚC 3: TRẢ VỀ KẾT QUẢ CUỐI CÙNG
    RETURN @TongTien;
END;
GO

```

<p><img width="1920" height="984" alt="image" src="https://github.com/user-attachments/assets/c1bc4876-0d29-4367-9d6a-53c5576189c3" />
Hình 8:thi hành lệnh thành công
</p>
* Mô tả:

@TransactionID (INT): Dùng để xác định khoản vay nào cần tính toán (truy xuất vào cột MaHD).

@TargetDate (DATETIME): Cột mốc thời gian bạn muốn chốt sổ để tính nợ.

RETURNS DECIMAL(18,2): Hàm trả về một giá trị số thực có 2 chữ số thập phân, đại diện cho tổng số tiền khách hàng nợ (chưa trừ đi số tiền đã trả trước đó).

**Function 2**:fn_CalcMoneyContract

```

USE QuanLyCamDo;
GO

CREATE FUNCTION fn_CalcMoneyContract
(
    @ContractID INT,      -- Mã hợp đồng (MaHD)
    @TargetDate DATETIME  -- Mốc thời gian muốn chốt công nợ
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    -- Khai báo các biến lưu trữ dòng tiền
    DECLARE @TongNoPhatSinh DECIMAL(18,2) = 0;
    DECLARE @TongDaTra DECIMAL(18,2) = 0;
    DECLARE @DuNoHienTai DECIMAL(18,2) = 0;

    -- Khai báo các biến lấy từ hợp đồng
    DECLARE @TienGoc DECIMAL(18,2);
    DECLARE @NgayVay DATETIME;
    DECLARE @Deadline1 DATETIME;
    
    -- Khai báo biến đếm thời gian
    DECLARE @SoNgayLaiDon INT = 0;
    DECLARE @SoNgayLaiKep INT = 0;

    -- =================================================================
    -- BƯỚC 1: LẤY THÔNG TIN GỐC CỦA HỢP ĐỒNG
    -- =================================================================
    SELECT 
        @TienGoc = SoTienVayGoc,
        @NgayVay = NgayVay,
        @Deadline1 = Deadline1
    FROM HopDong 
    WHERE MaHD = @ContractID;

    -- Bắt lỗi nếu mã hợp đồng không tồn tại
    IF @TienGoc IS NULL RETURN 0;

    -- =================================================================
    -- BƯỚC 2: TÍNH TỔNG NỢ LÝ THUYẾT (GỐC + LÃI ĐƠN + LÃI KÉP)
    -- =================================================================
    IF (@TargetDate <= @Deadline1)
    BEGIN
        -- Giai đoạn Lãi đơn (0.5% / ngày)
        SET @SoNgayLaiDon = DATEDIFF(day, @NgayVay, @TargetDate);
        IF @SoNgayLaiDon < 0 SET @SoNgayLaiDon = 0; 
        
        SET @TongNoPhatSinh = @TienGoc + (@TienGoc * 0.005 * @SoNgayLaiDon);
    END
    ELSE 
    BEGIN
        -- Giai đoạn Lãi kép
        SET @SoNgayLaiDon = DATEDIFF(day, @NgayVay, @Deadline1);
        SET @SoNgayLaiKep = DATEDIFF(day, @Deadline1, @TargetDate);

        -- Chốt số tiền (Gốc + Lãi đơn) tại mốc Deadline 1
        DECLARE @TienChotDeadline1 DECIMAL(18,2);
        SET @TienChotDeadline1 = @TienGoc + (@TienGoc * 0.005 * @SoNgayLaiDon);

        -- Áp dụng hàm POWER để tính lãi kép cộng dồn từ Deadline 1 đến TargetDate
        SET @TongNoPhatSinh = @TienChotDeadline1 * POWER(1.005, @SoNgayLaiKep);
    END

    -- =================================================================
    -- BƯỚC 3: ĐỐI TRỪ VỚI LỊCH SỬ THANH TOÁN (AUDIT LOG)
    -- =================================================================
    -- Quét toàn bộ bảng LogBienDong để xem khách đã trả bao nhiêu tiền cho hợp đồng này
    -- Dùng hàm ISNULL để phòng trường hợp khách chưa trả đồng nào (tránh lỗi NULL)
    SELECT @TongDaTra = ISNULL(SUM(SoTienTra), 0)
    FROM LogBienDong
    WHERE MaHD = @ContractID AND NgayGiaoDich <= @TargetDate;

    -- =================================================================
    -- BƯỚC 4: TÍNH TOÁN DƯ NỢ CUỐI CÙNG
    -- =================================================================
    SET @DuNoHienTai = @TongNoPhatSinh - @TongDaTra;
    
    -- Xử lý ngoại lệ: Nếu khách trả dư tiền thì dư nợ bằng 0
    IF @DuNoHienTai < 0 
        SET @DuNoHienTai = 0;

    RETURN @DuNoHienTai;
END;
GO

```

<p><img width="1919" height="1023" alt="image" src="https://github.com/user-attachments/assets/d3642bc4-7d3f-4c98-b91d-c2220ec4cd2c" />
Hình 9: lệnh thành công
</p>

* MÔ tả :

Tính tổng nợ lý thuyết: Hàm tự động kiểm tra mốc thời gian để áp dụng đúng công thức. Dùng phép nhân cơ bản cho giai đoạn lãi đơn (trước Deadline 1) và dùng hàm POWER lũy thừa để tính lãi kép cộng dồn (sau Deadline 1).

Đối trừ lịch sử trả góp: Lệnh SUM() quét toàn bộ bảng LogBienDong để gom các khoản khách đã trả lẻ tẻ. Hàm ISNULL bọc bên ngoài giúp hệ thống không bị lỗi báo rỗng (NULL) nếu khách chưa từng trả đồng nào.

Chốt dư nợ thực tế: Lấy (Tổng nợ lý thuyết) trừ đi (Tổng tiền đã trả). Nếu kết quả bị âm (do khách trả dư tiền), hệ thống sẽ tự động điều chỉnh dư nợ về 0.

Kiểm tra xem hai hàm fn_CalcMoneyTransaction và fn_CalcMoneyContract hoạt động không:

```

USE QuanLyCamDo;
GO

DECLARE @MaHD_Test INT = 1; -- Nhập Mã hợp đồng cần test vào đây
DECLARE @NgayTest DATETIME = GETDATE(); -- Mặc định test ngày hôm nay. Có thể sửa thành ngày quá hạn ví dụ: '2026-08-15'

SELECT 
    @MaHD_Test AS MaHopDong,
    @NgayTest AS NgayChotNo,
    -- Hàm 1: Chỉ tính tổng (Gốc + Lãi)
    dbo.fn_CalcMoneyTransaction(@MaHD_Test, @NgayTest) AS TongNoLyThuyet,
    -- Hàm 2: Dư nợ thực tế (Tổng nợ lý thuyết - Các khoản khách đã trả)
    dbo.fn_CalcMoneyContract(@MaHD_Test, @NgayTest) AS DuNoThucTe;

```

<p><img width="1920" height="982" alt="image" src="https://github.com/user-attachments/assets/eb09ffaf-e071-4b96-af65-11ca1298a599" />
hình 10: kiểm tra thành công
</p>

**Event 3**: Xử lý trả nợ và hoàn trả tài sản

```

USE QuanLyCamDo;
GO

CREATE OR ALTER PROCEDURE sp_XuLyThanhToan
    @MaHD INT,
    @MaNV INT, -- Cần có mã nhân viên thu tiền để ghi vào Log
    @SoTienKhachTra DECIMAL(18,2)
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Khai báo các biến cần thiết
        DECLARE @TrangThaiHD NVARCHAR(50);
        DECLARE @DuNoHienTai DECIMAL(18,2);
        DECLARE @DuNoMoi DECIMAL(18,2);
        DECLARE @TongGiaTriTS DECIMAL(18,2);

        -- =================================================================================
        -- Ý NHỎ 1: KIỂM TRA CHẶN LỖI (TÀI SẢN ĐÃ THANH LÝ)
        -- =================================================================================
        SELECT @TrangThaiHD = TrangThai FROM HopDong WHERE MaHD = @MaHD;
        
        -- Nếu hợp đồng đã đánh cờ thanh lý (IsSold) hoặc nợ xấu quá hạn không thể cứu vãn
        IF (@TrangThaiHD = N'Đã thanh lý' OR @TrangThaiHD = N'Đã bán thanh lý')
        BEGIN
            PRINT N'Tài sản đã bị thanh lý (IsSold). Không thu tiền, không trả đồ!';
            ROLLBACK TRANSACTION; -- Hủy giao dịch ngay lập tức
            RETURN; -- Thoát khỏi Procedure
        END

        -- =================================================================================
        -- Ý NHỎ 2: XỬ LÝ DÒNG TIỀN VÀ TRẠNG THÁI HỢP ĐỒNG
        -- =================================================================================
        -- Gọi Function ở Event 2 để xem khách đang nợ tổng cộng bao nhiêu
        SET @DuNoHienTai = dbo.fn_CalcMoneyContract(@MaHD, GETDATE());

        IF (@SoTienKhachTra >= @DuNoHienTai)
        BEGIN
            -- KỊCH BẢN 2A: KHÁCH TRẢ HẾT NỢ
            -- 1. Cập nhật hợp đồng
            UPDATE HopDong SET TrangThai = N'Đã thanh toán đủ' WHERE MaHD = @MaHD;
            
            -- 2. Trả lại toàn bộ tài sản
            UPDATE TaiSan SET TrangThaiTS = N'Đã trả khách' WHERE MaHD = @MaHD;
            
            -- 3. Ghi Log số tiền thực thu (Chỉ thu bằng đúng số nợ, tiền thừa trả lại khách)
            INSERT INTO LogBienDong (MaHD, MaNV, NgayGiaoDich, SoTienTra, NoiDung)
            VALUES (@MaHD, @MaNV, GETDATE(), @DuNoHienTai, N'Tất toán hợp đồng');
            
            PRINT N'Khách đã thanh toán đủ. Đã cập nhật trạng thái và trả toàn bộ tài sản.';
        END
        ELSE
        BEGIN
            -- KỊCH BẢN 2B: KHÁCH TRẢ GÓP (TRẢ MỘT PHẦN)
            -- 1. Cập nhật trạng thái
            UPDATE HopDong SET TrangThai = N'Đang trả góp' WHERE MaHD = @MaHD;
            
            -- 2. Ghi Log số tiền khách nộp vào
            INSERT INTO LogBienDong (MaHD, MaNV, NgayGiaoDich, SoTienTra, NoiDung)
            VALUES (@MaHD, @MaNV, GETDATE(), @SoTienKhachTra, N'Khách thanh toán trả góp');
            
            -- Tính số nợ còn lại sau khi trừ tiền vừa trả
            SET @DuNoMoi = @DuNoHienTai - @SoTienKhachTra;
            PRINT N'Đã ghi nhận trả góp. Dư nợ còn lại: ' + CAST(@DuNoMoi AS NVARCHAR);

            -- =================================================================================
            -- Ý NHỎ 3: ĐƯA RA DANH SÁCH GỢI Ý TRẢ ĐỒ (Chỉ áp dụng khi trả góp)
            -- =================================================================================
            -- Tính tổng giá trị các tài sản đang bị tiệm cầm đồ giữ
            SELECT @TongGiaTriTS = ISNULL(SUM(GiaTriDinhGia), 0) 
            FROM TaiSan 
            WHERE MaHD = @MaHD AND TrangThaiTS = N'Đang cầm cố';

            PRINT N'--- DANH SÁCH TÀI SẢN CÓ THỂ LẤY VỀ ---';
            -- Truy vấn những món đồ thỏa mãn: (Tổng giá trị TS - Giá trị món đồ đó) >= Dư nợ mới
            SELECT 
                MaTS, 
                TenTaiSan, 
                GiaTriDinhGia,
                (@TongGiaTriTS - GiaTriDinhGia) AS GiaTriTaiSanGiuLai,
                @DuNoMoi AS DuNoHienTai,
                N'Đủ điều kiện lấy về' AS GhiChu
            FROM TaiSan
            WHERE MaHD = @MaHD 
              AND TrangThaiTS = N'Đang cầm cố'
              AND (@TongGiaTriTS - GiaTriDinhGia) >= @DuNoMoi;
        END

        -- Chốt giao dịch thành công
        COMMIT TRANSACTION;
        
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        PRINT N'Lỗi hệ thống: ' + ERROR_MESSAGE();
    END CATCH
END;
GO

```

<p><img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9457622f-dd74-448f-a5e0-da73abc6c90b" />
hình 11:thi hành lệnh thành công
</p>

* Mô tả:

Ý nhỏ 1 (Chốt chặn an ninh): Đây là bước kiểm tra (Validation) đầu tiên và quan trọng nhất. Nếu cửa hàng đã bán thanh lý đồ của khách, lệnh ROLLBACK và RETURN sẽ chém đứt quy trình ngay lập tức, từ chối việc thu tiền để tránh kiện cáo lùm xùm.

Ý nhỏ 2 (Chia nhánh rẽ): * Việc bạn có hàm fn_CalcMoneyContract từ trước giúp bước này cực kỳ nhàn. Bạn chỉ cần lấy số tiền khách đưa so sánh với kết quả của hàm.

Lưu ý nghiệp vụ: Ở kịch bản trả hết nợ, hàm INSERT INTO LogBienDong chỉ ghi nhận số tiền bằng với @DuNoHienTai. Ví dụ khách nợ 9 triệu, đưa 10 triệu, hệ thống chỉ ghi nhận doanh thu 9 triệu và thối lại 1 triệu cho khách.

Ý nhỏ 3 (Thuật toán gợi ý trả đồ): * Quy tắc: Khách chỉ lấy được đồ A về nếu phần tài sản B, C... còn lại ở quán phải đủ để gánh số nợ.

Cách giải quyết: Tôi dùng lệnh SUM(GiaTriDinhGia) để lấy tổng giá trị kho. Ở câu SELECT cuối, mệnh đề WHERE (@TongGiaTriTS - GiaTriDinhGia) >= @DuNoMoi sẽ tự động lọc ra những món đồ nào mà khi rút nó ra khỏi kho, giá trị kho vẫn an toàn lớn hơn dư nợ.

**Event 4**: Truy vấn danh sách nợ xấu (Nợ khó đòi) 

```

USE QuanLyCamDo;
GO

-- Tạo View hoặc chỉ chạy lệnh SELECT để xuất danh sách
SELECT 
    KH.HoTen AS [Tên khách hàng],
    KH.SoDienThoai AS [Số điện thoại],
    HD.SoTienVayGoc AS [Số tiền vay gốc],
    
    -- Tính số ngày quá hạn kể từ Deadline 1
    DATEDIFF(DAY, HD.Deadline1, GETDATE()) AS [Số ngày quá hạn],
    
    -- Tổng tiền phải trả tính đến thời điểm hiện tại
    dbo.fn_CalcMoneyContract(HD.MaHD, GETDATE()) AS [Tổng tiền hiện tại],
    
    -- Dự báo tổng tiền phải trả sau 1 tháng nữa (Sử dụng DATEADD)
    dbo.fn_CalcMoneyContract(HD.MaHD, DATEADD(MONTH, 1, GETDATE())) AS [Dự báo nợ sau 1 tháng]

FROM HopDong HD
JOIN KhachHang KH ON HD.MaKH = KH.MaKH
WHERE 
    -- Điều kiện 1: Đã vượt quá Deadline 1
    GETDATE() > HD.Deadline1 
    -- Điều kiện 2: Hợp đồng chưa được tất toán
    AND HD.TrangThai <> N'Đã thanh toán đủ'
    -- Điều kiện 3: Loại bỏ các hợp đồng đã thanh lý hoàn toàn (tùy chọn)
    AND HD.TrangThai <> N'Đã bán thanh lý'
ORDER BY [Số ngày quá hạn] DESC;
GO

```

<p><img width="1918" height="1021" alt="image" src="https://github.com/user-attachments/assets/d4ad4a55-1876-4a81-97dc-604d591cdde2" />
hình 12 thi hành lệnh thành công
</p>

* Mô tả:

Hệ thống so sánh thời gian thực của máy chủ (GETDATE()) với mốc Deadline1. Nếu thời gian hiện tại đã bước qua mốc này mà trạng thái hợp đồng vẫn chưa là "Đã thanh toán đủ", khách hàng đó chính thức được liệt vào danh sách nợ xấu.

Việc loại trừ trạng thái N'Đã thanh toán đủ' là bắt buộc để tránh hiển thị những người đã trả hết nợ nhưng có ngày trả sau Deadline 1.

**Event 5**: Quản lý thanh lý tài sản

```

USE QuanLyCamDo;
GO

-- =================================================================================
-- Ý NHỎ 1 & 2: TỰ ĐỘNG CHUYỂN TRẠNG THÁI THEO DEADLINE
-- Hai ý này được gộp chung vào 1 Trigger trên bảng HopDong để tối ưu hiệu năng.
-- =================================================================================
CREATE OR ALTER TRIGGER trg_CheckDeadlines_HopDong
ON HopDong
AFTER UPDATE, INSERT -- Kích hoạt khi có thay đổi hoặc thêm mới hợp đồng
AS
BEGIN
    SET NOCOUNT ON;

    -- Ý 1: Chuyển sang "Quá hạn (nợ xấu)" nếu vượt Deadline 1
    UPDATE HopDong
    SET TrangThai = N'Quá hạn (nợ xấu)'
    FROM HopDong HD
    INNER JOIN inserted i ON HD.MaHD = i.MaHD
    WHERE HD.TrangThai = N'Đang vay' 
      AND GETDATE() > HD.Deadline1;

    -- Ý 2: Chuyển tài sản sang "Sẵn sàng thanh lý" nếu vượt Deadline 2
    -- Lưu ý: Chỉ cập nhật những tài sản đang bị giữ (Đang cầm cố)
    UPDATE TaiSan
    SET TrangThaiTS = N'Sẵn sàng thanh lý'
    FROM TaiSan TS
    INNER JOIN HopDong HD ON TS.MaHD = HD.MaHD
    INNER JOIN inserted i ON HD.MaHD = i.MaHD
    WHERE HD.TrangThai = N'Quá hạn (nợ xấu)' 
      AND GETDATE() > HD.Deadline2
      AND TS.TrangThaiTS = N'Đang cầm cố';
END;
GO

-- =================================================================================
-- Ý NHỎ 3: TỰ ĐỘNG CẬP NHẬT TÀI SẢN KHI HỢP ĐỒNG ĐÃ THANH LÝ
-- =================================================================================
CREATE OR ALTER TRIGGER trg_SyncAssetSold
ON HopDong
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Nếu trạng thái hợp đồng chuyển sang "Đã thanh lý" (IsSold)
    IF EXISTS (SELECT 1 FROM inserted i WHERE i.TrangThai = N'Đã thanh lý')
    BEGIN
        UPDATE TaiSan
        SET TrangThaiTS = N'Đã bán thanh lý'
        FROM TaiSan TS
        INNER JOIN inserted i ON TS.MaHD = i.MaHD
        WHERE i.TrangThai = N'Đã thanh lý'
          AND TS.TrangThaiTS = N'Sẵn sàng thanh lý'; -- Chỉ bán những đồ đã sẵn sàng
    END
END;
GO

```

<P><img width="1920" height="994" alt="image" src="https://github.com/user-attachments/assets/6fbdadb3-850c-46bb-94a8-c68159c3b723" />
HÌnh 13:thi hành lệnh thành công
</P>

* MÔ tả:

Ý nhỏ 1 (Kiểm soát nợ xấu): Trigger này đóng vai trò như một "người gác cổng". Ngay khi nhân viên có bất kỳ thao tác chỉnh sửa nào trên hợp đồng, hệ thống sẽ bí mật so sánh ngày hiện tại với Deadline1. Nếu khách hàng đã trễ hẹn, trạng thái sẽ bị "đóng dấu" nợ xấu ngay lập tức để hệ thống bắt đầu áp dụng lãi kép từ Event 2.

Ý nhỏ 2 (Chốt chặn thanh lý): Đây là logic bảo vệ chủ tiệm. Khi nợ xấu kéo dài vượt qua mốc Deadline 2, Trigger sẽ tự động quét xuống bảng TaiSan và gắn nhãn "Sẵn sàng thanh lý". Đây là tín hiệu cho phép bộ phận bán hàng bắt đầu đăng tin thanh lý món đồ đó.

Ý nhỏ 3 (Đồng bộ hóa cuối cùng): Đây là bước quan trọng để đảm bảo tính nhất quán dữ liệu giữa bảng Cha (HopDong) và bảng Con (TaiSan). Khi chủ tiệm chốt bán được món đồ và cập nhật hợp đồng thành "Đã thanh lý", Trigger này sẽ tự động chuyển trạng thái của toàn bộ tài sản liên quan sang "Đã bán thanh lý". Điều này giúp tránh sai sót nhầm lẫn khi tài sản đã bán rồi mà trong kho vẫn báo là "Sẵn sàng".
