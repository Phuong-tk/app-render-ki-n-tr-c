# app-render-ki-n-tr-c
bạn là một chuyên gia render kiến trúc nội thất 
import streamlit as st
import google.generativeai as genai
from PIL import Image

# 1. CẤU HÌNH TRANG WEB
st.set_page_config(layout="wide", page_title="AI Mastery Render", page_icon="🏢")

# 2. KẾT NỐI API (Lấy từ hệ thống bảo mật Secrets của Streamlit)
# Chúng ta KHÔNG dán trực tiếp API Key vào đây để bảo mật tuyệt đối
try:
    api_key = st.secrets["GOOGLE_API_KEY"]
    genai.configure(api_key=api_key)
except:
    st.error("Chưa cấu hình API Key. Vui lòng vào cài đặt Secrets trên Streamlit Cloud.")
    st.stop()

# 3. CẤU HÌNH AI (MODEL)
generation_config = {
  "temperature": 0.4,
  "top_p": 0.95,
  "top_k": 40,
  "max_output_tokens": 8192,
}

# --- PHẦN QUAN TRỌNG: DÁN SYSTEM INSTRUCTION CỦA BẠN VÀO GIỮA 3 DẤU NGOẶC KÉP ---
system_instruction = """
Bạn là một chuyên gia Render Kiến trúc AI (AI Mastery Render). 
Nhiệm vụ của bạn là phân tích và render hình ảnh kiến trúc dựa trên bản vẽ đầu vào.
Giữ nguyên cấu trúc hình học, chỉ thay đổi vật liệu và ánh sáng theo phong cách yêu cầu.
""" 
# -----------------------------------------------------------------------------------

model = genai.GenerativeModel(
    model_name="gemini-1.5-flash", # Chọn model nhanh và rẻ nhất
    generation_config=generation_config,
    system_instruction=system_instruction,
)

# 4. GIAO DIỆN NGƯỜI DÙNG (FRONTEND)
st.markdown("""
<style>
    .stApp {background-color: #0e1117; color: white;}
    .stButton>button {width: 100%;}
</style>
""", unsafe_allow_html=True)

with st.sidebar:
    st.title("AI MASTERY RENDER")
    st.caption("Next-Gen Architecture")
    st.markdown("---")
    st.write("📂 **DIỄN HỌA 3D**")
    st.radio("Chế độ", ["Render Kiến trúc", "Render Nội thất", "Cải tạo AI"])

col1, col2 = st.columns(2)

with col1:
    st.subheader("1. Tải bản vẽ")
    uploaded_file = st.file_uploader("Upload ảnh (JPG, PNG)", type=['png', 'jpg', 'jpeg'])
    if uploaded_file:
        image = Image.open(uploaded_file)
        st.image(image, caption="Bản vẽ gốc", use_column_width=True)

with col2:
    st.subheader("2. Mô tả (Prompt)")
    prompt = st.text_area("Nhập mô tả...", value="Nhà phố hiện đại, ánh sáng tự nhiên, đường phố Việt Nam, render 8k chân thực")
    
    st.markdown("---")
    run_btn = st.button("🚀 BẮT ĐẦU RENDER", type="primary")

# 5. XỬ LÝ KHI BẤM NÚT
if run_btn and uploaded_file:
    with st.spinner("AI đang xử lý..."):
        try:
            response = model.generate_content([prompt, image])
            st.success("Hoàn tất!")
            st.markdown(response.text) # Hiển thị text mô tả
            # Lưu ý: Hiện tại Gemini API Vision trả về text/phân tích. 
            # Để tạo ảnh, Google sắp ra mắt Imagen API, lúc đó chỉ cần cập nhật đoạn code này là xong.
        except Exception as e:
            st.error(f"Lỗi: {e}")
