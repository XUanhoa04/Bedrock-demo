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

1.8.  Chon Vector store: Chọn mục đầu tiên Quick create a new vector store hoặc tạo sẵn trực tiếp từ S3   

1.9.  Nhấn Create, sau khi tạo xong thì sync  

2.0.  Test với Knowledge Base, chọn Nova 2 Lite, kết quả cho thấy AI trả lời có dẫn nguồn và chính xác:
<img width="915" height="304" alt="image" src="https://github.com/user-attachments/assets/ad264a6a-f6cd-4f14-8f47-44434d327d7e" />

# Bước 2. Lamda truy vấn tới Bedrock thông qua VPC Interface Enpoints
## Giai đoạn 1: Thiết lập Quyền (IAM Roles)

### 2.1. IAM Role cho Lambda (`Lambda-Bedrock-Query-Role`)
Role này cho phép Lambda chạy trong VPC và gọi dịch vụ Bedrock.
* **Trust Relationship:** `lambda.amazonaws.com`
* **Permissions Policies:**
    1. `AWSLambdaVPCAccessExecutionRole` (Policy có sẵn của AWS).
    2. **Inline Policy (`BedrockAccess`):**
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "bedrock:RetrieveAndGenerate",
                    "bedrock:Retrieve",
                    "bedrock:InvokeModel"
                ],
                "Resource": "*"
            }
        ]
    }
    ```

### 2.2. Kích hoạt Model Access
1. Truy cập **Amazon Bedrock** tại `us-west-2`.
2. Chọn **Model access** > **Manage model access**.
3. Tích chọn: **Titan Text Embeddings V2** và **Claude 3 Haiku**.
4. Nhấn **Save changes** (Chờ trạng thái hiện *Access granted*).

---

## 3. Giai đoạn 2: Hạ tầng Mạng (VPC & Security Groups)

Để Lambda gọi được Bedrock mà không cần Internet, ta thiết lập hệ thống "đường hầm ngầm".

### 3.1. Tạo Security Groups (SG)
1. **Lambda-RAG-SG-minie:**
   * Inbound: Trống.
   * Outbound: All traffic (0.0.0.0/0).
2. **Bedrock-Endpoint-SG-minie:**
   * Inbound: Cổng **HTTPS (443)**, Source là `Lambda-RAG-SG-minie`.
   * Outbound: All traffic.

### 3.2. Tạo VPC Interface Endpoint
1. Vào **VPC** > **Endpoints** > **Create endpoint**.
2. **Service:** `com.amazonaws.us-west-2.bedrock-agent-runtime`.
3. **VPC & Subnets:** Chọn VPC dự án và các **Private Subnets**.
4. **Security Groups:** Chọn `Bedrock-Endpoint-SG-minie`.
5. Nhấn **Create**.

---

## 4. Giai đoạn 3: Cấu hình Lambda Function

### 4.1. Khởi tạo Function
* **Name:** `query-bedrock-minie`
* **Runtime:** `Node.js 24.x`
* **VPC Configuration:** Gắn VPC , Private Subnets và Security Group (`Lambda-RAG-SG-minie`).

### 4.2. Mã nguồn (index.mjs)
Đoạn code này xử lý việc nhận câu hỏi, truy vấn Knowledge Base và trả về câu trả lời kèm nguồn dẫn (Citations).

```javascript
import { BedrockAgentRuntimeClient, RetrieveAndGenerateCommand } from "@aws-sdk/client-bedrock-agent-runtime";

const client = new BedrockAgentRuntimeClient({ region: "us-west-2" });

export const handler = async (event) => {
    const userQuestion = event.question || "giá của 1 cái ip15 là bao nhiêu?";
    const knowledgeBaseId = "AVHHNWOIBA"; 
    const modelArn = "arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-3-haiku-20240307-v1:0";

    try {
        const command = new RetrieveAndGenerateCommand({
            input: { text: userQuestion },
            retrieveAndGenerateConfiguration: {
                type: "KNOWLEDGE_BASE",
                knowledgeBaseConfiguration: {
                    knowledgeBaseId: knowledgeBaseId,
                    modelArn: modelArn
                }
            }
        });

        const response = await client.send(command);
        
        return {
            statusCode: 200,
            body: {
                answer: response.output.text,
                citations: response.citations
            }
        };
    } catch (error) {
        console.error("Error:", error);
        return { 
            statusCode: 500, 
            body: { error: error.message } 
        };
    }
};






      
      






