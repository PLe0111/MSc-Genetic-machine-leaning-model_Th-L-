# MSc-Genetic-machine-leaning-model_Le
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c184ae23-b1ba-49c0-9bee-51dae31e57aa" />
## Mục tiêu của đề tài

* Mục tiêu chung: Xây dựng mô hình học máy hỗ trợ chẩn đoán ung thư đại trực tràng

* Đối tượng nghiên cứu: Các miRNA được tách chiết từ huyết tương sau khi phân lập từ máu ngoại vi của các bệnh nhân.

* Mục tiêu cụ thể:

- Xác định các miRNA tiềm năng trong chẩn đoán ung thư đại trực tràng.
- Xây dựng mô hình học máy hỗ trợ chẩn đoán các ung thư đại trực tràng.
- Xác định các tương tác miRNA-mRNA và các con đường sinh học liên quan đến các miRNA trong mô hình học máy.

## 1. Thu nhập, xử lý thông tin lâm sàng và preprocess-normalization các database GEO:

Thu thập dữ liệu về mức độ biểu hiện của miRNA và thông tin lâm sàng của bệnh nhân ung thư đại trực tràng từ cơ sở dữ liệu Gene Expression Omnibus – GEO (https://www.ncbi.nlm.nih.gov/geo/). Từ khóa được dùng để tìm kiếm các cơ sở dữ liệu gồm: "miRNA" AND “Homo sapiens” AND "Colorectal cancer" AND "blood" OR “plasma” OR “serum”.

Các dữ liệu được thu nhận để đưa vào quy trình xử lý cần có các tiêu chí sau:
- Dữ liệu thực nghiệm trên mẫu máu ngoại vi ở người.
- Tệp dữ liệu chứa cả số lượng mẫu bệnh ung thư (có chứa mẫu ung thư đại trực tràng) và mẫu không ung thư.
- Các tập dữ liệu thu nhận có ít nhất từ 40 mẫu ở mỗi phân loại nhóm mẫu.
- Ưu tiên các tập dữ liệu thực hiện đánh giá trên quần thể Á Đông (Nhật Bản, Hàn Quốc, Trung Quốc, Đài Loan) vì có sự tương đồng về di truyền và lối sống với người Việt Nam

Các dữ liệu bị loại trừ trong quá trình thu nhận dữ liệu:
- Không lấy dữ liệu đánh giá trên mẫu mô, các nhóm bệnh ung thư khác hoặc di căn từ cơ quan khác.
- Không thu nhận dữ liệu đánh giá trên động vật, tế bào ung thư.
- Không lấy dữ liệu giải mẫu bộ gen DNA.

* Tiền xử lý và chuẩn hóa dữ liệu

Các bộ dữ liệu thu nhận sử dụng các công nghệ khác nhau để phân tích nên cần phải loại bỏ các mẫu bị lặp, chuyển đổi tín hiệu từ probe ID sang gene symbol dựa vào file platform annotation. Mẫu trong các tệp dữ liệu được phân loại thành nhóm không ung thư/khỏe mạnh (no-cancer/healthy) và các nhóm của các loại ung thư khác nhau. Sau đó, các tập dữ liệu sẽ được chuẩn hóa (Normalization) bằng cách sử dụng hàm "normalizeBetweenArrays" trong gói phần mềm R “limma”. Các tập dữ liệu GSE sau khi chuẩn hóa sẽ được gộp lại và thực hiện loại bỏ hiệu ứng theo lô (Batch effect) bằng cách sử dụng gói “sva” (v3.5.0) để chia thành 2 nhóm dữ liệu tập huấn (training) và kiểm tra (testing) theo tỷ lệ 70:30 để đảm bảo đánh giá tính tổng quá hóa của mô hình [1]. Nhóm dữ liệu training giúp xác định các dấu ấn sinh học chẩn đoán tiềm năng, trong khi nhóm dữ liệu testing dùng để xác nhận kết quả phân tích.

--> Loại data em download từ GEO để chạy code là file SOFT (Simple Omnibus Format in Text). Đây là thông tin lâm sàng của các tập GSE mà em quan tâm và đã sử dụng để thực hiện preprocess:

    - GSE113486: no-cancer control = 100 samples; CRC=40 samples
link GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE113486 

    - GSE211692: no-cancer control = 5643 samples; CRC = 1596 samples
link GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE211692 

    - GSE106817: no-cancer control = 2759 samples; CRC = 115 samples 
link GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE106817 

--> Cả 3 GSE trên đều thực hiện trên mẫu serum của người Nhật và cùng 1 platform GPL21263 - 3D-Gene Human miRNA V21_1.0.0. Các tập dữ liệu GSE sau khi chuẩn hóa (Normalization) sẽ được gộp lại và thực hiện Batch effect bằng cách sử dụng gói “sva” (v3.5.0) để chia thành 2 tapaj dữ liệu tập huấn (training) và kiểm tra (testing) theo tỷ lệ 70:30 để đảm bảo đánh giá tính tổng quá hóa của mô hình
-->sau khi gộp và batch effect thì chia ra trainig: testing (internal validation)= 70:30 
	Tập Train: (7177, 2565) | Tập Test: (3076, 2565)

--> Tập dữ liệu external validation em preprocess, normalization và gộp 2 tập GSE dưới đây cũng thực hiện trên mẫu serum người Nhật và cùng platform GPL21263 - 3D-Gene Human miRNA V21_1.0.0:

    - GSE113740: no-cancer control = 969 samples; CRC = 25 samples 
link GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE113740 

    - GSE112264: no-cancer control = 41 samples; CRC = 50 samples
link GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE112264 

### --> thắc mắc việc chia các tập dữ liệu training-validation-testing:

workflow thực hiện ở hình 1 em đã được thầy hướng dẫn chỉnh sửa và đồng ý. Tuy nhiên, trong quá trình tìm hiểu và xử lý dữ liệu ra được các miRNA tiềm năng thì em vẫn có những khúc mắc về cách chia tập dữ liệu (Data Splitting Strategy):

- Em đánh giá final model trên tập external validation độc lập như vậy thì cách làm này có tránh bị data leakage không và có đánh giá đúng được khả năng học hay khả năng phân loại mẫu của model dựa trên dữ liệu mức độ biểu hiện của miRNA không ạ?

Vì khi lựa database độc lập để external validation như vậy thì số samples của tập data được gộp em đang thấy không có cơ sở lựa chọn phù hợp so với 2 tập training-testing để đánh giá final model.
- Em tìm hiểu thì có thấy việc chia training-validation-testing từ 1 full database đã gộp và xử lý batch effect thì em nên thay đổi cách chia dữ liệu sau khi preprocess các GSE hay ở bước nào em thực hiện chia tập cho hợp lý ạ?
- Hiện tại pipline code em chạy ra đang thực hiện huấn luyện và đánh giá model trên serum của người Nhật, nhưng nếu em lựa chọn database của quần thể dân tộc khác với platform khác để làm external validation thì nó có thực sự đánh giá khả năng phân loại của model trên data của nhiều hơn 2 quần thể người khác nhau không ạ? 		(do đây cũng là câu hỏi em đã từng nhận được từ hội đồng rằng nếu ứng dụng model đã xây dựng để ứng dụng cho bệnh nhân người Việt thì Cơ sở lý luận cho việc sử dụng dữ liệu nghiên cứu quốc tế để lựa chọn biomarker miRNA cho việc sàng lọc trên quần thể người Việt Nam?)
- Cuối tháng 6/2026, thầy hướng dẫn có định hướng cho em preprocess database của miRNA trên mẫu mô (tissue) (trên cơ sở TCGA hoặc GEO hoặc cBioPortal) với thông tin lâm sàng là survival hoặc về treatment và sau đó so sánh tìm các miRNA giao thoa với bộ data circulating miRNA trên mẫu máu ngoại vi (blood/serum/plasma) ở bước DE-feature selection rồi mới đưa vào chạy model để cuối cùng có thể kết luận các miRNA tiềm năng được chọn không những có sự khác biệt giữa CRC-no cancer mà còn có thể đánh giá khả năng tiên lượng, vì thầy thấy nếu chỉ làm circulation miRNA trong hệ máu ngoại vi thì vẫn cần thực nghiệm để đánh giá. Tuy nhiên, em tìm hiểu biểu hiện của miRNA trong mẫu mô và máu ngoại vi khác nhau do cơ chế xuất nhập bào của miRNA giữa các tê bào; và việc lựa chọn input data từ 2 loại mẫu khác nhau em nghĩ sẽ dẫn đến mục tiêu của xây dựng model khác nhau. Theo anh thì em nên phân tích databáe của miRNA trên mô kết hợp với database của circular miRNA như thế nào để xây dựng mô hình hợp lý?

## 2. Phân tích mức độ biểu hiện khác biệt (Difference Expression Gene – DEG):

Mức độ biểu hiện miRNA giữa nhóm mẫu bệnh và nhóm chứng có sự khác nhau nên sẽ được phân tích dựa trên phân tích biểu hiện khác biệt được thực hiện để tìm các miRNA được biểu hiện khác biệt (Difference Expression – DE) trong các điều kiện khác nhau.

Cụ thể, sự biểu hiện khác biệt của các miRNA sẽ được so sánh giữa nhóm CRC – không ung thư, nhóm có u tuyến – không ung thư, nhóm CRCs – nhóm u tuyến. Kiểm định unpaired t-test với phương pháp limma sẽ đưa một mô hình tuyến tính phù hợp với dữ liệu đã được chuẩn hóa và sau đó sử dụng phương pháp Bayes thực nghiệm để điều chỉnh các lỗi chuẩn của các thay đổi log-fold ước tính [2,3]. DEGs của các miRNA được xác định với Log Fold Change: |log2 FC| ≥ 1; adj-p (adjust p-value) theo phương pháp False Discovery Rate: (FDR) < 0.05 [3]. Để trực quan hóa, các gói "dplyr", "pheatmap" và "ggplot2" sẽ được sử dụng để tạo bản đồ nhiệt (heatmap) và biểu đồ núi lửa (volcano plot). 

Các dữ liệu được đưa vào phân tích sẽ có sự khác nhau về cỡ mẫu, quy trình thực nghiệm, dữ liệu kết quả thu được và cường độ tín hiệu liên quan đến từng miRNA khác nhau do sử dụng các công nghệ phân tích khác nhau. Vì thế việc sử dụng trực tiếp dữ liệu mà không qua xử lý sẽ dẫn đến hiệu quả kém khi thực hiện xây dựng mô hình học máy nên dữ liệu sau tiền xử lý và phân tích DEGs sẽ được chuẩn hóa bằng phương pháp Standardization [4]. Sau khi chuẩn hóa, mỗi tập dữ liệu sẽ có một đặc tính với giá trị trung bình bằng 0 và độ lệch chuẩn bằn 1 trên toàn bộ tập dữ liệu. Tất cả các đặc tính (ký hiệu là x) sẽ được chuẩn hóa theo công thức:
<img width="83" height="61" alt="image" src="https://github.com/user-attachments/assets/8547cf5e-e7b8-4587-beb5-a7ccfeb6ea93" />
(1)
trong đó μ và σ là giá trị trung bình và độ lệch chuẩn của đặc tính x được tính toán trên tất cả các mẫu trong tập dữ liệu. Giá trị chuẩn hóa của x là z [4]. Bên cạch đó, để tìm ra các miRNA có giá trị khác biệt giữa nhóm bệnh và nhóm đối chứng của mỗi tệp dữ liệu thì giá trị p-value giữa hai nhóm bệnh và đối chứng được xác định. Khi p-value giảm, ý nghĩa thống kê tăng lên. Do đó, trong mỗi tệp dữ liệu các miRNA được sắp xếp theo thứ tự tăng dần dựa trên p-value của chúng [4]. Sau khi xử lý và chuẩn hóa dữ liệu, số liệu về mức độ biểu hiện của miRNA trong nhóm mẫu không ung thư/khỏe mạnh với nhóm CRCs sẽ được so sánh để đưa ra giá trị Mean cho mức độ biểu hiện miRNA giữa các nhóm thông qua kiểm nghiệm tham số, phi tham số.

## --> Thắc mắc về chỉ số Z-score:

Khi em được hướng dẫn vẽ workflow Hình 1 vào tháng 3/2026 thì thầy hướng dẫn và chị NCS phụ trách pipline code có mẫu thuẫn việc sử dụng Z-score sau khi đã thực hiện DE hay trước khi gộp+batch effect các GSE. Trong pipline code em chạy em tham khảo các pipline của các labmate thì các bạn đang chạy chuẩn hóa bằng Quantile bằng gói limma của R và thực hiện Z-score sau khi chạy DE.
--> Nên em thắc mắc về sự khác biệt của phương pháp Quantile với Zscore và có cần thiết chạy Z-score sau khi normalizeation các GSE không ạ? Nếu cần chạy Z-score thì em nên thực hiện ở bước nào ạ?

## 3. Lựa chọn đặc trưng (Feature Selection):

Lựa chọn đặc trưng chủ yếu được áp dụng cho dữ liệu đa chiều; nói một cách đơn giản, lựa chọn đặc trưng là một kỹ thuật giảm chiều dữ liệu. Phương pháp Least Absolute Shrinkage and Selection Operator (LASSO), một phân tích hồi quy, thường được sử dụng để cải thiện độ chính xác của dự đoán. Nó thuộc họ mô hình hồi quy tuyến tính và sử dụng xác thực chéo mười lần mặc định. Phân tích hồi quy LASSO đã được sử dụng rộng rãi trong các nghiên cứu để sàng lọc các yếu tố chẩn đoán hoặc tiên lượng [5]. Bên cạnh đó, nhóm sẽ thực hiện thêm thuật toán Recursive Feature Addition (RFA) để đưa ra các miRNA có các dữ liệu đặc trưng và quan trọng nhất ở các nhóm dữ liệu mẫu chứng và mẫu CRCs. 

Để chọn ra những DEG tốt nhất để phát triển bộ phân loại dựa trên gen cho quá trình tiến hóa của u tuyến-CRC, thuật toán Boruta và Stepwise Regression được sử dụng tuần tự [6]. Thuật toán Boruta có trong gói ‘Boruta’ của ngôn ngữ lập trình R; sử dụng phương pháp bao bọc được xây dựng xung quanh bộ phân loại Rừng ngẫu nhiên (Random Forest) để thực hiện lựa chọn đặc trưng có giám sát và mạnh mẽ. Boruta dựa trên cùng một ý tưởng tạo thành cơ sở của bộ phân loại Rừng ngẫu nhiên. Thuật toán Random Forest dựa trên các nguyên tắc bagging (Bootstrap Aggregating) và lựa chọn đặc trưng ngẫu nhiên, giúp giảm phương sai của mô hình và tránh hiện tượng quá khớp. Trong quá trình xây dựng mỗi cây, rừng ngẫu nhiên chọn một tập hợp con các đặc trưng ngẫu nhiên tại mỗi điểm phân chia. Quá trình này, được gọi là Feature Bagging, đảm bảo rằng các cây không tương quan, làm giảm thêm nguy cơ quá khớp [7]

Boruta khởi tạo quá trình bằng cách tạo ra các đặc điểm bóng. Đây là các bản sao của các đặc điểm gốc với các giá trị được xáo trộn ngẫu nhiên, hoạt động như các đặc điểm nhiễu. Bước này rất quan trọng vì nó cung cấp một đường cơ sở để so sánh ý nghĩa của các đặc điểm gốc với nhiễu [7]. Thuật toán huấn luyện một bộ phân loại rừng ngẫu nhiên trên tập dữ liệu mở rộng, bao gồm cả đặc điểm gốc và đặc điểm bóng. Ý nghĩa của mỗi đặc điểm được đo bằng cách sử dụng mức giảm trung bình của chỉ số Gini hoặc bất kỳ số liệu phù hợp nào khác do rừng ngẫu nhiên cung cấp. Điểm ý nghĩa của các đặc điểm gốc được so sánh với điểm ý nghĩa cao nhất của các đặc điểm bóng [7]. Nếu một đặc điểm gốc có điểm ý nghĩa cao hơn đáng kể so với đặc điểm bóng tốt nhất, thì nó được coi là có ý nghĩa. Ngược lại, nếu một đặc điểm gốc có điểm ý nghĩa thấp hơn, thì nó được coi là không có ý nghĩa. Các đặc điểm được xác định là không có ý nghĩa sau đó sẽ bị loại khỏi tập dữ liệu. Quá trình này được lặp lại liên tục cho đến khi đạt được điều kiện dừng được xác định trước, chẳng hạn như số lần lặp tối đa hoặc tính ổn định trong việc lựa chọn đặc điểm. Sau khi hoàn tất các lần lặp, các đặc điểm được phân loại thành ba nhóm: có ý nghĩa đã xác nhận, không có ý nghĩa đã xác nhận và tạm thời. Các đặc điểm tạm thời cần được phân tích thêm để xác định ý nghĩa của chúng [7]. 

## 4.  Lựa chọn và thiết lập mô hình học máy:

Để xác định các mối liên kết và mẫu giữa tập hợp các biến đầu vào, xác minh xem biểu hiện miRNA có thể mô tả giữa hai nhóm bệnh và nhóm đối chứng hay không, nhóm nghiên cứu thực hiện các thuật toán theo phương pháp chính: học có giám sát (Under Supervised Learning). Phương pháp học có giám sát (Under Supervised Learning) được sử dụng cho xây dựng học máy để phân loại và kết quả trả ra là có hoặc không hoặc nhị phân 0-1 như (tốt hoặc xấu) để có được kết quả đa lớp như (ví dụ: cây, bụi cây hoặc cỏ). Hồi quy (Regression) một phần phụ khác của loại học có giám sát, trong đó các thuật toán được sử dụng trong loại này luôn tạo ra một giá trị số. Phân cụm (Clustering) là một phân của phương pháp học có giám sát với nhiệm vụ là nhóm các tập dữ liệu chưa được gắn nhãn, chính xác hơn, đó là một phương pháp nhóm các điểm dữ liệu thành các cụm riêng biệt dựa trên sự tương đồng của chúng [8].

Các thuật toán được sử dụng để xây dựng mô hình học máy gồm: Hồi quy tuyến tính (Linear Regression), Hồi quy Logistic (Logistic Regression),  Máy hỗ trợ vectơ (Support Vector Machine), Naïve Bayes, K-Means Clustering, K-Nearest Neighbors, Decision Tree, Thuật toán giảm chiều dữ liệu (Dimensionality Reduction Algorithms), Artificial Neural Networks.

### --> Thắc mắc về việc lựa chọn model phù hợp:

Các model em kể ở phần nội dung trên và 7 model em thực hiện trong pipline code là các model mà lab đang thực hiện và em tham khảo để chạy. Tuy nhiên cách xya dựng model hiện tại em được hướng dẫn là chạy các model có thể và cho ra model có các chỉ số tối ưu nhất làm final model. Bên cạnh đó, các kết quả model trả ra trong pipline em chạy thì em không ra được các dữ liệu phân biệt sư thay đổi biểu hiện của miRNA giữa mẫu CRC với No Cancer

--> Vì thế em thắc mắc sau khi em phân tích DE và Feature Selection, em có được danh sách 11 miRNA tiềm năng với giá trị logFC và  adj.P.Val, làm thế nào để em có thể đưa ra được các tiêu chí để lựa chọn các model để xây dựng-huấn luyện không ạ?

## 5. Đánh giá mô hình học máy

Khi dữ liệu được thu thập và chuẩn bị một cách thích hợp, thì cần phải chọn một mô hình phù hợp. Không phải mọi thuật toán học máy đều phù hợp với mọi vấn đề. Các mô hình ML cần được xem xét hoặc lựa chọn cuối cùng được xác định bởi khối lượng và bản chất của dữ liệu. Để kiểm tra mô hình học máy đã huấn luyện thì tập dữ liệu kiểm tra (Testing) được sử dụng để kiểm tra mô hình nhằm hiểu rõ hơn về hiệu suất của mô hình. 

Độ tin cậy của mô hình được kiểm chứng bằng phương pháp Kiểm chứng chéo [9]. Kiểm chứng chéo (Cross-Validation) có tầm quan trọng to lớn trong việc xây dựng mô hình học máy bằng cách so sánh, lựa chọn các mô hình phù hợp và đánh giá hiệu suất dự báo của chúng. Quá trình này không chỉ giới hạn ở giai đoạn huấn luyện mô hình mà còn bao gồm việc đánh giá hiệu suất của mô hình trên tập dữ liệu mới (dữ liệu thử nghiệm) [10]. Kiểm chứng chéo K-fold dễ xây dựng, dễ điều chỉnh và ít phụ thuộc vào cấu trúc của mô hình. Phương pháp Kiểm chứng chéo K-fold chia tập dữ liệu thành các tập con k-so sánh được hoặc nếp gấp (k-comparable subsets or folds). Việc gấp phân vùng (partition folding) được huấn luyện và kiểm tra trong K lần lặp, và phần còn lại của mô hình gấp K-1 được kiểm tra và huấn luyện bằng phương pháp gấp. Các giá trị K phổ biến nhất là 5 và 10 [10]. 

Trong nghiên cứu này sẽ áp dụng kiểm chứng chéo k- fold [9, 11]; cụ thể đầu tiên, tập dữ liệu được chi thành k phần và mô hình được áp dụng cho k phần đó. Sau đó, trên k mẫu con, một mẫu con duy nhất được giữ lại làm dữ liệu xác thực để kiểm tra mô hình, và k-1 mẫu con còn lại được sử dụng làm dữ liệu huấn luyện. Quá trình xác thực chéo sau đó được lặp lại nhiều lần, với mỗi mẫu con trong k mẫu con được sử dụng một lần làm dữ liệu xác thực [9]. Ở giai đoạn sau, giá trị trung bình thu được từ các mô hình được đánh giá theo phương pháp xác thực chéo.

Mô hình học máy trước khi được đưa vào ứng dụng thực tế thì cần được thực hiện thử nghiệm trên tập dữ liệu mà mô hình chưa được huấn luyện hoặc kiểm tra (External Validation) để đo lường hiệu suất khách quan, nhằm đảm bảo mô hình không chỉ hoạt động tốt trên dữ liệu đã được sử dụng trong quá trình huấn luyện và kiểm tra mà còn có khả năng dự đoán tốt trên dữ liệu mới chưa qua huấn luyện. Hiệu suất của mô hình phân loại được đánh giá bằng các thông số sau [9]:

* Diện tích dưới đường cong (AUC) là thước đo khả năng của bộ phân loại để phân biệt giữa các lớp và được sử dụng làm bản tóm tắt của đường cong ROC, được ước tính bởi hàm 'multiclass.roc' từ gói 'pROC' [12] trong R.

* Độ nhạy (Sensitivity – Se) là tỷ lệ phần trăm tổng số quan sát có liên quan thực sự được thu thập [9]: <img width="216" height="73" alt="image" src="https://github.com/user-attachments/assets/2c662a8f-23c3-45de-ac98-20f1c527f6fb" />

* Độ đặc hiệu (Specificity – Sp) đo lường số lượng kết quả âm tính thực tế được dự đoán là âm tính [13]: <img width="241" height="82" alt="image" src="https://github.com/user-attachments/assets/225516c6-8725-4db0-bf54-bf3948cc5b8a" />

* Độ chính xác (Accuracy – Acc) tỷ lệ kết quả đúng trong tổng số trường hợp được kiểm tra [9]: <img width="275" height="77" alt="image" src="https://github.com/user-attachments/assets/84ff953a-5881-4980-a9e1-c7f1cc9976f7" />
TPi = True Positive; FPi = False Positive; TNi = True Negative; FNi = False Negative lần lượt là dương tính thật, âm tính thật, dương tính giả và âm tính giả trong bài toán phân loại với N lớp, độ chính xác cho lớp i [9]

* Hệ số tương quan Matthews (Matthews Correlation Coefficient – MCC) đo lường chất lượng phân loại nhị phân [13]: <img width="560" height="71" alt="image" src="https://github.com/user-attachments/assets/960687af-55eb-426f-9b07-b80e8b365e24" />

* Giá trị dự đoán dương tính (Positive Predictive Value – PPV) đo lường tỷ lệ dự đoán dương tính được dự đoán đúng [13]:<img width="245" height="62" alt="image" src="https://github.com/user-attachments/assets/2da707f1-93fc-4170-bf72-9026852e4f4c" />

* Độ chính xác (Precision) là tỷ lệ các quan sát có liên quan trong số các quan sát được thu thập [9]: <img width="171" height="66" alt="image" src="https://github.com/user-attachments/assets/1aab095b-7cef-47db-a004-86dee52a330c" />

* F1 score là giá trị trung bình hài hòa của độ chính xác và độ nhạy [9]:<img width="223" height="65" alt="image" src="https://github.com/user-attachments/assets/8075dd91-c419-4142-8244-9187ac926077" />

Acc, Se, Sp và PPV đều trả về giá trị phần trăm từ 0 đến 100, trong đó số càng cao biểu thị dự đoán càng chính xác. MCC thường được sử dụng trong học máy và được coi là một thước đo cân bằng, đồng thời là một trong những thước đo hữu ích nhất của bộ phân loại nhị phân. MCC trả về giá trị từ -1 đến 1. Hệ số +1 biểu thị dự đoán hoàn hảo, 0 biểu thị dự đoán ngẫu nhiên trung bình và -1 biểu thị nghịch đảo [13]. PPV cũng là một công cụ hữu ích để phân tích các công cụ dự đoán miRNA vì việc xác minh thực nghiệm các dự đoán này có thể rất khó khăn và tốn thời gian, nhưng nếu công cụ đó có PPV rất cao thì người dùng có thể tin tưởng hơn vào dự đoán đó [13]. Hiệu suất của mô hình được xác thực bằng cách sử dụng các tập dữ liệu xác thực đọc lập (External Validation). Mô hình có AUC (Diện tích dưới đường cong) trung bình cao nhất trên các dữ liệu huấn luyện và xác thực là mô hình tối ưu

Reference paper:
[1] Joseph, V.R.: Optimal ratio for data splitting. Statistical Analysis and Data Mining: The ASA Data Science Journal. 15, 531–538 (2022). https://doi.org/10.1002/sam.11583 

[2] Lacalamita, A., Piccinno, E., Scalavino, V., Bellotti, R., Giannelli, G., Serino, G.: A Gene-Based Machine Learning Classifier Associated to the Colorectal Adenoma—Carcinoma Sequence. Biomedicines. 9, 1937 (2021). https://doi.org/10.3390/biomedicines9121937 

[3] Smyth, G.K.: Linear models and empirical bayes methods for assessing differential expression in microarray experiments. Stat Appl Genet Mol Biol. 3, Article3 (2004). https://doi.org/10.2202/1544-6115.1027

[4] Fahami, M.A., Roshanzamir, M., Izadi, N.H., Keyvani, V., Alizadehsani, R.: Detection of effective genes in colon cancer: A machine learning approach. Informatics in Medicine Unlocked. 24, 100605 (2021). https://doi.org/10.1016/j.imu.2021.100605

[5] Alzubi, J., Nayyar, A., Kumar, A.: Machine Learning from Theory to Algorithms: An Overview. J. Phys.: Conf. Ser. 1142, 012012 (2018). https://doi.org/10.1088/1742-6596/1142/1/012012

[6] Lacalamita, A., Piccinno, E., Scalavino, V., Bellotti, R., Giannelli, G., Serino, G.: A Gene-Based Machine Learning Classifier Associated to the Colorectal Adenoma—Carcinoma Sequence. Biomedicines. 9, 1937 (2021). https://doi.org/10.3390/biomedicines9121937

[7] Hamidi, F., Gilani, N., Kazemnejad, A., Aftabi, Y., Shirforoush-Sattari, M., Jahanimoghadam, A.: Decision tree-based machine learning methods for identifying colorectal cancer-associated microRNA signatures and their regulatory networks. Sci Rep. 15, 34700 (2025). https://doi.org/10.1038/s41598-025-17037-7

[8] Solomon, D.D., Sonia, Kumar, K., Kanwar, K., Iyer, S., Kumar, M.: Extensive Review on the Role of Machine Learning for Multifactorial Genetic Disorders Prediction. Arch Computat Methods Eng. 31, 623–640 (2024). https://doi.org/10.1007/s11831-023-09996-9

[9] Lacalamita, A., Piccinno, E., Scalavino, V., Bellotti, R., Giannelli, G., Serino, G.: A Gene-Based Machine Learning Classifier Associated to the Colorectal Adenoma—Carcinoma Sequence. Biomedicines. 9, 1937 (2021). https://doi.org/10.3390/biomedicines9121937

[10] Qiu, J.: An Analysis of Model Evaluation with Cross-Validation: Techniques, Applications, and Recent Advances. AEMPS. 99, 69–72 (2024). https://doi.org/10.54254/2754-1169/99/2024OX0213

[11] Machine Learning-Based Prediction of the Genetic Background of Colorectal Liver Metastasis Using Mirna Expression Data. General Medicine.

[12] Robin, X., Turck, N., Hainard, A., Tiberti, N., Lisacek, F., Sanchez, J.-C., Müller, M.: pROC: an open-source package for R and S+ to analyze and compare ROC curves. BMC Bioinformatics. 12, 77 (2011). https://doi.org/10.1186/1471-2105-12-77

[13] Ikeoka, S.: Analysis of Machine Learning Based Methods for Identifying MicroRNA Precursors. Master’s Projects. (2009). https://doi.org/10.31979/etd.hvfw-ew3m
