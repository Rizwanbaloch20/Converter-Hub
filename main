import streamlit as st
from PIL import Image
import os
from io import BytesIO
from PyPDF2 import PdfMerger, PdfReader, PdfWriter
from pdf2image import convert_from_bytes
import pytesseract
import zipfile
import tempfile
from pydub import AudioSegment
import moviepy.editor as mp

# Simulated user session storage
if "history" not in st.session_state:
    st.session_state.history = []

# Set up page
st.set_page_config(page_title="All-in-One File Converter", layout="wide")
st.title("🔄 All-in-One File & Media Converter")

# Sidebar
st.sidebar.title("Select Conversion Tool")
tool = st.sidebar.selectbox("Choose a Tool", [
    "Image Converter",
    "PDF Tools",
    "Compress Files",
    "OCR (Image to Text)",
    "Audio Converter",
    "Video Converter",
    "Conversion History",
    "About"
])

# Helper Functions
def convert_image(img_file, output_format):
    img = Image.open(img_file)
    img = img.convert("RGB")
    buf = BytesIO()
    img.save(buf, format=output_format.upper())
    return buf.getvalue()

def compress_image(img_file):
    img = Image.open(img_file)
    img = img.convert("RGB")
    buf = BytesIO()
    img.save(buf, optimize=True, quality=40, format='JPEG')
    return buf.getvalue()

def compress_pdf(pdf_bytes):
    reader = PdfReader(BytesIO(pdf_bytes))
    writer = PdfWriter()
    for page in reader.pages:
        writer.add_page(page)
    output = BytesIO()
    writer.write(output)
    return output.getvalue()

def save_to_history(tool_name, filename):
    st.session_state.history.append((tool_name, filename))

# Image Converter
if tool == "Image Converter":
    st.header("🖼️ Image Format Converter")
    uploaded_files = st.file_uploader("Upload images", type=["jpg", "jpeg", "png", "webp"], accept_multiple_files=True)
    output_format = st.selectbox("Convert to format", ["JPEG", "PNG", "WEBP"])
    if st.button("Convert Images") and uploaded_files:
        with zipfile.ZipFile("converted_images.zip", "w") as zipf:
            for uploaded_file in uploaded_files:
                data = convert_image(uploaded_file, output_format)
                filename = os.path.splitext(uploaded_file.name)[0] + "." + output_format.lower()
                zipf.writestr(filename, data)
                save_to_history("Image Converted", filename)
        with open("converted_images.zip", "rb") as f:
            st.download_button("Download All Converted Images", f, file_name="converted_images.zip")

# PDF Tools
elif tool == "PDF Tools":
    st.header("📄 PDF Tools")
    action = st.selectbox("Choose PDF action", ["Merge PDFs", "Split PDF", "Password Protect PDF"])

    if action == "Merge PDFs":
        files = st.file_uploader("Upload PDF files", type="pdf", accept_multiple_files=True)
        if st.button("Merge") and files:
            merger = PdfMerger()
            for pdf in files:
                merger.append(BytesIO(pdf.read()))
            out = BytesIO()
            merger.write(out)
            save_to_history("PDF Merged", "merged.pdf")
            st.download_button("Download Merged PDF", out.getvalue(), file_name="merged.pdf")

    elif action == "Split PDF":
        file = st.file_uploader("Upload a PDF to split", type="pdf")
        if file:
            reader = PdfReader(file)
            for i, page in enumerate(reader.pages):
                writer = PdfWriter()
                writer.add_page(page)
                output = BytesIO()
                writer.write(output)
                filename = f"page_{i+1}.pdf"
                save_to_history("PDF Split", filename)
                st.download_button(f"Download Page {i+1}", data=output.getvalue(), file_name=filename)

    elif action == "Password Protect PDF":
        file = st.file_uploader("Upload PDF", type="pdf")
        password = st.text_input("Enter password")
        if file and password and st.button("Encrypt"):
            reader = PdfReader(file)
            writer = PdfWriter()
            for page in reader.pages:
                writer.add_page(page)
            writer.encrypt(password)
            output = BytesIO()
            writer.write(output)
            save_to_history("PDF Encrypted", "encrypted.pdf")
            st.download_button("Download Encrypted PDF", output.getvalue(), file_name="encrypted.pdf")

# Compress Files
elif tool == "Compress Files":
    st.header("📦 File Compression")
    file = st.file_uploader("Upload file to compress (image or PDF)", type=["jpg", "jpeg", "png", "pdf"])
    if file and st.button("Compress"):
        ext = os.path.splitext(file.name)[1].lower()
        if ext in [".jpg", ".jpeg", ".png"]:
            compressed = compress_image(file)
            save_to_history("Image Compressed", "compressed.jpg")
            st.download_button("Download Compressed Image", compressed, file_name="compressed.jpg")
        elif ext == ".pdf":
            compressed = compress_pdf(file.read())
            save_to_history("PDF Compressed", "compressed.pdf")
            st.download_button("Download Compressed PDF", compressed, file_name="compressed.pdf")

# OCR Tool
elif tool == "OCR (Image to Text)":
    st.header("🔍 OCR: Extract Text from Image")
    img_file = st.file_uploader("Upload an image", type=["jpg", "jpeg", "png"])
    if img_file and st.button("Extract Text"):
        img = Image.open(img_file)
        text = pytesseract.image_to_string(img)
        st.text_area("Extracted Text", text, height=300)
        save_to_history("OCR Performed", img_file.name)

# Audio Converter
elif tool == "Audio Converter":
    st.header("🎵 Audio Format Converter")
    audio_file = st.file_uploader("Upload Audio File", type=["mp3", "wav", "ogg"])
    output_format = st.selectbox("Convert to format", ["mp3", "wav", "ogg"])
    if audio_file and st.button("Convert Audio"):
        sound = AudioSegment.from_file(audio_file)
        out_audio = BytesIO()
        sound.export(out_audio, format=output_format)
        out_audio.seek(0)
        filename = f"converted_audio.{output_format}"
        save_to_history("Audio Converted", filename)
        st.download_button("Download Converted Audio", out_audio, file_name=filename)

# Video Converter
elif tool == "Video Converter":
    st.header("🎬 Video Format Converter")
    video_file = st.file_uploader("Upload Video File", type=["mp4", "avi", "mov"])
    output_format = st.selectbox("Convert to format", ["mp4", "avi", "webm"])
    if video_file and st.button("Convert Video"):
        temp_input = tempfile.NamedTemporaryFile(delete=False)
        temp_input.write(video_file.read())
        temp_input.close()

        clip = mp.VideoFileClip(temp_input.name)
        temp_output = tempfile.NamedTemporaryFile(delete=False, suffix=f".{output_format}")
        clip.write_videofile(temp_output.name, codec='libx264')
        with open(temp_output.name, "rb") as f:
            video_bytes = f.read()
            save_to_history("Video Converted", f"converted_video.{output_format}")
            st.download_button("Download Converted Video", video_bytes, file_name=f"converted_video.{output_format}")

# History
elif tool == "Conversion History":
    st.header("📜 Conversion History")
    if st.session_state.history:
        for item in st.session_state.history:
            st.write(f"🔹 {item[0]} → `{item[1]}`")
    else:
        st.info("No history available yet.")

# About
elif tool == "About":
    st.markdown("""
    ### 🛠️ Built with Streamlit
    This app supports:
    - Image format conversion
    - PDF merging/splitting/encryption
    - File compression
    - OCR text extraction
    - Audio/Video conversion
    - Conversion history tracking
    
    More features like drag-and-drop UI enhancements and login authentication coming soon!
    """)
