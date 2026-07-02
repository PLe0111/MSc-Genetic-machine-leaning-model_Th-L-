# MSc-Genetic-machine-leaning-model_Le
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c184ae23-b1ba-49c0-9bee-51dae31e57aa" />
##Mục tiêu của đề tài
    • Mục tiêu chung: Xây dựng mô hình học máy hỗ trợ chẩn đoán ung thư đại trực tràng 
    • Đối tượng nghiên cứu: Các miRNA được tách chiết từ huyết tương sau khi phân lập từ máu ngoại vi của các bệnh nhân.
    • Mục tiêu cụ thể:
    • Xác định các miRNA tiềm năng trong chẩn đoán ung thư đại trực tràng.
    • Xây dựng mô hình học máy hỗ trợ chẩn đoán các ung thư đại trực tràng.
    • Xác định các tương tác miRNA-mRNA và các con đường sinh học liên quan đến các miRNA trong mô hình học máy.
###1. Thu nhập, xử lý thông tin lâm sàng và preprocess-normalization các database GEO:
Thu thập dữ liệu về mức độ biểu hiện của miRNA và thông tin lâm sàng của bệnh nhân ung thư đại trực tràng từ cơ sở dữ liệu Gene Expression Omnibus – GEO (https://www.ncbi.nlm.nih.gov/geo/). Từ khóa được dùng để tìm kiếm các cơ sở dữ liệu gồm: "miRNA" AND “Homo sapiens” AND "Colorectal cancer" AND "blood" OR “plasma” OR “serum”.
Các dữ liệu được thu nhận để đưa vào quy trình xử lý cần có các tiêu chí sau:
    • Dữ liệu thực nghiệm trên mẫu máu ngoại vi ở người.
    • Tệp dữ liệu chứa cả số lượng mẫu bệnh ung thư (có chứa mẫu ung thư đại trực tràng) và mẫu không ung thư.
    • Các tập dữ liệu thu nhận có ít nhất từ 40 mẫu ở mỗi phân loại nhóm mẫu.
    • Ưu tiên các tập dữ liệu thực hiện đánh giá trên quần thể Á Đông (Nhật Bản, Hàn Quốc, Trung Quốc, Đài Loan) vì có sự tương đồng về di truyền và lối sống với người Việt Nam 
Các dữ liệu bị loại trừ trong quá trình thu nhận dữ liệu:
    • Không lấy dữ liệu đánh giá trên mẫu mô, các nhóm bệnh ung thư khác hoặc di căn từ cơ quan khác.
    • Không thu nhận dữ liệu đánh giá trên động vật, tế bào ung thư.
    • Không lấy dữ liệu giải mẫu bộ gen DNA.

           Tiền xử lý và chuẩn hóa dữ liệu
Các bộ dữ liệu thu nhận sử dụng các công nghệ khác nhau để phân tích nên cần phải loại bỏ các mẫu bị lặp, chuyển đổi tín hiệu từ probe ID sang gene symbol dựa vào file platform annotation. Mẫu trong các tệp dữ liệu được phân loại thành nhóm không ung thư/khỏe mạnh (no-cancer/healthy) và các nhóm của các loại ung thư khác nhau. Sau đó, các tập dữ liệu sẽ được chuẩn hóa (Normalization) bằng cách sử dụng hàm "normalizeBetweenArrays" trong gói phần mềm R “limma”. Các tập dữ liệu GSE sau khi chuẩn hóa sẽ được gộp lại và thực hiện loại bỏ hiệu ứng theo lô (Batch effect) bằng cách sử dụng gói “sva” (v3.5.0) để chia thành 2 nhóm dữ liệu tập huấn (training) và kiểm tra (testing) theo tỷ lệ 70:30 để đảm bảo đánh giá tính tổng quá hóa của mô hình [1]. Nhóm dữ liệu training giúp xác định các dấu ấn sinh học chẩn đoán tiềm năng, trong khi nhóm dữ liệu testing dùng để xác nhận kết quả phân tích.
Reference paper: [1] Joseph, V.R.: Optimal ratio for data splitting. Statistical Analysis and Data Mining: The ASA Data Science Journal. 15, 531–538 (2022). https://doi.org/10.1002/sam.11583 
