```sql
use QLDT

-- PHẦN 1: RÀNG BUỘC MIỀN GIÁ TRỊ - KỸ THUẬT CHECK

-- IC1: Giới tính GV chỉ nhận 'Nam' hoặc 'Nữ'
ALTER TABLE GIAOVIEN
ADD CONSTRAINT CK_GV_PHAI CHECK (PHAI IN (N'Nam', N'Nữ'))
GO

-- IC2: Lương GV là bội số của 10
ALTER TABLE GIAOVIEN
ADD CONSTRAINT CK_GV_LUONG CHECK (LUONG % 10 = 0)
GO

-- IC3: Tuổi GV từ 18 đến 60
ALTER TABLE GIAOVIEN
ADD CONSTRAINT CK_GV_TUOI CHECK (
    DATEDIFF(YEAR, NGSINH, GETDATE())
        - CASE WHEN DATEADD(YEAR, DATEDIFF(YEAR, NGSINH, GETDATE()), NGSINH) > GETDATE()
               THEN 1 ELSE 0 END
    BETWEEN 18 AND 60
)
GO

-- IC1
CREATE RULE RL_PHAI AS @PHAI IN (N'Nam', N'Nữ')
GO
EXEC sp_bindrule 'RL_PHAI', 'GIAOVIEN.PHAI'
GO

-- IC2
CREATE RULE RL_LUONG AS @LUONG % 10 = 0
GO
EXEC sp_bindrule 'RL_LUONG', 'GIAOVIEN.LUONG'
GO

-- IC3
CREATE RULE RL_TUOI AS
    DATEDIFF(YEAR, @NGSINH, GETDATE()) BETWEEN 18 AND 60
GO
EXEC sp_bindrule 'RL_TUOI', 'GIAOVIEN.NGSINH'
GO

-- Bổ sung cột QUAN HỆ cho NGUOITHAN (phục vụ IC10, IC11, IC12)
ALTER TABLE NGUOITHAN ADD QUANHE NVARCHAR(20)
GO

-- PHẦN 2: TRIGGER

-- IC1: Tên đề tài phải duy nhất
CREATE OR ALTER TRIGGER UTR_IC1_TenDeTai_DuyNhat
ON DETAI
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(TENDT)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN DETAI D ON D.MADT <> I.MADT AND D.TENDT = I.TENDT
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC1: Ten de tai da ton tai', 16, 1)
            RETURN
        END
    END
END
GO

-- IC2: Trưởng bộ môn phải sinh trước năm 1975
CREATE OR ALTER TRIGGER UTR_IC2_TruongBM_SinhTruoc1975
ON BOMON
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(TRUONGBM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN G ON G.MAGV = I.TRUONGBM
            WHERE I.TRUONGBM IS NOT NULL AND YEAR(G.NGSINH) >= 1975
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC2: Truong bo mon phai sinh truoc nam 1975', 16, 1)
            RETURN
        END
    END
END
GO

-- IC3: Mỗi bộ môn phải có ít nhất 1 GV nữ
-- (chỉ chặn hành vi xóa/sửa làm mất GV nữ cuối cùng của bộ môn - xem ghi chú đầu file)
CREATE OR ALTER TRIGGER UTR_IC3_BoMon_CoGVNu
ON GIAOVIEN
FOR DELETE, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT D.MABM
        FROM deleted D
        WHERE D.MABM IS NOT NULL
          AND D.PHAI = N'Nữ'
          AND NOT EXISTS (
                SELECT 1 FROM GIAOVIEN G
                WHERE G.MABM = D.MABM AND G.PHAI = N'Nữ'
          )
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC3: Bo mon phai co it nhat 1 giao vien nu', 16, 1)
        RETURN
    END
END
GO

-- IC4: Mỗi GV phải có ít nhất 1 số điện thoại
-- (chỉ chặn hành vi xóa/sửa làm GV mất số điện thoại cuối cùng - xem ghi chú đầu file)
CREATE OR ALTER TRIGGER UTR_IC4_GV_CoSDT
ON GV_DT
FOR DELETE, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT D.MAGV
        FROM deleted D
        WHERE NOT EXISTS (SELECT 1 FROM GV_DT G WHERE G.MAGV = D.MAGV)
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC4: Moi giao vien phai co it nhat 1 so dien thoai', 16, 1)
        RETURN
    END
END
GO

-- IC5: Mỗi GV có tối đa 3 số điện thoại
CREATE OR ALTER TRIGGER UTR_IC5_GV_ToiDa3SDT
ON GV_DT
FOR INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT I.MAGV
        FROM inserted I
        GROUP BY I.MAGV
        HAVING (SELECT COUNT(*) FROM GV_DT G WHERE G.MAGV = I.MAGV) > 3
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC5: Moi giao vien co toi da 3 so dien thoai', 16, 1)
        RETURN
    END
END
GO

-- IC6: Mỗi bộ môn phải có ít nhất 4 GV
-- (chỉ chặn hành vi xóa/sửa làm bộ môn còn dưới 4 GV - xem ghi chú đầu file)
CREATE OR ALTER TRIGGER UTR_IC6_BoMon_ToiThieu4GV
ON GIAOVIEN
FOR DELETE, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT D.MABM
        FROM deleted D
        WHERE D.MABM IS NOT NULL
        GROUP BY D.MABM
        HAVING (SELECT COUNT(*) FROM GIAOVIEN G WHERE G.MABM = D.MABM) < 4
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC6: Moi bo mon phai co it nhat 4 giao vien', 16, 1)
        RETURN
    END
END
GO

-- IC7: Trưởng bộ môn phải là người lớn tuổi nhất trong bộ môn
CREATE OR ALTER TRIGGER UTR_IC7a_TruongBM_LonTuoiNhat
ON BOMON
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(TRUONGBM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN GT ON GT.MAGV = I.TRUONGBM
            WHERE I.TRUONGBM IS NOT NULL
              AND EXISTS (
                    SELECT 1 FROM GIAOVIEN G2
                    WHERE G2.MABM = I.MABM AND G2.NGSINH < GT.NGSINH
              )
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC7: Truong bo mon phai la nguoi lon tuoi nhat trong bo mon', 16, 1)
            RETURN
        END
    END
END
GO

-- IC7 (bổ sung): khi GV được thêm/chuyển bộ môn hoặc đổi ngày sinh,
-- không được lớn tuổi hơn trưởng bộ môn hiện tại của bộ môn đó
CREATE OR ALTER TRIGGER UTR_IC7b_GV_KhongLonHonTruongBM
ON GIAOVIEN
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(MABM) OR UPDATE(NGSINH)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN BOMON B ON B.MABM = I.MABM
            JOIN GIAOVIEN GT ON GT.MAGV = B.TRUONGBM
            WHERE B.TRUONGBM IS NOT NULL
              AND B.TRUONGBM <> I.MAGV
              AND I.NGSINH < GT.NGSINH
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC7: Khong duoc lon tuoi hon truong bo mon', 16, 1)
            RETURN
        END
    END
END
GO

-- IC8: GV đã là trưởng bộ môn thì không được làm GVQLCM (người quản lý chuyên môn) của GV khác
CREATE OR ALTER TRIGGER UTR_IC8a_TruongBM_KhongLaGVQLCM
ON GIAOVIEN
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(GVQLCM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            WHERE I.GVQLCM IS NOT NULL
              AND EXISTS (SELECT 1 FROM BOMON B WHERE B.TRUONGBM = I.GVQLCM)
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC8: Truong bo mon khong duoc lam GVQLCM cua GV khac', 16, 1)
            RETURN
        END
    END
END
GO

CREATE OR ALTER TRIGGER UTR_IC8b_GVQLCM_KhongDuocLaTruongBM
ON BOMON
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(TRUONGBM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            WHERE I.TRUONGBM IS NOT NULL
              AND EXISTS (SELECT 1 FROM GIAOVIEN G WHERE G.GVQLCM = I.TRUONGBM)
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC8: GV dang la GVQLCM cua nguoi khac, khong the lam truong bo mon', 16, 1)
            RETURN
        END
    END
END
GO

-- IC9: GV và GVQLCM (người quản lý chuyên môn) phải cùng bộ môn
CREATE OR ALTER TRIGGER UTR_IC9_GV_GVQLCM_CungBoMon
ON GIAOVIEN
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(GVQLCM) OR UPDATE(MABM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN GQ ON GQ.MAGV = I.GVQLCM
            WHERE I.GVQLCM IS NOT NULL AND GQ.MABM <> I.MABM
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC9: GV va GVQLCM phai cung bo mon', 16, 1)
            RETURN
        END
    END
END
GO

-- IC10: Mỗi GV có tối đa 1 vợ/chồng (QUANHE = N'Vợ' hoặc N'Chồng')
CREATE OR ALTER TRIGGER UTR_IC10_ToiDaMotVoChong
ON NGUOITHAN
FOR INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT I.MAGV
        FROM inserted I
        WHERE I.QUANHE IN (N'Vợ', N'Chồng')
        GROUP BY I.MAGV
        HAVING (
            SELECT COUNT(*) FROM NGUOITHAN N
            WHERE N.MAGV = I.MAGV AND N.QUANHE IN (N'Vợ', N'Chồng')
        ) > 1
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC10: Moi giao vien co toi da 1 vo/chong', 16, 1)
        RETURN
    END
END
GO

-- IC11: Giới tính vợ/chồng phải đối nghịch với GV
CREATE OR ALTER TRIGGER UTR_IC11_GioiTinhVoChongDoiNghich
ON NGUOITHAN
FOR INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT 1 FROM inserted I
        JOIN GIAOVIEN G ON G.MAGV = I.MAGV
        WHERE I.QUANHE IN (N'Vợ', N'Chồng') AND I.PHAI = G.PHAI
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC11: Gioi tinh vo/chong phai doi nghich voi GV', 16, 1)
        RETURN
    END
END
GO

-- IC12: Nếu quan hệ là 'Con trai'/'Con gái' thì năm sinh GV phải sớm hơn năm sinh người thân
CREATE OR ALTER TRIGGER UTR_IC12_NamSinhSomHonConCai
ON NGUOITHAN
FOR INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT 1 FROM inserted I
        JOIN GIAOVIEN G ON G.MAGV = I.MAGV
        WHERE I.QUANHE IN (N'Con trai', N'Con gái')
          AND YEAR(G.NGSINH) >= YEAR(I.NGSINH)
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC12: Nam sinh GV phai som hon nam sinh con', 16, 1)
        RETURN
    END
END
GO

-- IC13: Một GV chủ nhiệm (GVCNDT) tối đa 3 đề tài
CREATE OR ALTER TRIGGER UTR_IC13_ToiDa3DeTaiChuNhiem
ON DETAI
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(GVCNDT)
    BEGIN
        IF EXISTS (
            SELECT I.GVCNDT
            FROM inserted I
            WHERE I.GVCNDT IS NOT NULL
            GROUP BY I.GVCNDT
            HAVING (SELECT COUNT(*) FROM DETAI D WHERE D.GVCNDT = I.GVCNDT) > 3
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC13: Mot GV chu nhiem toi da 3 de tai', 16, 1)
            RETURN
        END
    END
END
GO

-- IC14: Mỗi đề tài phải có ít nhất 1 công việc
CREATE OR ALTER TRIGGER UTR_IC14_DeTai_CoCongViec
ON CONGVIEC
FOR DELETE
AS
BEGIN
    IF EXISTS (
        SELECT D.MADT
        FROM deleted D
        WHERE NOT EXISTS (SELECT 1 FROM CONGVIEC C WHERE C.MADT = D.MADT)
    )
    BEGIN
        ROLLBACK TRAN
        RAISERROR(N'IC14: Moi de tai phai co it nhat 1 cong viec', 16, 1)
        RETURN
    END
END
GO

-- IC15: Lương GV phải thấp hơn lương của GVQLCM (2 chiều)
CREATE OR ALTER TRIGGER UTR_IC15_LuongThapHonGVQLCM
ON GIAOVIEN
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(LUONG) OR UPDATE(GVQLCM)
    BEGIN
        -- Chiều 1: GV vừa thêm/sửa phải có lương thấp hơn GVQLCM của mình
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN GQ ON GQ.MAGV = I.GVQLCM
            WHERE I.GVQLCM IS NOT NULL AND I.LUONG >= GQ.LUONG
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC15: Luong GV phai thap hon luong GVQLCM', 16, 1)
            RETURN
        END
        -- Chiều 2: nếu GV này đang là GVQLCM của người khác, lương người đó vẫn phải thấp hơn
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN G2 ON G2.GVQLCM = I.MAGV
            WHERE G2.LUONG >= I.LUONG
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC15: Luong nhan vien duoi quyen phai thap hon luong GV nay', 16, 1)
            RETURN
        END
    END
END
GO

-- IC16: Lương trưởng bộ môn phải cao hơn lương tất cả GV trong bộ môn
CREATE OR ALTER TRIGGER UTR_IC16a_LuongTruongBMCaoNhat_TuBOMON
ON BOMON
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(TRUONGBM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN GT ON GT.MAGV = I.TRUONGBM
            WHERE I.TRUONGBM IS NOT NULL
              AND EXISTS (
                    SELECT 1 FROM GIAOVIEN G2
                    WHERE G2.MABM = I.MABM AND G2.MAGV <> I.TRUONGBM
                      AND G2.LUONG >= GT.LUONG
              )
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC16: Luong truong bo mon phai cao hon tat ca GV trong bo mon', 16, 1)
            RETURN
        END
    END
END
GO

CREATE OR ALTER TRIGGER UTR_IC16b_LuongTruongBMCaoNhat_TuGIAOVIEN
ON GIAOVIEN
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(LUONG) OR UPDATE(MABM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN BOMON B ON B.MABM = I.MABM
            JOIN GIAOVIEN GT ON GT.MAGV = B.TRUONGBM
            WHERE B.TRUONGBM IS NOT NULL
              AND B.TRUONGBM <> I.MAGV
              AND I.LUONG >= GT.LUONG
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC16: Luong GV khong duoc lon hon hoac bang luong truong bo mon', 16, 1)
            RETURN
        END
    END
END
GO

-- IC17: Mỗi bộ môn phải có trưởng bộ môn, và trưởng bộ môn phải là GV của trường.
-- "Trưởng bộ môn là GV của trường" đã được đảm bảo bởi khóa ngoại FK_BOMON_GIAOVIEN.
-- "Luôn phải có trưởng" là ràng buộc vòng (xem ghi chú đầu file) - trigger dưới đây
-- chỉ ngăn KHÔNG cho xóa/đặt trưởng bộ môn hiện có về NULL.
CREATE OR ALTER TRIGGER UTR_IC17_BoMon_KhongDuocXoaTruong
ON BOMON
FOR UPDATE
AS
BEGIN
    IF UPDATE(TRUONGBM)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN deleted D ON D.MABM = I.MABM
            WHERE D.TRUONGBM IS NOT NULL AND I.TRUONGBM IS NULL
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC17: Bo mon phai luon co truong bo mon', 16, 1)
            RETURN
        END
    END
END
GO

-- IC18: Một GV làm GVQLCM cho tối đa 3 GV khác
CREATE OR ALTER TRIGGER UTR_IC18_ToiDa3NguoiDuocQuanLy
ON GIAOVIEN
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(GVQLCM)
    BEGIN
        IF EXISTS (
            SELECT I.GVQLCM
            FROM inserted I
            WHERE I.GVQLCM IS NOT NULL
            GROUP BY I.GVQLCM
            HAVING (SELECT COUNT(*) FROM GIAOVIEN G WHERE G.GVQLCM = I.GVQLCM) > 3
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC18: Mot GV lam GVQLCM toi da cho 3 GV khac', 16, 1)
            RETURN
        END
    END
END
GO

-- IC19: GV chỉ được tham gia đề tài mà GVCNDT (chủ nhiệm) cùng bộ môn với mình
CREATE OR ALTER TRIGGER UTR_IC19_ThamGia_CungBoMonChuNhiem
ON THAMGIADT
FOR INSERT, UPDATE
AS
BEGIN
    IF UPDATE(MAGV) OR UPDATE(MADT)
    BEGIN
        IF EXISTS (
            SELECT 1 FROM inserted I
            JOIN GIAOVIEN G ON G.MAGV = I.MAGV
            JOIN DETAI D ON D.MADT = I.MADT
            JOIN GIAOVIEN GCN ON GCN.MAGV = D.GVCNDT
            WHERE G.MABM <> GCN.MABM
        )
        BEGIN
            ROLLBACK TRAN
            RAISERROR(N'IC19: GV chi duoc tham gia de tai co chu nhiem cung bo mon', 16, 1)
            RETURN
        END
    END
END
GO
```
