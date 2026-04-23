## Bedrock Demo
# Bước 1: Khởi tạo S3 Bucket và Amazon Bedrock Knowledge Base
1.1 Chuẩn bị S3 Bucket (Nơi chứa file text của Database)  

1.2. tạo sẵn 3 file text (.txt hoặc .json) ghi thông tin 3 sản phẩm của bạn (ví dụ: sp1.txt, sp2.txt, sp3.txt) và nhấn Upload vào S3 này  

<img width="1495" height="603" alt="image" src="https://github.com/user-attachments/assets/bf850998-8cd5-4754-928d-8e4e733e3d7b" />  

1.3. Truy cập vào giao diện Amazon Bedrock  
1.4. Tạo Knowledge Base, chọn Knowledge Base With Vector Store
1.5. Chọn data source type - S3
1.6. Data Source: S3 URI: Nhấn nút Browse S3, trỏ vào bucket đã tạo ở 1.1.  
1.7. Chọn Embeddings model:Titan Embeddings V2 hoặc Titan Embeddings G1  

<img width="933" height="343" alt="image" src="https://github.com/user-attachments/assets/ad82052c-81a2-46a6-b48f-31cfadc0b20a" />  

1.8.  Chon Vector store: Chọn mục đầu tiên Quick create a new vector store hoặc tạo sẵn trực tiếp từ S3  
1.9.  Nhấn Create, sau khi tạo xong thì sync
2.0.  Test với Knowledge Base, chọn Nova 2 Lite, kết quả cho thấy AI trả lời có dẫn nguồn và chính xác:
<img width="915" height="304" alt="image" src="https://github.com/user-attachments/assets/ad264a6a-f6cd-4f14-8f47-44434d327d7e" />








      
      






