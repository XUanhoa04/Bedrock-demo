# Bedrock Demo
# Bước 1: Khởi tạo S3 Bucket và Amazon Bedrock Knowledge Base
1.1 Chuẩn bị S3 Bucket (Nơi chứa file text của Database)  

1.2. tạo sẵn 3 file text (.txt hoặc .json) để test ghi thông tin 3 sản phẩm của bạn (ví dụ: sp1.txt, sp2.txt, sp3.txt) và nhấn Upload vào S3 này  

<img width="1495" height="603" alt="image" src="https://github.com/user-attachments/assets/bf850998-8cd5-4754-928d-8e4e733e3d7b" />  

1.3. Truy cập vào giao diện Amazon Bedrock  

1.4. Tạo Knowledge Base, chọn Knowledge Base With Vector Store  

1.5. Chọn data source type - S3  

1.6. Data Source: S3 URI: Nhấn nút Browse S3, trỏ vào bucket đã tạo ở 1.1.  

1.7. Chọn Embeddings model:Titan Embeddings V2 hoặc Titan Embeddings G1  

<img width="933" height="343" alt="image" src="https://github.com/user-attachments/assets/ad82052c-81a2-46a6-b48f-31cfadc0b20a" />  

1.8.  Chọn Vector store: Chọn mục đầu tiên Quick create a new vector store hoặc tạo sẵn trực tiếp từ S3   

1.9.  Nhấn Create, sau khi tạo xong thì sync  

2.0.  Test với Knowledge Base, chọn Nova 2 Lite, kết quả cho thấy AI trả lời có dẫn nguồn và chính xác:
<img width="915" height="304" alt="image" src="https://github.com/user-attachments/assets/ad264a6a-f6cd-4f14-8f47-44434d327d7e" />

### Bước 2: Thiết lập Quyền hạn (IAM Roles)
Role thực thi cho Lambda được đặt tên là `Lambda-Bedrock-Query-Role`.
* **Policies đính kèm:**
    1. `AWSLambdaVPCAccessExecutionRole`: Cho phép Lambda truy cập tài nguyên trong VPC.
    2. `Lambda-Bedrock-Query` (Inline Policy): Cho phép thao tác với Bedrock.
* **Chi tiết quyền (Permissions):**
    * `bedrock:RetrieveAndGenerate`: Truy vấn và tạo câu trả lời.
    * `bedrock:Retrieve`: Truy xuất dữ liệu từ Knowledge Base.
    * `bedrock:InvokeModel`: Gọi mô hình ngôn ngữ (Nova Lite).
    * `bedrock:GetInferenceProfile`: Đọc hồ sơ định tuyến cho model Nova.
<img width="1459" height="601" alt="image" src="https://github.com/user-attachments/assets/436a78bd-5456-40ed-b3ef-7537e3ae86b5" />

### Bước 3: Cấu hình Mạng (VPC Endpoint)
* **VPC Interface Endpoint:** Tạo endpoint cho dịch vụ `bedrock-agent-runtime` để Lambda gọi Bedrock qua mạng nội bộ của AWS mà không cần Internet.
* **Security Group:** Cấu hình cho phép cổng 443 (HTTPS) giữa Lambda và Endpoint.

### Bước 4: Triển khai Lambda Function
* **Runtime:** Node.js 20.x.
* **Mô hình sử dụng:** Amazon Nova Lite v1.0 (via Inference Profile).
* **Prompt Template:** Tối ưu hóa để AI hiểu các từ viết tắt như "ip15", "mb", "ap".

**Đoạn mã xử lý chính:**
```javascript
const modelArn = "arn:aws:bedrock:us-west-2:041627896542:inference-profile/us.amazon.nova-lite-v1:0";
const command = new RetrieveAndGenerateCommand({
    input: { text: userQuestion },
    retrieveAndGenerateConfiguration: {
        type: "KNOWLEDGE_BASE",
        knowledgeBaseConfiguration: {
            knowledgeBaseId: "AVHHNWOIBA",
            modelArn: modelArn,
            generationConfiguration: {
                promptTemplate: {
                    textPromptTemplate: "Bạn là trợ lý bán hàng shop mini_e. Dựa trên $search_results$, trả lời: $query$"
                }
            }
        }
    }
});
```
#### Kết quả:
```Event Json
{
  "question": "Giá của iPhone 15 Pro Max là bao nhiêu?"
}
```
<img width="1388" height="402" alt="image" src="https://github.com/user-attachments/assets/ba95ab92-cd5e-42a5-9525-0788e86632e0" />






      
      






