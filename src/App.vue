<template>
  <div class="page-wrapper">
    <div class="container">
      <div class="header">
        <h1>PDF Compressor</h1>
        <p>Easily compress your PDF files while maintaining readability</p>
      </div>

      <div
        class="upload-area"
        @click="triggerFileInput"
        @dragover.prevent="highlight"
        @dragleave="unhighlight"
        @drop.prevent="handleDrop"
        :class="{ dragover: isDragover }"
        aria-label="Upload or drop PDF file"
      >
        <div class="upload-icon">📄</div>
        <p>Drop your PDF here or click to browse</p>
        <input
          type="file"
          ref="fileInput"
          accept=".pdf"
          @change="handleFiles"
          aria-hidden="true"
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
          <span class="compression-ratio">
            ({{ Math.round((1 - compressedSize / file.size) * 100) }}% reduction)
          </span>
        </div>
      </div>

      <div class="compression-options" v-if="file">
        <label>
          Compression Level:
          <select v-model="compressionLevel">
            <option value="low">Low (Better Quality)</option>
            <option value="medium">Medium (Balanced)</option>
            <option value="high">High (Smaller Size)</option>
          </select>
        </label>
      </div>

      <button
        class="compress-btn"
        :disabled="!file || isCompressing"
        @click="compressFile"
        aria-label="Compress PDF"
      >
        <span v-if="isCompressing" class="spinner"></span>
        <span>{{ isCompressing ? 'Compressing...' : 'Compress PDF' }}</span>
      </button>

      <div
        v-if="successMessage"
        class="success-message"
        @click="downloadFile"
        aria-label="Download compressed PDF"
      >
        PDF compressed successfully! Click to download.
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { PDFDocument } from 'pdf-lib';

const file = ref(null);
const isCompressing = ref(false);
const successMessage = ref(false);
const isDragover = ref(false);
const fileInput = ref(null);
const compressedSize = ref(null);
const compressionLevel = ref('medium');
const compressedPdfBlob = ref(null);

// Updated compression settings with PDF-specific parameters
const compressionSettings = {
  low: {
    useObjectStreams: true,
    objectsPerTick: 20,
    compression: {
      imageQuality: 0.9,
      imageCompression: true
    }
  },
  medium: {
    useObjectStreams: true,
    objectsPerTick: 50,
    compression: {
      imageQuality: 0.7,
      imageCompression: true
    }
  },
  high: {
    useObjectStreams: true,
    objectsPerTick: 100,
    compression: {
      imageQuality: 0.5,
      imageCompression: true
    }
  }
};

const triggerFileInput = () => {
  fileInput.value.click();
};

const handleFiles = (event) => {
  const selectedFile = event.target.files[0];
  validateFile(selectedFile);
};

const handleDrop = (event) => {
  const droppedFile = event.dataTransfer.files[0];
  validateFile(droppedFile);
  isDragover.value = false;
};

const validateFile = (selectedFile) => {
  if (selectedFile && selectedFile.type === "application/pdf") {
    file.value = selectedFile;
    successMessage.value = false;
    compressedSize.value = null;
    compressedPdfBlob.value = null;
  } else {
    alert("Please select a valid PDF file.");
  }
};

const highlight = () => {
  isDragover.value = true;
};

const unhighlight = () => {
  isDragover.value = false;
};

const compressFile = async () => {
  try {
    isCompressing.value = true;
    successMessage.value = false;

    // Read the PDF file
    const arrayBuffer = await file.value.arrayBuffer();
    
    // Load the PDF document
    const pdfDoc = await PDFDocument.load(arrayBuffer);

    // Get compression settings based on selected level
    const settings = compressionSettings[compressionLevel.value];

    // Copy pages to a new document with compression
    const compressedPdf = await PDFDocument.create();
    
    // Copy each page with compression
    const pages = await compressedPdf.copyPages(pdfDoc, pdfDoc.getPageIndices());
    pages.forEach(page => compressedPdf.addPage(page));

    // Apply compression and save
    const compressedBytes = await compressedPdf.save({
      useObjectStreams: settings.useObjectStreams,
      objectsPerTick: settings.objectsPerTick,
      updateMetadata: false,
      addDefaultPage: false,
      // Compress images if present
      fullyCompressImages: settings.compression.imageCompression,
      // Set image quality
      imageQuality: settings.compression.imageQuality
    });

    // Create blob from compressed PDF
    compressedPdfBlob.value = new Blob([compressedBytes], { type: 'application/pdf' });
    compressedSize.value = compressedPdfBlob.value.size;
    
    isCompressing.value = false;
    successMessage.value = true;
  } catch (error) {
    console.error('Compression error:', error);
    alert("An error occurred during compression. Please try again.");
    isCompressing.value = false;
  }
};

const downloadFile = () => {
  if (!compressedPdfBlob.value) return;

  const url = URL.createObjectURL(compressedPdfBlob.value);
  const a = document.createElement("a");
  a.href = url;
  a.download = `${file.value.name.replace(".pdf", "")}_compressed.pdf`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
};

const formattedFileSize = computed(() => {
  if (!file.value) return "";
  return formatFileSize(file.value.size);
});

const formattedCompressedSize = computed(() => {
  if (!compressedSize.value) return "";
  return formatFileSize(compressedSize.value);
});

const formatFileSize = (bytes) => {
  if (bytes === 0) return "0 Bytes";
  const k = 1024;
  const sizes = ["Bytes", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
};
</script>

<style scoped>
.page-wrapper {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
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

.compression-options {
  background: #f7fafc;
  padding: 15px;
  border-radius: 8px;
  margin-top: 10px;
}

.compression-options select {
  margin-left: 10px;
  padding: 5px;
  border-radius: 4px;
  border: 1px solid #cbd5e0;
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

.compression-ratio {
  margin-left: 8px;
  color: #4a5568;
  font-size: 0.9em;
}
</style>
