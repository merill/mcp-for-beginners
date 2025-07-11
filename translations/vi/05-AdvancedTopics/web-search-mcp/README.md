<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "7a11a5dcf2f9fdf6392f5a4545cf005e",
  "translation_date": "2025-06-11T16:02:38+00:00",
  "source_file": "05-AdvancedTopics/web-search-mcp/README.md",
  "language_code": "vi"
}
-->
# Bài học: Xây dựng MCP Server Tìm kiếm Web

Chương này hướng dẫn cách xây dựng một AI agent thực tế tích hợp với các API bên ngoài, xử lý nhiều loại dữ liệu khác nhau, quản lý lỗi và điều phối nhiều công cụ—tất cả trong một định dạng sẵn sàng cho môi trường sản xuất. Bạn sẽ thấy:

- **Tích hợp với các API bên ngoài yêu cầu xác thực**
- **Xử lý đa dạng loại dữ liệu từ nhiều điểm cuối**
- **Chiến lược xử lý lỗi và ghi log vững chắc**
- **Điều phối đa công cụ trong một server duy nhất**

Kết thúc bài học, bạn sẽ có kinh nghiệm thực tế với các mẫu và thực hành tốt nhất cần thiết cho các ứng dụng AI và LLM nâng cao.

## Giới thiệu

Trong bài học này, bạn sẽ học cách xây dựng một MCP server và client nâng cao mở rộng khả năng LLM với dữ liệu web thời gian thực bằng SerpAPI. Đây là kỹ năng quan trọng để phát triển các AI agent động có thể truy cập thông tin cập nhật từ web.

## Mục tiêu học tập

Kết thúc bài học, bạn sẽ có thể:

- Tích hợp các API bên ngoài (như SerpAPI) một cách an toàn vào MCP server
- Triển khai nhiều công cụ cho tìm kiếm web, tin tức, sản phẩm và hỏi đáp
- Phân tích và định dạng dữ liệu có cấu trúc để LLM tiêu thụ
- Xử lý lỗi và quản lý giới hạn tốc độ API hiệu quả
- Xây dựng và kiểm thử cả client tự động và tương tác

## MCP Server Tìm kiếm Web

Phần này giới thiệu kiến trúc và tính năng của MCP Server Tìm kiếm Web. Bạn sẽ thấy cách FastMCP và SerpAPI được sử dụng cùng nhau để mở rộng khả năng LLM với dữ liệu web thời gian thực.

### Tổng quan

Triển khai này có bốn công cụ thể hiện khả năng của MCP trong việc xử lý các tác vụ API bên ngoài đa dạng, an toàn và hiệu quả:

- **general_search**: Tìm kiếm web rộng
- **news_search**: Tin tức mới nhất
- **product_search**: Dữ liệu thương mại điện tử
- **qna**: Đoạn hỏi đáp

### Tính năng
- **Ví dụ mã nguồn**: Bao gồm các đoạn mã theo ngôn ngữ cụ thể cho Python (và dễ dàng mở rộng sang ngôn ngữ khác) với các phần có thể thu gọn để rõ ràng hơn

<details>  
<summary>Python</summary>  

```python
# Example usage of the general_search tool
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("general_search", arguments={"query": "open source LLMs"})
            print(result)
```
</details>

Trước khi chạy client, bạn nên hiểu server làm gì. File [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py) file implements the MCP server, exposing tools for web, news, product search, and Q&A by integrating with SerpAPI. It handles incoming requests, manages API calls, parses responses, and returns structured results to the client.

You can review the full implementation in [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py).

Dưới đây là ví dụ ngắn gọn về cách server định nghĩa và đăng ký một công cụ:

<details>  
<summary>Python Server</summary> 

```python
# server.py (excerpt)
from mcp.server import MCPServer, Tool

async def general_search(query: str):
    # ...implementation...

server = MCPServer()
server.add_tool(Tool("general_search", general_search))

if __name__ == "__main__":
    server.run()
```
</details>

- **Tích hợp API bên ngoài**: Minh họa cách xử lý an toàn khóa API và các yêu cầu bên ngoài
- **Phân tích dữ liệu có cấu trúc**: Biến đổi phản hồi API thành định dạng thân thiện với LLM
- **Xử lý lỗi**: Xử lý lỗi vững chắc kèm ghi log phù hợp
- **Client tương tác**: Bao gồm cả kiểm thử tự động và chế độ tương tác để thử nghiệm
- **Quản lý ngữ cảnh**: Tận dụng MCP Context để ghi log và theo dõi các yêu cầu

## Yêu cầu trước

Trước khi bắt đầu, hãy đảm bảo môi trường của bạn đã được thiết lập đúng bằng cách làm theo các bước sau. Điều này đảm bảo tất cả phụ thuộc được cài đặt và khóa API của bạn được cấu hình chính xác để phát triển và kiểm thử mượt mà.

- Python 3.8 trở lên
- Khóa API SerpAPI (Đăng ký tại [SerpAPI](https://serpapi.com/) - có gói miễn phí)

## Cài đặt

Để bắt đầu, làm theo các bước sau để thiết lập môi trường:

1. Cài đặt các phụ thuộc bằng uv (khuyến nghị) hoặc pip:

```bash
# Using uv (recommended)
uv pip install -r requirements.txt

# Using pip
pip install -r requirements.txt
```

2. Tạo file `.env` ở thư mục gốc dự án với khóa SerpAPI của bạn:

```
SERPAPI_KEY=your_serpapi_key_here
```

## Sử dụng

MCP Server Tìm kiếm Web là thành phần cốt lõi cung cấp các công cụ tìm kiếm web, tin tức, sản phẩm và hỏi đáp bằng cách tích hợp với SerpAPI. Nó xử lý các yêu cầu đến, quản lý các cuộc gọi API, phân tích phản hồi và trả về kết quả có cấu trúc cho client.

Bạn có thể xem toàn bộ triển khai trong [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py).

### Khởi chạy Server

Để chạy MCP server, dùng lệnh sau:

```bash
python server.py
```

Server sẽ chạy dưới dạng MCP server dựa trên stdio mà client có thể kết nối trực tiếp.

### Chế độ Client

Client (`client.py`) supports two modes for interacting with the MCP server:

- **Normal mode**: Runs automated tests that exercise all the tools and verify their responses. This is useful for quickly checking that the server and tools are working as expected.
- **Interactive mode**: Starts a menu-driven interface where you can manually select and call tools, enter custom queries, and see results in real time. This is ideal for exploring the server's capabilities and experimenting with different inputs.

You can review the full implementation in [`client.py`](../../../../05-AdvancedTopics/web-search-mcp/client.py).

### Chạy Client

Để chạy các kiểm thử tự động (lệnh này sẽ tự động khởi động server):

```bash
python client.py
```

Hoặc chạy ở chế độ tương tác:

```bash
python client.py --interactive
```

### Kiểm thử với các phương pháp khác nhau

Có nhiều cách để kiểm thử và tương tác với các công cụ do server cung cấp, tùy theo nhu cầu và quy trình làm việc của bạn.

#### Viết script kiểm thử tùy chỉnh với MCP Python SDK
Bạn cũng có thể tự viết các script kiểm thử sử dụng MCP Python SDK:

<details>
<summary>Python</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def test_custom_query():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            # Call tools with your custom parameters
            result = await session.call_tool("general_search", 
                                           arguments={"query": "your custom query"})
            # Process the result
```
</details>

Trong ngữ cảnh này, "script kiểm thử" là chương trình Python tùy chỉnh bạn viết để làm client cho MCP server. Thay vì là một unit test chính thức, script này cho phép bạn kết nối lập trình với server, gọi bất kỳ công cụ nào với tham số bạn chọn và kiểm tra kết quả. Cách làm này hữu ích cho:
- Thử nghiệm và khám phá các lời gọi công cụ
- Xác thực phản hồi của server với các đầu vào khác nhau
- Tự động hóa các lần gọi công cụ lặp lại
- Xây dựng các quy trình làm việc hoặc tích hợp riêng trên MCP server

Bạn có thể dùng script kiểm thử để nhanh chóng thử các truy vấn mới, gỡ lỗi hành vi công cụ, hoặc làm điểm khởi đầu cho tự động hóa nâng cao. Dưới đây là ví dụ về cách dùng MCP Python SDK để tạo script như vậy:

## Mô tả các công cụ

Bạn có thể sử dụng các công cụ sau do server cung cấp để thực hiện các loại tìm kiếm và truy vấn khác nhau. Mỗi công cụ được mô tả bên dưới với các tham số và ví dụ sử dụng.

Phần này cung cấp chi tiết về từng công cụ có sẵn và tham số của chúng.

### general_search

Thực hiện tìm kiếm web tổng quát và trả về kết quả đã định dạng.

**Cách gọi công cụ này:**

Bạn có thể gọi `general_search` từ script của bạn sử dụng MCP Python SDK, hoặc tương tác qua Inspector hoặc chế độ client tương tác. Dưới đây là ví dụ mã nguồn dùng SDK:

<details>
<summary>Ví dụ Python</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_general_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("general_search", arguments={"query": "latest AI trends"})
            print(result)
```
</details>

Ngoài ra, ở chế độ tương tác, chọn `general_search` from the menu and enter your query when prompted.

**Parameters:**
- `query` (chuỗi): Truy vấn tìm kiếm

**Ví dụ yêu cầu:**

```json
{
  "query": "latest AI trends"
}
```

### news_search

Tìm kiếm các bài báo tin tức gần đây liên quan đến truy vấn.

**Cách gọi công cụ này:**

Bạn có thể gọi `news_search` từ script của bạn sử dụng MCP Python SDK, hoặc tương tác qua Inspector hoặc chế độ client tương tác. Dưới đây là ví dụ mã nguồn dùng SDK:

<details>
<summary>Ví dụ Python</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_news_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("news_search", arguments={"query": "AI policy updates"})
            print(result)
```
</details>

Ngoài ra, ở chế độ tương tác, chọn `news_search` from the menu and enter your query when prompted.

**Parameters:**
- `query` (chuỗi): Truy vấn tìm kiếm

**Ví dụ yêu cầu:**

```json
{
  "query": "AI policy updates"
}
```

### product_search

Tìm kiếm sản phẩm phù hợp với truy vấn.

**Cách gọi công cụ này:**

Bạn có thể gọi `product_search` từ script của bạn sử dụng MCP Python SDK, hoặc tương tác qua Inspector hoặc chế độ client tương tác. Dưới đây là ví dụ mã nguồn dùng SDK:

<details>
<summary>Ví dụ Python</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_product_search():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("product_search", arguments={"query": "best AI gadgets 2025"})
            print(result)
```
</details>

Ngoài ra, ở chế độ tương tác, chọn `product_search` from the menu and enter your query when prompted.

**Parameters:**
- `query` (chuỗi): Truy vấn tìm kiếm sản phẩm

**Ví dụ yêu cầu:**

```json
{
  "query": "best AI gadgets 2025"
}
```

### qna

Lấy câu trả lời trực tiếp cho các câu hỏi từ công cụ tìm kiếm.

**Cách gọi công cụ này:**

Bạn có thể gọi `qna` từ script của bạn sử dụng MCP Python SDK, hoặc tương tác qua Inspector hoặc chế độ client tương tác. Dưới đây là ví dụ mã nguồn dùng SDK:

<details>
<summary>Ví dụ Python</summary>

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_qna():
    server_params = StdioServerParameters(
        command="python",
        args=["server.py"],
    )
    async with stdio_client(server_params) as (reader, writer):
        async with ClientSession(reader, writer) as session:
            await session.initialize()
            result = await session.call_tool("qna", arguments={"question": "what is artificial intelligence"})
            print(result)
```
</details>

Ngoài ra, ở chế độ tương tác, chọn `qna` from the menu and enter your question when prompted.

**Parameters:**
- `question` (chuỗi): Câu hỏi cần tìm câu trả lời

**Ví dụ yêu cầu:**

```json
{
  "question": "what is artificial intelligence"
}
```

## Chi tiết mã nguồn

Phần này cung cấp các đoạn mã và tham chiếu cho triển khai server và client.

<details>
<summary>Python</summary>

Xem [`server.py`](../../../../05-AdvancedTopics/web-search-mcp/server.py) and [`client.py`](../../../../05-AdvancedTopics/web-search-mcp/client.py) để biết chi tiết triển khai đầy đủ.

```python
# Example snippet from server.py:
import os
import httpx
# ...existing code...
```
</details>

## Các khái niệm nâng cao trong bài học này

Trước khi bắt đầu xây dựng, dưới đây là một số khái niệm nâng cao quan trọng sẽ xuất hiện xuyên suốt chương này. Hiểu những điều này sẽ giúp bạn theo kịp, ngay cả khi bạn mới làm quen:

- **Điều phối đa công cụ**: Nghĩa là chạy nhiều công cụ khác nhau (như tìm kiếm web, tin tức, sản phẩm và hỏi đáp) trong một MCP server duy nhất. Điều này cho phép server xử lý nhiều loại tác vụ, không chỉ một.
- **Xử lý giới hạn tốc độ API**: Nhiều API bên ngoài (như SerpAPI) giới hạn số lần yêu cầu bạn có thể thực hiện trong một khoảng thời gian nhất định. Mã tốt sẽ kiểm tra các giới hạn này và xử lý một cách nhẹ nhàng, tránh làm ứng dụng bị lỗi khi vượt hạn mức.
- **Phân tích dữ liệu có cấu trúc**: Phản hồi API thường phức tạp và lồng nhau. Khái niệm này là biến những phản hồi đó thành định dạng sạch, dễ dùng, thân thiện với LLM hoặc các chương trình khác.
- **Phục hồi lỗi**: Đôi khi có sự cố—có thể mạng bị lỗi, hoặc API không trả về dữ liệu như mong đợi. Phục hồi lỗi có nghĩa là mã của bạn có thể xử lý những vấn đề này và vẫn cung cấp phản hồi hữu ích, thay vì bị crash.
- **Xác thực tham số**: Đây là việc kiểm tra tất cả đầu vào cho công cụ của bạn là chính xác và an toàn để sử dụng. Bao gồm thiết lập giá trị mặc định và đảm bảo kiểu dữ liệu đúng, giúp tránh lỗi và nhầm lẫn.

Phần này sẽ giúp bạn chẩn đoán và giải quyết các vấn đề thường gặp khi làm việc với MCP Server Tìm kiếm Web. Nếu bạn gặp lỗi hoặc hành vi không mong muốn, phần xử lý sự cố này cung cấp các giải pháp cho những vấn đề phổ biến nhất. Hãy xem qua các mẹo dưới đây trước khi tìm kiếm trợ giúp thêm—chúng thường giải quyết nhanh các vấn đề.

## Xử lý sự cố

Khi làm việc với MCP Server Tìm kiếm Web, đôi khi bạn sẽ gặp vấn đề—điều này bình thường khi phát triển với API bên ngoài và công cụ mới. Phần này cung cấp các giải pháp thực tế cho những sự cố phổ biến nhất, giúp bạn nhanh chóng trở lại làm việc. Nếu gặp lỗi, hãy bắt đầu từ đây: các mẹo dưới đây giải quyết các vấn đề mà đa số người dùng gặp phải và thường giúp bạn xử lý mà không cần hỗ trợ thêm.

### Vấn đề thường gặp

Dưới đây là một số vấn đề phổ biến người dùng gặp phải, kèm theo giải thích rõ ràng và các bước khắc phục:

1. **Thiếu SERPAPI_KEY trong file .env**
   - Nếu bạn thấy lỗi `SERPAPI_KEY environment variable not found`, it means your application can't find the API key needed to access SerpAPI. To fix this, create a file named `.env` in your project root (if it doesn't already exist) and add a line like `SERPAPI_KEY=your_serpapi_key_here`. Make sure to replace `your_serpapi_key_here` with your actual key from the SerpAPI website.

2. **Module not found errors**
   - Errors such as `ModuleNotFoundError: No module named 'httpx'` indicate that a required Python package is missing. This usually happens if you haven't installed all the dependencies. To resolve this, run `pip install -r requirements.txt` in your terminal to install everything your project needs.

3. **Connection issues**
   - If you get an error like `Error during client execution`, it often means the client can't connect to the server, or the server isn't running as expected. Double-check that both the client and server are compatible versions, and that `server.py` is present and running in the correct directory. Restarting both the server and client can also help.

4. **SerpAPI errors**
   - Seeing `Search API returned error status: 401` means your SerpAPI key is missing, incorrect, or expired. Go to your SerpAPI dashboard, verify your key, and update your `.env`, hãy tạo file `.env` nếu cần. Nếu khóa của bạn đúng nhưng vẫn gặp lỗi này, hãy kiểm tra xem gói miễn phí của bạn đã hết hạn mức chưa.

### Chế độ Debug

Mặc định, ứng dụng chỉ ghi log những thông tin quan trọng. Nếu bạn muốn xem chi tiết hơn về các bước ứng dụng đang thực hiện (ví dụ, để chẩn đoán các vấn đề khó), bạn có thể bật chế độ DEBUG. Điều này sẽ hiển thị nhiều thông tin hơn về từng bước.

**Ví dụ: Output bình thường**
```plaintext
2025-06-01 10:15:23,456 - __main__ - INFO - Calling general_search with params: {'query': 'open source LLMs'}
2025-06-01 10:15:24,123 - __main__ - INFO - Successfully called general_search

GENERAL_SEARCH RESULTS:
... (search results here) ...
```

**Ví dụ: Output chế độ DEBUG**
```plaintext
2025-06-01 10:15:23,456 - __main__ - INFO - Calling general_search with params: {'query': 'open source LLMs'}
2025-06-01 10:15:23,457 - httpx - DEBUG - HTTP Request: GET https://serpapi.com/search ...
2025-06-01 10:15:23,458 - httpx - DEBUG - HTTP Response: 200 OK ...
2025-06-01 10:15:24,123 - __main__ - INFO - Successfully called general_search

GENERAL_SEARCH RESULTS:
... (search results here) ...
```

Lưu ý chế độ DEBUG bao gồm thêm các dòng về yêu cầu HTTP, phản hồi và các chi tiết nội bộ khác. Điều này rất hữu ích cho việc xử lý sự cố.

Để bật chế độ DEBUG, đặt mức logging thành DEBUG ở đầu file `client.py` or `server.py`:

<details>
<summary>Python</summary>

```python
# At the top of your client.py or server.py
import logging
logging.basicConfig(
    level=logging.DEBUG,  # Change from INFO to DEBUG
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
```
</details>

---

## Tiếp theo là gì

- [5.10 Real Time Streaming](../mcp-realtimestreaming/README.md)

**Tuyên bố từ chối trách nhiệm**:  
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng các bản dịch tự động có thể chứa lỗi hoặc không chính xác. Tài liệu gốc bằng ngôn ngữ gốc nên được xem là nguồn chính xác và đáng tin cậy. Đối với các thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp do con người thực hiện. Chúng tôi không chịu trách nhiệm về bất kỳ sự hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.