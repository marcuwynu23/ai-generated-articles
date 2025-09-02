# 🚀 How to Upload Large Files in Chunks (Flask + Axios)

Uploading large files (100MB, 500MB, or even several GB) is a challenge because browsers, proxies (like Cloudflare), and servers often impose request size and timeout limits. The solution? **Chunked uploads** — split the file into smaller parts (e.g., 20MB each), upload them sequentially, and let the server reassemble them.  

In this article, we’ll build:
- ✅ A Flask backend route (`/upload_chunk`) to handle chunk uploads  
- ✅ A frontend uploader with **FormData + Axios**  
- ✅ Automatic chunk splitting (20MB)  
- ✅ Progress tracking  

---

## 🔹 1. Backend: Flask chunk upload route

Here’s a simplified backend using **Flask**:

```python
from flask import Flask, request, jsonify
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)

BASE_FOLDER = "uploads"
os.makedirs(BASE_FOLDER, exist_ok=True)

@app.route('/upload_chunk', methods=['POST'])
def upload_file_chunk():
    folder = request.form.get('folder')
    if not folder:
        return jsonify({"error": "No folder specified"}), 400

    upload_folder = os.path.join(BASE_FOLDER, folder)
    os.makedirs(upload_folder, exist_ok=True)

    file = request.files.get('file')
    if not file:
        return jsonify({"error": "No file provided"}), 400

    chunk_index = int(request.form.get('dzchunkindex', 0))
    total_chunks = int(request.form.get('dztotalchunkcount', 1))

    filename = secure_filename(file.filename).replace(" ", "_")
    chunk_path = os.path.join(upload_folder, f"{filename}.part{chunk_index}")

    # Save the chunk
    file.save(chunk_path)

    # If last chunk, merge them
    if chunk_index + 1 == total_chunks:
        final_path = os.path.join(upload_folder, filename)
        with open(final_path, "wb") as final_file:
            for i in range(total_chunks):
                with open(os.path.join(upload_folder, f"{filename}.part{i}"), "rb") as chunk:
                    final_file.write(chunk.read())
                os.remove(os.path.join(upload_folder, f"{filename}.part{i}"))
        return jsonify({"message": "Upload complete", "path": final_path}), 200

    return jsonify({"message": f"Chunk {chunk_index + 1}/{total_chunks} uploaded"}), 200
```

👉 This route:
1. Receives one chunk per request.  
2. Saves it as `filename.part0`, `filename.part1`, …  
3. When the **last chunk arrives**, merges them into the final file.  

---

## 🔹 2. Frontend: Splitting & uploading with Axios

We’ll split the file into **20MB chunks** and upload them one by one.

```tsx
import axios from "axios";

async function uploadFile(file: File, folder: string) {
  const chunkSize = 20 * 1024 * 1024; // 20 MB
  const totalChunks = Math.ceil(file.size / chunkSize);

  for (let i = 0; i < totalChunks; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);

    const formData = new FormData();
    formData.append("folder", folder);
    formData.append("dzchunkindex", i.toString());
    formData.append("dztotalchunkcount", totalChunks.toString());
    formData.append("file", chunk, file.name);

    try {
      const response = await axios.post("http://localhost:5000/upload_chunk", formData, {
        headers: { "Content-Type": "multipart/form-data" },
        onUploadProgress: (e) => {
          const percent = Math.round((e.loaded * 100) / e.total);
          console.log(`Chunk ${i + 1}/${totalChunks}: ${percent}%`);
        },
      });
      console.log(response.data.message);
    } catch (err) {
      console.error("Upload failed at chunk", i, err);
      break;
    }
  }
}
```

---

## 🔹 3. React file input

```tsx
function FileUploader() {
  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      await uploadFile(file, "my_folder");
    }
  };

  return <input type="file" onChange={handleChange} />;
}
```

---

## 🔹 4. Why chunk uploads?
- ✅ Works around **Cloudflare’s 100MB request limit**  
- ✅ Avoids browser/server timeouts on giant files  
- ✅ Allows resuming failed uploads (advanced feature)  
- ✅ Provides **progress updates per chunk**  

---

## 🔹 5. Example: 147.7 MB file
- Chunk size = **20 MB**  
- 147.7 ÷ 20 = 7 full chunks + 1 small chunk  
- `dztotalchunkcount = 8`  
- You’ll upload chunks with `dzchunkindex = 0 … 7`.  
- Backend merges them into the final file.  

---

## 🎯 Conclusion
By splitting files into 20MB parts and sending them with **Axios + FormData**, you can upload files far beyond proxy/server limits. Flask handles the reassembly seamlessly, and your users get smooth progress tracking.

This approach scales from **100MB** to **multi-GB** uploads 🚀.  
