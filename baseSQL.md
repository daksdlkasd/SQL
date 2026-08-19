```sql
-- Q1. Cho biết họ tên và mức lương của các giáo viên nữ.
SELECT HOTEN, LUONG FROM GIAOVIEN WHERE PHAI = N'Nữ';

-- Q2. Cho biết họ tên của các giáo viên và lương của họ sau khi tăng 10%.
SELECT HOTEN, LUONG * 1.1 AS LUONG_SAU_TANG FROM GIAOVIEN;

-- Q3. Cho biết mã của các giáo viên có họ tên bắt đầu là “Nguyễn” và lương trên $2000 hoặc, giáo viên là trưởng bộ môn nhận chức sau năm 1995.
SELECT DISTINCT GV.MAGV
FROM GIAOVIEN GV
LEFT JOIN BOMON BM ON GV.MAGV = BM.TRUONGBM
WHERE (GV.HOTEN LIKE N'Nguyễn%' AND GV.LUONG > 2000)
   OR (BM.NGAYNHANCHUC > '1995-12-31');

-- Q4. Cho biết tên những giáo viên khoa Công nghệ thông tin.
SELECT GV.HOTEN
FROM GIAOVIEN GV
JOIN BOMON BM ON GV.MABM = BM.MABM
JOIN KHOA K ON BM.MAKHOA = K.MAKHOA
WHERE K.TENKHOA = N'Công nghệ thông tin';

-- Q5. Cho biết thông tin của bộ môn cùng thông tin giảng viên làm trưởng bộ môn đó.
SELECT
    BM.MABM,
    BM.TENBM,
    BM.PHONG,
    BM.DIENTHOAI AS DIENTHOAI_BOMON,
    BM.MAKHOA,
    BM.NGAYNHANCHUC,
    GV.MAGV AS MAGV_TRUONGBM,
    GV.HOTEN AS HOTEN_TRUONGBM,
    GV.LUONG,
    GV.PHAI,
    GV.NGSINH,
    GV.DIACHI
FROM BOMON BM
LEFT JOIN GIAOVIEN GV ON BM.TRUONGBM = GV.MAGV;

-- Q6. Với mỗi giáo viên, hãy cho biết thông tin của bộ môn mà họ đang làm việc.
SELECT
    GV.MAGV,
    GV.HOTEN,
    GV.LUONG,
    GV.PHAI,
    GV.NGSINH,
    GV.DIACHI,
    BM.MABM,
    BM.TENBM,
    BM.PHONG,
    BM.DIENTHOAI AS DIENTHOAI_BOMON,
    BM.MAKHOA
FROM GIAOVIEN GV
JOIN BOMON BM ON GV.MABM = BM.MABM;

-- Q7. Cho biết tên đề tài và giáo viên chủ nhiệm đề tài.
SELECT
    DT.TENDT AS TEN_DE_TAI,
    GV.HOTEN AS GIAO_VIEN_CHU_NHIEM
FROM DETAI DT
JOIN GIAOVIEN GV ON DT.GVCNDT = GV.MAGV;

-- Q8. Với mỗi khoa cho biết thông tin trưởng khoa.
SELECT
    K.MAKHOA,
    K.TENKHOA,
    K.NAMTL,
    K.PHONG,
    K.DIENTHOAI AS DIENTHOAI_KHOA,
    K.NGAYNHANCHUC,
    GV.MAGV AS MAGV_TRUONGKHOA,
    GV.HOTEN AS HOTEN_TRUONGKHOA,
    GV.LUONG,
    GV.PHAI,
    GV.NGSINH,
    GV.DIACHI
FROM KHOA K
JOIN GIAOVIEN GV ON K.TRUONGKHOA = GV.MAGV;

-- Q9. Cho biết các giáo viên của bộ môn “Vi sinh” có tham gia đề tài 006.
SELECT DISTINCT GV.*
FROM GIAOVIEN GV
JOIN BOMON BM ON GV.MABM = BM.MABM
JOIN THAMGIADT TG ON GV.MAGV = TG.MAGV
WHERE BM.TENBM = N'Vi sinh'
  AND TG.MADT = '006';

-- Q10. Với những đề tài thuộc cấp quản lý “Thành phố”, cho biết mã đề tài, đề tài thuộc về chủ đề nào, họ tên người chủ nghiệm đề tài cùng với ngày sinh và địa chỉ của người ấy.
SELECT
    DT.MADT,
    CD.TENCD AS TEN_CHU_DE,
    GV.HOTEN AS HOTEN_GVCNDT,
    GV.NGSINH,
    GV.DIACHI
FROM DETAI DT
JOIN CHUDE CD ON DT.MACD = CD.MACD
JOIN GIAOVIEN GV ON DT.GVCNDT = GV.MAGV
WHERE DT.CAPQL = N'Thành phố';

-- Q11. Tìm họ tên của từng giáo viên và người phụ trách chuyên môn trực tiếp của giáo viên đó.
SELECT
    GV.HOTEN AS HOTEN_GIAOVIEN,
    QL.HOTEN AS HOTEN_NGUOI_PHU_TRACH
FROM GIAOVIEN GV
LEFT JOIN GIAOVIEN QL ON GV.GVQLCM = QL.MAGV;

-- Q12. Tìm họ tên của những giáo viên được “Nguyễn Thanh Tùng” phụ trách trực tiếp.
SELECT GV.HOTEN
FROM GIAOVIEN GV
JOIN GIAOVIEN QL ON GV.GVQLCM = QL.MAGV
WHERE QL.HOTEN = N'Nguyễn Thanh Tùng';

-- Q13. Cho biết tên giáo viên là trưởng bộ môn “Hệ thống thông tin”.
SELECT GV.HOTEN
FROM BOMON BM
JOIN GIAOVIEN GV ON BM.TRUONGBM = GV.MAGV
WHERE BM.TENBM = N'Hệ thống thông tin';

-- Q14. Cho biết tên người chủ nhiệm đề tài của những đề tài thuộc chủ đề Quản lý giáo dục.
SELECT DISTINCT GV.HOTEN
FROM DETAI DT
JOIN CHUDE CD ON DT.MACD = CD.MACD
JOIN GIAOVIEN GV ON DT.GVCNDT = GV.MAGV
WHERE CD.TENCD = N'Quản lý giáo dục';

-- Q15. Cho biết tên các công việc của đề tài HTTT quản lý các trường ĐH có thời gian bắt đầu trong tháng 3/2008.
SELECT CV.TENCV
FROM CONGVIEC CV
JOIN DETAI DT ON CV.MADT = DT.MADT
WHERE DT.TENDT = N'HTTT quản lý các trường ĐH'
  AND CV.NGAYBD >= '2008-03-01'
  AND CV.NGAYBD < '2008-04-01';

-- Q16. Cho biết tên giáo viên và tên người quản lý chuyên môn của giáo viên đó.
SELECT
    GV.HOTEN AS HOTEN_GIAOVIEN,
    QL.HOTEN AS HOTEN_NGUOI_QUAN_LY_CHUYEN_MON
FROM GIAOVIEN GV
LEFT JOIN GIAOVIEN QL ON GV.GVQLCM = QL.MAGV;

-- Q17. Cho các công việc bắt đầu trong khoảng từ 01/01/2007 đến 01/08/2007.
SELECT *
FROM CONGVIEC
WHERE NGAYBD >= '2007-01-01'
  AND NGAYBD <= '2007-08-01';

-- Q18. Cho biết họ tên các giáo viên cùng bộ môn với giáo viên “Trần Trà Hương”.
SELECT HOTEN
FROM GIAOVIEN
WHERE MABM = (SELECT MABM FROM GIAOVIEN WHERE HOTEN = N'Trần Trà Hương')
  AND MAGV <> (SELECT MAGV FROM GIAOVIEN WHERE HOTEN = N'Trần Trà Hương');

-- Q19. Tìm những giáo viên vừa là trưởng bộ môn vừa chủ nhiệm đề tài.
SELECT MAGV, HOTEN
FROM GIAOVIEN
WHERE MAGV IN (SELECT TRUONGBM FROM BOMON)
  AND MAGV IN (SELECT GVCNDT FROM DETAI);

-- Q20. Cho biết tên những giáo viên vừa là trưởng khoa và vừa là trưởng bộ môn.
SELECT DISTINCT GV.HOTEN
FROM GIAOVIEN GV
JOIN KHOA K ON GV.MAGV = K.TRUONGKHOA
JOIN BOMON BM ON GV.MAGV = BM.TRUONGBM;

-- Q21. Cho biết tên những trưởng bộ môn mà vừa chủ nhiệm đề tài.
SELECT MAGV, HOTEN
FROM GIAOVIEN
WHERE MAGV IN (SELECT TRUONGBM FROM BOMON)
  AND MAGV IN (SELECT GVCNDT FROM DETAI);

-- Q22. Cho biết mã số các trưởng khoa có chủ nhiệm đề tài.
SELECT DISTINCT TRUONGKHOA AS MAGV
FROM KHOA
WHERE TRUONGKHOA IN (SELECT GVCNDT FROM DETAI);

-- Q23. Cho biết mã số các giáo viên thuộc bộ môn “HTTT” hoặc có tham gia đề tài mã “001”.
SELECT MAGV FROM GIAOVIEN WHERE MABM = 'HTTT'
UNION
SELECT MAGV FROM THAMGIADT WHERE MADT = '001';

-- Q24. Cho biết giáo viên làm việc cùng bộ môn với giáo viên 002.
SELECT *
FROM GIAOVIEN
WHERE MABM = (SELECT MABM FROM GIAOVIEN WHERE MAGV = '002')
  AND MAGV <> '002';

-- Q25. Tìm những giáo viên là trưởng bộ môn.
SELECT DISTINCT GV.*
FROM GIAOVIEN GV
JOIN BOMON BM ON GV.MAGV = BM.TRUONGBM;

-- Q26. Cho biết họ tên và mức lương của các giáo viên.
SELECT HOTEN, LUONG
FROM GIAOVIEN;


-- Q27. Display the number of lecturers and their total salary.
SELECT
    COUNT(MAGV) AS NUMBER_OF_LECTURERS,
    SUM(LUONG) AS TOTAL_SALARY
FROM GIAOVIEN;

-- Q28. Display the number of lecturers and the average salary for each department.
SELECT
    BM.TENBM AS DEPARTMENT_NAME,
    COUNT(GV.MAGV) AS NUMBER_OF_LECTURERS,
    AVG(GV.LUONG) AS AVERAGE_SALARY
FROM BOMON BM
JOIN GIAOVIEN GV ON BM.MABM = GV.MABM
GROUP BY BM.MABM, BM.TENBM;

-- Q29. Display the topic name and the number of projects belonging to that topic.
SELECT
    CD.TENCD AS TOPIC_NAME,
    COUNT(DT.MADT) AS NUMBER_OF_PROJECTS
FROM CHUDE CD
LEFT JOIN DETAI DT ON CD.MACD = DT.MACD
GROUP BY CD.MACD, CD.TENCD;

-- Q30. Display the lecturer name and the number of projects in which the lecturer has participated.
SELECT
    GV.HOTEN AS LECTURER_NAME,
    COUNT(DISTINCT TG.MADT) AS NUMBER_OF_PROJECTS
FROM GIAOVIEN GV
JOIN THAMGIADT TG ON GV.MAGV = TG.MAGV
GROUP BY GV.MAGV, GV.HOTEN;

-- Q31. Display the lecturer name and the number of projects for which the lecturer serves as the project leader.
SELECT
    GV.HOTEN AS LECTURER_NAME,
    COUNT(DT.MADT) AS NUMBER_OF_LED_PROJECTS
FROM GIAOVIEN GV
JOIN DETAI DT ON GV.MAGV = DT.GVCNDT
GROUP BY GV.MAGV, GV.HOTEN;

-- Q32. For each lecturer, display the lecturer name and the number of relatives of that lecturer.
SELECT
    GV.HOTEN AS LECTURER_NAME,
    COUNT(NT.TEN) AS NUMBER_OF_RELATIVES
FROM GIAOVIEN GV
LEFT JOIN NGUOITHAN NT ON GV.MAGV = NT.MAGV
GROUP BY GV.MAGV, GV.HOTEN;

-- Q33. Display the names of lecturers who have participated in three or more projects.
SELECT
    GV.HOTEN AS LECTURER_NAME
FROM GIAOVIEN GV
JOIN THAMGIADT TG ON GV.MAGV = TG.MAGV
GROUP BY GV.MAGV, GV.HOTEN
HAVING COUNT(DISTINCT TG.MADT) >= 3;

-- Q34. Display the number of lecturers who have participated in the project “Ứng dụng hoá học xanh”.
SELECT
    COUNT(DISTINCT TG.MAGV) AS NUMBER_OF_LECTURERS
FROM THAMGIADT TG
JOIN DETAI DT ON TG.MADT = DT.MADT
WHERE DT.TENDT = N'Ứng dụng hóa học xanh';
```

# SQL Cheat Sheet — Ôn tập nhanh trước kiểm tra

> Áp dụng cho các bài truy vấn trên cơ sở dữ liệu `GIAOVIEN`, `BOMON`, `KHOA`, `DETAI`, `THAMGIADT`, `CHUDE`, `CONGVIEC`, `NGUOITHAN`.

---

# 1. Thứ tự viết một câu SQL

```sql
SELECT ...
FROM ...
JOIN ... ON ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...;
```

## Thứ tự xử lý logic

```text
FROM / JOIN
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ DISTINCT
→ ORDER BY
```

---

# 2. SELECT — Chọn cột cần hiển thị

## Cú pháp

```sql
SELECT cot1, cot2
FROM TEN_BANG;
```

## Trường hợp sử dụng

- Lấy một hoặc nhiều cột.
- Tính toán giá trị mới.
- Đặt bí danh cho cột.

## Ví dụ

```sql
SELECT HOTEN, LUONG
FROM GIAOVIEN;
```

```sql
SELECT HOTEN, LUONG * 1.1 AS LUONG_SAU_TANG
FROM GIAOVIEN;
```

## Lấy toàn bộ cột

```sql
SELECT *
FROM GIAOVIEN;
```

> Khi nộp bài, nên ghi rõ tên cột nếu đề chỉ yêu cầu một số thông tin.

---

# 3. AS — Đặt bí danh

## Bí danh cho cột

```sql
SELECT HOTEN AS TEN_GIAO_VIEN
FROM GIAOVIEN;
```

## Bí danh cho bảng

```sql
SELECT GV.HOTEN
FROM GIAOVIEN AS GV;
```

Có thể bỏ từ khóa `AS` khi đặt bí danh bảng:

```sql
FROM GIAOVIEN GV
```

## Trường hợp sử dụng

- Làm tên kết quả dễ hiểu.
- Viết truy vấn ngắn hơn.
- Phân biệt các cột trùng tên khi kết nhiều bảng.
- Bắt buộc nên dùng khi self join.

---

# 4. DISTINCT — Loại bỏ kết quả trùng

## Cú pháp

```sql
SELECT DISTINCT cot
FROM TEN_BANG;
```

## Ví dụ

```sql
SELECT DISTINCT GV.MAGV
FROM GIAOVIEN GV
JOIN BOMON BM
    ON GV.MAGV = BM.TRUONGBM;
```

## Khi dùng

- Một giáo viên có thể xuất hiện nhiều lần sau phép `JOIN`.
- Chỉ cần mỗi giá trị xuất hiện một lần.
- Đếm số đối tượng khác nhau:

```sql
COUNT(DISTINCT TG.MADT)
```

---

# 5. WHERE — Lọc từng dòng

## Cú pháp

```sql
SELECT ...
FROM ...
WHERE dieu_kien;
```

## Ví dụ

```sql
SELECT HOTEN, LUONG
FROM GIAOVIEN
WHERE PHAI = N'Nữ';
```

## Toán tử so sánh

| Toán tử        | Ý nghĩa           |
| -------------- | ----------------- |
| `=`            | Bằng              |
| `<>` hoặc `!=` | Khác              |
| `>`            | Lớn hơn           |
| `<`            | Nhỏ hơn           |
| `>=`           | Lớn hơn hoặc bằng |
| `<=`           | Nhỏ hơn hoặc bằng |

---

# 6. AND, OR, NOT — Kết hợp điều kiện

## AND

Tất cả điều kiện phải đúng.

```sql
WHERE PHAI = N'Nữ'
  AND LUONG > 2000
```

## OR

Chỉ cần một điều kiện đúng.

```sql
WHERE MABM = 'HTTT'
   OR LUONG > 3000
```

## NOT

Phủ định điều kiện.

```sql
WHERE NOT PHAI = N'Nữ'
```

## Quy tắc quan trọng

Luôn dùng ngoặc khi kết hợp `AND` và `OR`.

```sql
WHERE (HOTEN LIKE N'Nguyễn%' AND LUONG > 2000)
   OR NGAYNHANCHUC > '1995-12-31'
```

> `AND` được xử lý trước `OR`.

---

# 7. LIKE — Tìm chuỗi theo mẫu

## Cú pháp

```sql
WHERE cot LIKE mau
```

## Ký tự đại diện

| Ký tự | Ý nghĩa                     |
| ----- | --------------------------- |
| `%`   | Không, một hoặc nhiều ký tự |
| `_`   | Chính xác một ký tự         |

## Ví dụ

Tên bắt đầu bằng `Nguyễn`:

```sql
WHERE HOTEN LIKE N'Nguyễn%'
```

Tên kết thúc bằng `An`:

```sql
WHERE HOTEN LIKE N'%An'
```

Tên chứa `Thanh`:

```sql
WHERE HOTEN LIKE N'%Thanh%'
```

Tên có ký tự thứ hai là `g`:

```sql
WHERE HOTEN LIKE N'_g%'
```

---

# 8. N'...' — Chuỗi Unicode trong SQL Server

Khi chuỗi có tiếng Việt, nên viết:

```sql
N'Nữ'
N'Công nghệ thông tin'
N'Quản lý giáo dục'
```

Ví dụ:

```sql
WHERE TENKHOA = N'Công nghệ thông tin'
```

---

# 9. BETWEEN — Lọc trong một khoảng

## Cú pháp

```sql
WHERE cot BETWEEN gia_tri_1 AND gia_tri_2
```

`BETWEEN` lấy cả hai đầu mút.

```sql
WHERE NGAYBD BETWEEN '2007-01-01' AND '2007-08-01'
```

Tương đương:

```sql
WHERE NGAYBD >= '2007-01-01'
  AND NGAYBD <= '2007-08-01'
```

## Lọc trọn một tháng

Nên dùng khoảng nửa mở:

```sql
WHERE NGAYBD >= '2008-03-01'
  AND NGAYBD <  '2008-04-01'
```

Cách này an toàn nếu cột có cả ngày và giờ.

---

# 10. IN — Thuộc một tập giá trị

## Danh sách cố định

```sql
WHERE MABM IN ('HTTT', 'CNPM', 'KHMT')
```

## Kết quả từ truy vấn con

```sql
WHERE MAGV IN (
    SELECT TRUONGBM
    FROM BOMON
)
```

## Khi dùng

- So sánh với nhiều giá trị.
- Kiểm tra một giá trị có xuất hiện trong kết quả truy vấn con hay không.

---

# 11. IS NULL và IS NOT NULL

Không được viết:

```sql
WHERE GVQLCM = NULL
```

Cách đúng:

```sql
WHERE GVQLCM IS NULL
```

```sql
WHERE GVQLCM IS NOT NULL
```

---

# 12. INNER JOIN — Chỉ lấy các dòng khớp nhau

`JOIN` mặc định là `INNER JOIN`.

## Cú pháp

```sql
SELECT ...
FROM BANG_A A
JOIN BANG_B B
    ON A.KHOA = B.KHOA;
```

## Ví dụ

Lấy giáo viên và bộ môn của họ:

```sql
SELECT GV.HOTEN, BM.TENBM
FROM GIAOVIEN GV
JOIN BOMON BM
    ON GV.MABM = BM.MABM;
```

## Khi dùng

- Chỉ cần các đối tượng có dữ liệu liên quan ở cả hai bảng.
- Lấy thông tin qua khóa chính và khóa ngoại.

## Ghi nhớ

```text
INNER JOIN = chỉ giữ phần giao nhau
```

---

# 13. LEFT JOIN — Giữ toàn bộ bảng bên trái

## Cú pháp

```sql
SELECT ...
FROM BANG_TRAI A
LEFT JOIN BANG_PHAI B
    ON A.KHOA = B.KHOA;
```

## Ví dụ

Hiển thị mọi giáo viên, kể cả người chưa có quản lý chuyên môn:

```sql
SELECT
    GV.HOTEN,
    QL.HOTEN AS NGUOI_QUAN_LY
FROM GIAOVIEN GV
LEFT JOIN GIAOVIEN QL
    ON GV.GVQLCM = QL.MAGV;
```

## Khi dùng

- Đề có từ:
  - “Mỗi giáo viên”.
  - “Mỗi khoa”.
  - “Mỗi đề tài”.
  - Kể cả đối tượng chưa có dữ liệu liên quan.
- Muốn kết quả đếm bằng `0`.

## Ghi nhớ

```text
LEFT JOIN = giữ tất cả dòng bảng bên trái
```

---

# 14. RIGHT JOIN và FULL JOIN

## RIGHT JOIN

Giữ toàn bộ bảng bên phải.

```sql
FROM A
RIGHT JOIN B
    ON A.ID = B.ID
```

Có thể đổi vị trí hai bảng để viết thành `LEFT JOIN`, nên ít dùng hơn.

## FULL OUTER JOIN

Giữ toàn bộ dòng của cả hai bảng.

```sql
FROM A
FULL OUTER JOIN B
    ON A.ID = B.ID
```

---

# 15. JOIN nhiều bảng

## Mẫu

```sql
SELECT ...
FROM GIAOVIEN GV
JOIN BOMON BM
    ON GV.MABM = BM.MABM
JOIN KHOA K
    ON BM.MAKHOA = K.MAKHOA;
```

## Ví dụ

Giáo viên thuộc khoa Công nghệ thông tin:

```sql
SELECT GV.HOTEN
FROM GIAOVIEN GV
JOIN BOMON BM
    ON GV.MABM = BM.MABM
JOIN KHOA K
    ON BM.MAKHOA = K.MAKHOA
WHERE K.TENKHOA = N'Công nghệ thông tin';
```

## Cách xác định đường JOIN

```text
GIAOVIEN.MABM → BOMON.MABM
BOMON.MAKHOA  → KHOA.MAKHOA
```

---

# 16. SELF JOIN — Kết một bảng với chính nó

## Khi dùng

- Giáo viên và người quản lý của giáo viên.
- Những giáo viên cùng bộ môn.
- So sánh các dòng trong cùng một bảng.

## Ví dụ

```sql
SELECT
    GV.HOTEN AS GIAO_VIEN,
    QL.HOTEN AS NGUOI_QUAN_LY
FROM GIAOVIEN GV
LEFT JOIN GIAOVIEN QL
    ON GV.GVQLCM = QL.MAGV;
```

`GV` và `QL` đều là bảng `GIAOVIEN`, nhưng có hai vai trò khác nhau.

---

# 17. Hàm kết tập

| Hàm       | Chức năng       |
| --------- | --------------- |
| `COUNT()` | Đếm             |
| `SUM()`   | Tính tổng       |
| `AVG()`   | Tính trung bình |
| `MAX()`   | Lấy lớn nhất    |
| `MIN()`   | Lấy nhỏ nhất    |

## Ví dụ

```sql
SELECT
    COUNT(MAGV) AS SO_GIAO_VIEN,
    SUM(LUONG) AS TONG_LUONG,
    AVG(LUONG) AS LUONG_TRUNG_BINH,
    MAX(LUONG) AS LUONG_CAO_NHAT,
    MIN(LUONG) AS LUONG_THAP_NHAT
FROM GIAOVIEN;
```

---

# 18. COUNT(\*) và COUNT(cột)

## COUNT(\*)

Đếm số dòng.

```sql
COUNT(*)
```

## COUNT(cột)

Đếm số giá trị khác `NULL`.

```sql
COUNT(TG.MADT)
```

## COUNT(DISTINCT cột)

Đếm số giá trị khác nhau.

```sql
COUNT(DISTINCT TG.MADT)
```

## Quy tắc chọn

| Yêu cầu                              | Cách dùng               |
| ------------------------------------ | ----------------------- |
| Đếm số dòng                          | `COUNT(*)`              |
| Không đếm `NULL`                     | `COUNT(cột)`            |
| Không đếm trùng                      | `COUNT(DISTINCT cột)`   |
| Sau `LEFT JOIN`, muốn kết quả bằng 0 | `COUNT(khóa_bảng_phải)` |

---

# 19. GROUP BY — Gom nhóm

## Cú pháp

```sql
SELECT
    cot_nhom,
    HAM_KET_TAP(cot)
FROM TEN_BANG
GROUP BY cot_nhom;
```

## Ví dụ

Số giáo viên và lương trung bình theo bộ môn:

```sql
SELECT
    BM.MABM,
    BM.TENBM,
    COUNT(GV.MAGV) AS SO_GIAO_VIEN,
    AVG(GV.LUONG) AS LUONG_TRUNG_BINH
FROM BOMON BM
LEFT JOIN GIAOVIEN GV
    ON GV.MABM = BM.MABM
GROUP BY
    BM.MABM,
    BM.TENBM;
```

## Quy tắc bắt buộc

Mọi cột trong `SELECT` mà không nằm trong hàm kết tập phải xuất hiện trong `GROUP BY`.

Đúng:

```sql
SELECT BM.MABM, BM.TENBM, COUNT(GV.MAGV)
...
GROUP BY BM.MABM, BM.TENBM;
```

Sai:

```sql
SELECT BM.MABM, BM.TENBM, COUNT(GV.MAGV)
...
GROUP BY BM.MABM;
```

---

# 20. HAVING — Lọc sau khi gom nhóm

## Cú pháp

```sql
SELECT cot_nhom, COUNT(...)
FROM ...
GROUP BY cot_nhom
HAVING dieu_kien_nhom;
```

## Ví dụ

Giáo viên tham gia từ ba đề tài trở lên:

```sql
SELECT
    GV.MAGV,
    GV.HOTEN
FROM GIAOVIEN GV
JOIN THAMGIADT TG
    ON GV.MAGV = TG.MAGV
GROUP BY
    GV.MAGV,
    GV.HOTEN
HAVING COUNT(DISTINCT TG.MADT) >= 3;
```

## Phân biệt WHERE và HAVING

| `WHERE`                           | `HAVING`                                 |
| --------------------------------- | ---------------------------------------- |
| Lọc từng dòng                     | Lọc từng nhóm                            |
| Trước `GROUP BY`                  | Sau `GROUP BY`                           |
| Không dùng trực tiếp hàm tổng hợp | Thường dùng với `COUNT`, `SUM`, `AVG`... |

Ví dụ:

```sql
WHERE GV.LUONG > 2000
```

```sql
HAVING COUNT(DISTINCT TG.MADT) >= 3
```

---

# 21. ORDER BY — Sắp xếp kết quả

## Cú pháp

```sql
ORDER BY cot ASC;
```

```sql
ORDER BY cot DESC;
```

## Ý nghĩa

| Từ khóa | Ý nghĩa            |
| ------- | ------------------ |
| `ASC`   | Tăng dần, mặc định |
| `DESC`  | Giảm dần           |

## Ví dụ

```sql
SELECT HOTEN, LUONG
FROM GIAOVIEN
ORDER BY LUONG DESC;
```

Sắp xếp nhiều tiêu chí:

```sql
ORDER BY MABM ASC, LUONG DESC;
```

> `ORDER BY` thường nằm cuối câu truy vấn.

---

# 22. UNION — Hợp hai tập kết quả

## Cú pháp

```sql
SELECT cot
FROM BANG_A

UNION

SELECT cot
FROM BANG_B;
```

## Ví dụ

Giáo viên thuộc bộ môn `HTTT` hoặc tham gia đề tài `001`:

```sql
SELECT MAGV
FROM GIAOVIEN
WHERE MABM = 'HTTT'

UNION

SELECT MAGV
FROM THAMGIADT
WHERE MADT = '001';
```

## Điều kiện

Hai câu `SELECT` phải có:

- Cùng số cột.
- Kiểu dữ liệu tương thích.
- Thứ tự cột tương ứng.

## UNION và UNION ALL

| Lệnh        | Kết quả         |
| ----------- | --------------- |
| `UNION`     | Loại dòng trùng |
| `UNION ALL` | Giữ dòng trùng  |

---

# 23. INTERSECT — Giao hai tập kết quả

```sql
SELECT MAGV
FROM BOMON

INTERSECT

SELECT GVCNDT
FROM DETAI;
```

Dùng khi cần đối tượng xuất hiện trong cả hai tập.

Ví dụ ngữ nghĩa:

```text
Vừa là trưởng bộ môn, vừa là chủ nhiệm đề tài
```

---

# 24. EXCEPT — Hiệu hai tập kết quả

```sql
SELECT MAGV
FROM GIAOVIEN

EXCEPT

SELECT MAGV
FROM THAMGIADT;
```

Dùng để tìm các đối tượng thuộc tập thứ nhất nhưng không thuộc tập thứ hai.

Ví dụ:

```text
Giáo viên chưa từng tham gia đề tài
```

> Oracle sử dụng `MINUS` thay cho `EXCEPT`.

---

# 25. Truy vấn con — Subquery

## Truy vấn con trả về một giá trị

```sql
SELECT *
FROM GIAOVIEN
WHERE MABM = (
    SELECT MABM
    FROM GIAOVIEN
    WHERE MAGV = '002'
);
```

## Truy vấn con trả về nhiều giá trị

Dùng `IN`:

```sql
SELECT *
FROM GIAOVIEN
WHERE MAGV IN (
    SELECT TRUONGBM
    FROM BOMON
);
```

## Khi dùng

- So sánh với kết quả truy vấn khác.
- Tìm đối tượng có cùng thuộc tính với một đối tượng cho trước.
- Kiểm tra thành viên trong một tập.

---

# 26. EXISTS và NOT EXISTS

## EXISTS

Kiểm tra có ít nhất một dòng phù hợp.

```sql
SELECT GV.*
FROM GIAOVIEN GV
WHERE EXISTS (
    SELECT 1
    FROM THAMGIADT TG
    WHERE TG.MAGV = GV.MAGV
);
```

## NOT EXISTS

Tìm đối tượng không có dòng liên quan.

```sql
SELECT GV.*
FROM GIAOVIEN GV
WHERE NOT EXISTS (
    SELECT 1
    FROM THAMGIADT TG
    WHERE TG.MAGV = GV.MAGV
);
```

## Khi dùng

- “Có tham gia ít nhất một”.
- “Chưa từng tham gia”.
- “Không tồn tại”.
- Các bài toán “tất cả” với `NOT EXISTS` lồng nhau.

---

# 27. Mẫu truy vấn thường gặp

## 27.1. Lọc theo điều kiện

```sql
SELECT cot
FROM bang
WHERE dieu_kien;
```

## 27.2. Lấy dữ liệu từ nhiều bảng

```sql
SELECT ...
FROM A
JOIN B ON ...
JOIN C ON ...
WHERE ...;
```

## 27.3. Đếm theo nhóm

```sql
SELECT
    cot_nhom,
    COUNT(cot_khoa) AS SO_LUONG
FROM ...
GROUP BY cot_nhom;
```

## 27.4. Giữ nhóm có số lượng đạt điều kiện

```sql
SELECT
    cot_nhom
FROM ...
GROUP BY cot_nhom
HAVING COUNT(DISTINCT cot) >= 3;
```

## 27.5. Giữ cả đối tượng có số lượng bằng 0

```sql
SELECT
    A.ID,
    COUNT(B.ID) AS SO_LUONG
FROM A
LEFT JOIN B
    ON A.ID = B.A_ID
GROUP BY A.ID;
```

## 27.6. Tìm đối tượng cùng nhóm với một người

```sql
SELECT *
FROM GIAOVIEN
WHERE MABM = (
    SELECT MABM
    FROM GIAOVIEN
    WHERE MAGV = '002'
)
AND MAGV <> '002';
```

## 27.7. Tìm đối tượng giữ đồng thời hai vai trò

```sql
SELECT GV.*
FROM GIAOVIEN GV
WHERE GV.MAGV IN (
    SELECT TRUONGBM
    FROM BOMON
)
AND GV.MAGV IN (
    SELECT GVCNDT
    FROM DETAI
);
```

---

# 28. LEFT JOIN kết hợp GROUP BY

## Mẫu chuẩn

```sql
SELECT
    A.ID,
    A.TEN,
    COUNT(B.ID) AS SO_LUONG
FROM A
LEFT JOIN B
    ON A.ID = B.A_ID
GROUP BY
    A.ID,
    A.TEN;
```

## Ví dụ

Số người thân của từng giáo viên:

```sql
SELECT
    GV.MAGV,
    GV.HOTEN,
    COUNT(NT.TEN) AS SO_NGUOI_THAN
FROM GIAOVIEN GV
LEFT JOIN NGUOITHAN NT
    ON GV.MAGV = NT.MAGV
GROUP BY
    GV.MAGV,
    GV.HOTEN;
```

---

# 29. DISTINCT trước khi đếm mối quan hệ nhiều dòng

Bảng `THAMGIADT` có thể chứa nhiều công việc của cùng một giáo viên trong cùng một đề tài.

Đếm số đề tài:

```sql
COUNT(DISTINCT TG.MADT)
```

Đếm số giáo viên tham gia đề tài:

```sql
COUNT(DISTINCT TG.MAGV)
```

Không nên dùng:

```sql
COUNT(*)
```

nếu một người có thể xuất hiện nhiều dòng trong cùng một đề tài.

---

# 30. Kiểm tra lỗi nhanh

## Lỗi 1: Quên điều kiện JOIN

Sai:

```sql
FROM GIAOVIEN GV, BOMON BM
```

nếu không có điều kiện liên kết sẽ tạo tích Descartes.

Đúng:

```sql
FROM GIAOVIEN GV
JOIN BOMON BM
    ON GV.MABM = BM.MABM
```

---

## Lỗi 2: Dùng WHERE với hàm kết tập

Sai:

```sql
WHERE COUNT(*) >= 3
```

Đúng:

```sql
HAVING COUNT(*) >= 3
```

---

## Lỗi 3: Thiếu cột trong GROUP BY

Mọi cột không được tổng hợp trong `SELECT` phải nằm trong `GROUP BY`.

---

## Lỗi 4: Dùng INNER JOIN làm mất đối tượng có số lượng 0

Muốn giữ đối tượng chưa có dữ liệu liên quan, dùng:

```sql
LEFT JOIN
```

---

## Lỗi 5: Đếm sai vì dữ liệu trùng

Dùng:

```sql
COUNT(DISTINCT cot)
```

---

## Lỗi 6: So sánh NULL bằng dấu bằng

Sai:

```sql
WHERE cot = NULL
```

Đúng:

```sql
WHERE cot IS NULL
```

---

## Lỗi 7: Truy vấn con trả nhiều dòng nhưng dùng dấu bằng

Sai:

```sql
WHERE MAGV = (
    SELECT MAGV
    FROM THAMGIADT
)
```

Đúng:

```sql
WHERE MAGV IN (
    SELECT MAGV
    FROM THAMGIADT
)
```

---

# 31. Bảng nhận diện từ khóa trong đề

| Từ khóa trong đề                | Lệnh thường dùng            |
| ------------------------------- | --------------------------- |
| Cho biết, hiển thị              | `SELECT`                    |
| Có điều kiện                    | `WHERE`                     |
| Và                              | `AND`                       |
| Hoặc                            | `OR`, `UNION`               |
| Bắt đầu bằng                    | `LIKE N'...%'`              |
| Chứa                            | `LIKE N'%...%'`             |
| Trong khoảng                    | `BETWEEN` hoặc `>=` và `<=` |
| Thuộc nhiều giá trị             | `IN`                        |
| Thông tin từ nhiều bảng         | `JOIN`                      |
| Mỗi đối tượng, kể cả chưa có    | `LEFT JOIN`                 |
| Cùng bảng, hai vai trò          | Self join                   |
| Số lượng                        | `COUNT`                     |
| Tổng                            | `SUM`                       |
| Trung bình                      | `AVG`                       |
| Cao nhất                        | `MAX`                       |
| Thấp nhất                       | `MIN`                       |
| Theo từng                       | `GROUP BY`                  |
| Có số lượng từ ... trở lên      | `HAVING`                    |
| Không trùng                     | `DISTINCT`                  |
| Sắp xếp                         | `ORDER BY`                  |
| Thuộc một trong hai tập         | `UNION`                     |
| Thuộc cả hai tập                | `INTERSECT`                 |
| Thuộc tập A nhưng không thuộc B | `EXCEPT`                    |
| Có ít nhất một                  | `EXISTS`                    |
| Không tồn tại                   | `NOT EXISTS`                |

---

# 32. Công thức ghi nhớ trước kiểm tra

## Truy vấn cơ bản

```sql
SELECT cot
FROM bang
WHERE dieu_kien;
```

## Kết bảng

```sql
SELECT ...
FROM A
JOIN B
    ON A.KHOA = B.KHOA;
```

## Gom nhóm

```sql
SELECT
    cot_nhom,
    COUNT(cot) AS SO_LUONG
FROM bang
GROUP BY cot_nhom;
```

## Lọc nhóm

```sql
SELECT
    cot_nhom
FROM bang
GROUP BY cot_nhom
HAVING COUNT(cot) >= n;
```

## Giữ số lượng bằng 0

```sql
SELECT
    A.ID,
    COUNT(B.ID)
FROM A
LEFT JOIN B
    ON A.ID = B.A_ID
GROUP BY A.ID;
```

## Đếm không trùng

```sql
COUNT(DISTINCT cot)
```

## Tìm cùng nhóm

```sql
WHERE cot_nhom = (
    SELECT cot_nhom
    FROM bang
    WHERE dieu_kien_doi_tuong
)
```

---

# 33. Checklist làm bài

- [ ] Xác định chính xác cột cần hiển thị.
- [ ] Xác định bảng chứa từng cột.
- [ ] Viết đúng đường liên kết giữa các bảng.
- [ ] Dùng `WHERE` để lọc dòng.
- [ ] Dùng `GROUP BY` khi có hàm kết tập và cột thường.
- [ ] Dùng `HAVING` để lọc kết quả sau gom nhóm.
- [ ] Dùng `LEFT JOIN` nếu cần giữ đối tượng không có dữ liệu liên quan.
- [ ] Dùng `DISTINCT` nếu phép kết tạo kết quả trùng.
- [ ] Dùng `COUNT(DISTINCT ...)` khi đếm đối tượng khác nhau.
- [ ] Dùng `N'...'` cho chuỗi tiếng Việt trong SQL Server.
- [ ] Dùng ngoặc khi kết hợp `AND` và `OR`.
- [ ] Đặt `ORDER BY` ở cuối truy vấn.
