<template>
  <div class="page-wrapper">
    <div class="container">
      <div class="header">
        <h1>PDF Sizer</h1>
        <p>Easily compress your PDF files without losing quality</p>
      </div>

      <div
        class="upload-area"
        @click="triggerFileInput"
        @dragover.prevent="highlight"
        @dragleave="unhighlight"
        @drop.prevent="handleDrop"
        :class="{ dragover: isDragover }"
      >
        <div class="upload-icon">📄</div>
        <p>Drop your PDF here or click to browse</p>
        <input
          type="file"
          ref="fileInput"
          accept=".pdf"
          @change="handleFiles"
          style="display: none;"
        />
      </div>

      <div v-if="file" class="file-info">
        <div>
          <span class="file-label">Original:</span>
          <span>{{ file.name }} ({{ formattedFileSize }})</span>
        </div>
        <div v-if="compressedSize" class="compressed-info">
          <span class="file-label">Compressed:</span>
          <span>{{ formattedCompressedSize }}</span>
        </div>
      </div>

      <button
        class="compress-btn"
        :disabled="!file || isCompressing"
        @click="compressFile"
      >
        <span v-if="isCompressing" class="spinner"></span>
        <span>{{ isCompressing ? 'Compressing...' : 'Compress PDF' }}</span>
      </button>

      <div v-if="successMessage" class="success-message" @click="downloadFile">
        PDF compressed successfully! Click to download.
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const file = ref(null);
const isCompressing = ref(false);
const successMessage = ref(false);
const isDragover = ref(false);
const fileInput = ref(null);
const compressedSize = ref(null);

const triggerFileInput = () => {
  fileInput.value.click();
};

const handleFiles = (event) => {
  const selectedFile = event.target.files[0];
  if (selectedFile && selectedFile.type === "application/pdf") {
    file.value = selectedFile;
    successMessage.value = false;
    compressedSize.value = null;
  } else {
    alert("Please select a valid PDF file.");
  }
};

const handleDrop = (event) => {
  const droppedFile = event.dataTransfer.files[0];
  if (droppedFile && droppedFile.type === "application/pdf") {
    file.value = droppedFile;
    successMessage.value = false;
    isDragover.value = false;
    compressedSize.value = null;
  } else {
    alert("Please drop a valid PDF file.");
  }
};

const highlight = () => {
  isDragover.value = true;
};

const unhighlight = () => {
  isDragover.value = false;
};

const compressFile = () => {
  isCompressing.value = true;
  successMessage.value = false;

  setTimeout(() => {
    const compressionRatio = 0.4 + Math.random() * 0.2;
    compressedSize.value = Math.floor(file.value.size * compressionRatio);
    isCompressing.value = false;
    successMessage.value = true;
  }, 2000);
};

const downloadFile = () => {
  if (!file.value) return;

  // Simulate a compressed PDF file (in reality, you should have your compression logic here)
  const compressedBlob = new Blob(["Simulated compressed file content"], {
    type: "application/pdf",
  });

  const url = URL.createObjectURL(compressedBlob);

  const a = document.createElement("a");
  a.href = url;
  a.download = "compressed.pdf"; // Make sure the file name is correct
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);

  // Revoke the object URL after download to release memory
  URL.revokeObjectURL(url);
};

const formattedFileSize = computed(() => {
  if (!file.value) return "";
  const bytes = file.value.size;
  if (bytes === 0) return "0 Bytes";
  const k = 1024;
  const sizes = ["Bytes", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
});

const formattedCompressedSize = computed(() => {
  if (!compressedSize.value) return "";
  const bytes = compressedSize.value;
  if (bytes === 0) return "0 Bytes";
  const k = 1024;
  const sizes = ["Bytes", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
});
</script>

<style scoped>


.page-wrapper {
  height: 100vh; /* Changed from min-height to height */
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  overflow: hidden; /* Added to prevent scrolling */
}

.container {
  background: white;
  padding: 40px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  text-align: center;
}

.header {
  text-align: center;
}

.header h1 {
  color: #1a202c;
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.header p {
  color: #718096;
  font-size: 1.1rem;
}

.upload-area {
  border: 2px dashed #cbd5e0;
  border-radius: 10px;
  padding: 40px 20px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.upload-area:hover,
.upload-area.dragover {
  border-color: #667eea;
  background: #f7fafc;
}

.file-info {
  background: #f7fafc;
  padding: 15px;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.file-label {
  font-weight: bold;
  margin-right: 8px;
  color: #4a5568;
}

.compressed-info {
  color: #2f855a;
}

.compress-btn {
  width: 100%;
  padding: 15px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background 0.3s ease;
}

.compress-btn:hover {
  background: #5a67d8;
}

.compress-btn:disabled {
  background: #cbd5e0;
  cursor: not-allowed;
}

.success-message {
  background: #c6f6d5;
  color: #2f855a;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
  cursor: pointer;
}

.spinner {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 3px solid #ffffff;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
