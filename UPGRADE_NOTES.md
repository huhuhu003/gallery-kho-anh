# Nâng cấp giao diện và tìm ảnh

Web chính vẫn là `index.html`, không cần backend hoặc API suy luận trả phí. Không thay đổi `demo-60fps.html`. Bản trước nâng cấp nằm trong `backups/`.

## Giao diện

- Điều hướng Kho ảnh / Tìm bằng ảnh / Chèn logo có trạng thái đang chọn và icon SVG đồng bộ.
- Nút Bộ lọc cạnh số kết quả; các nhãn bộ lọc cho phép bỏ từng điều kiện.
- Lưới ảnh, nút, ô nhập và hộp thoại thống nhất; hỗ trợ màn hình nhỏ và theme tối.
- Màn tìm có ảnh mẫu, khoanh vùng bằng kéo khung hoặc thanh điều chỉnh, tiến độ, nút dừng và thông báo lỗi rõ ràng.
- Chèn logo có áp dụng màu/cỡ cho tất cả ảnh, nền xem trước và vùng xuất ảnh dễ thấy. Nếu thiếu `logo-trong-suot.png`, có thể chọn logo PNG/WebP/SVG từ máy; file này chỉ được giữ trong phiên đang mở.
- Giữ nguyên hàm `revealTheme`, helper đóng băng logo, CSS pseudo-element View Transition và các hàm quản lý phiên Google. Chấm công và công cụ lấy ảnh được giữ lại.

## Tìm ảnh trên máy 4 GB RAM

Mặc định **Nhanh, tiết kiệm · CLIP**. Không tự tải cả ba mô hình.

- **CLIP:** tìm ảnh tương đồng trong hãng/danh mục đã chọn.
- **CLIP + DINOv2:** CLIP xếp hạng toàn bộ phạm vi; DINOv2 chấm lại tối đa 80 ứng viên, hoặc **24** nếu trình duyệt báo RAM ≤ 4 GB, CPU ≤ 4 luồng hay chế độ tiết kiệm dữ liệu. Kết hợp thứ hạng bằng reciprocal rank fusion; chưa có trọng số được hiệu chỉnh trên kho ảnh thực tế.
- **SigLIP 2:** chế độ độc lập để thử và so sánh, chỉ tải khi được chọn.
- **Ảnh gần trùng:** pHash + dHash, không tải mô hình; dùng cho ảnh cùng nguồn bị nén/đổi kích thước. Không xác nhận cùng sản phẩm ở góc chụp khác; ảnh cắt nhiều hoặc thêm chữ lớn vẫn có thể khó so khớp.

Suy luận chạy trong Web Worker với WASM một luồng, xử lý từng ảnh. Trên thiết bị thuộc nhóm tiết kiệm, ảnh đầu vào trung gian giới hạn cạnh dài 512 px, worker được tạo lại khi chuyển mô hình và giải phóng ngay khi tìm xong. Máy khác dùng tối đa 1024 px và giải phóng worker sau 60 giây rảnh; khi ẩn trang và không tìm kiếm cũng giải phóng worker.

Các vector lưu bằng IndexedDB, tách theo thư mục Drive, ID/phiên bản ảnh, mô hình, revision trọng số, kiểu lượng tử và tiền xử lý. Không dùng lẫn cache CLIP cũ. Nếu trình duyệt chặn cache hoặc hết dung lượng, vẫn tìm được nhưng lần sau có thể phải tính lại. Dừng tìm sẽ hủy tải ảnh và chấm dứt worker.

RAM/CPU do trình duyệt báo chỉ là tín hiệu chọn cấu hình, không phải giới hạn RAM cứng. Lần tìm đầu cần tải thư viện/trọng số, có thể hàng trăm MB tùy chế độ. Nên chọn hãng và danh mục sát nhất; hiệu năng còn phụ thuộc CPU, mạng và số ảnh. Đã bỏ thông báo cũ khẳng định tải mô hình chỉ ~25 MB và lần sau luôn tức thì.

## Điểm và đánh giá

Đã bỏ cách ép điểm lên tối thiểu 50%. Giao diện hiển thị điểm cosine, điểm gần trùng hoặc điểm của từng mô hình cùng thứ hạng. Đây không phải xác suất đúng.

Trong mục **Đánh giá độ chính xác trên ảnh mẫu**, nhập cùng tên ca thử khi thử các chế độ, chọn ảnh đúng đã biết và lưu. Ứng dụng thống kê tỷ lệ đúng đầu tiên / trong 5 kết quả đầu, cho xuất CSV. Dữ liệu giữ trên trình duyệt, tối đa 500 bản ghi, không lưu nội dung ảnh mẫu. Mỗi bản ghi lưu dấu nhận dạng phạm vi và giới hạn chấm lại để đối chiếu cấu hình. Chỉ so sánh công bằng khi dùng cùng ảnh mẫu, vùng cắt và phạm vi kho.

Chưa có bộ ảnh sản phẩm thật đã gán đáp án, nên chưa thể kết luận chế độ hai mô hình tăng độ chính xác bao nhiêu. Nếu DINOv2 lỗi, toàn bộ kết quả quay về CLIP và giao diện ghi rõ; ảnh không tải/chấm được được báo riêng. Bước chấm lại không thể cứu ảnh đúng nằm ngoài nhóm ứng viên CLIP đã chọn.

## Miễn phí và giấy phép

- [CLIP: MIT](https://github.com/openai/CLIP/blob/main/LICENSE).
- [DINOv2-small: Apache 2.0](https://huggingface.co/facebook/dinov2-small).
- [SigLIP 2: Apache 2.0](https://huggingface.co/google/siglip2-base-patch16-224).
- [Transformers.js: Apache 2.0](https://github.com/huggingface/transformers.js/blob/main/LICENSE), ghim phiên bản 3.8.1.

Đây là mã/thư viện và trọng số có giấy phép công khai; không có nghĩa mọi dữ liệu huấn luyện gốc đều được cung cấp. Không cần khóa API AI. Ảnh được phân tích trên thiết bị; máy vẫn cần truy cập Google Drive để lấy ảnh và CDN/Hugging Face để tải phần mềm/trọng số. Chi phí internet/hosting nếu có là riêng.

## Kiểm tra

`tests/browser_check.py` chạy Chromium với dữ liệu ảnh tổng hợp. Nó mô phỏng Google Identity và ảnh Drive để không cần tài khoản thật; mô hình CLIP, DINOv2 và SigLIP 2 chạy bằng trọng số thật.

Đã kiểm tra: bộ lọc, điều hướng, crop/reset, tìm gần trùng, lưu đánh giá, chọn logo riêng, áp dụng hàng loạt và tải ảnh; bố cục rộng 320/375/768/1024/1440 px; tọa độ logo sau đổi theme; suy luận cả ba mô hình; truy xuất kết hợp; cache; thông báo khi DINOv2 lỗi; dừng tìm và chế độ tiết kiệm.

Bài kiểm tra tiết kiệm mô phỏng thông tin RAM/CPU và giảm tốc CPU bằng Chromium DevTools, không giới hạn bộ nhớ tiến trình thật. Cần dùng thử trên máy 4 GB RAM và kho Drive thật để đo mức dùng RAM, thời gian tìm và độ chính xác sản phẩm.

Chạy kiểm tra nếu đã cài Python Playwright, Pillow và Chromium:

```powershell
python tests/browser_check.py --ui-only
python tests/browser_check.py
```

Script tự mở máy chủ HTTP trên localhost, đóng sau khi xong. Lần kiểm tra đầy đủ cần mạng và dung lượng để tải trọng số. Ảnh chụp trong `artifacts/` dùng sản phẩm tổng hợp; không phải ảnh trong kho của người dùng.
