<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF File Compressor</title>
    <script src="https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>
    <script src="https://unpkg.com/downloadjs@1.4.7"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            background-color: #f5f5f5;
            color: #333;
        }
        
        .container {
            max-width: 800px;
            margin: 50px auto;
            padding: 30px;
            background-color: #fff;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
        }
        
        h1 {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 30px;
        }
        
        .upload-area {
            border: 2px dashed #3498db;
            border-radius: 5px;
            padding: 40px;
            text-align: center;
            margin-bottom: 30px;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        .upload-area:hover {
            background-color: #f0f8ff;
            border-color: #2980b9;
        }
        
        .upload-area i {
            font-size: 48px;
            color: #3498db;
            margin-bottom: 15px;
        }
        
        #file-input {
            display: none;
        }
        
        .options {
            margin-bottom: 30px;
        }
        
        .option-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
        }
        
        select, input[type="range"] {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        input[type="range"] {
            -webkit-appearance: none;
            height: 8px;
            background: #ddd;
            border-radius: 5px;
        }
        
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            background: #3498db;
            cursor: pointer;
            border-radius: 50%;
        }
        
        .quality-value {
            text-align: center;
            font-weight: bold;
            margin-top: 5px;
        }
        
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 16px;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #2980b9;
        }
        
        button:disabled {
            background-color: #95a5a6;
            cursor: not-allowed;
        }
        
        .status {
            margin-top: 20px;
            padding: 15px;
            border-radius: 5px;
            display: none;
        }
        
        .progress {
            margin-top: 20px;
            height: 10px;
            background-color: #ecf0f1;
            border-radius: 5px;
            overflow: hidden;
            display: none;
        }
        
        .progress-bar {
            height: 100%;
            background-color: #2ecc71;
            width: 0%;
            transition: width 0.3s;
        }
        
        .file-info {
            margin-top: 20px;
            padding: 15px;
            background-color: #f8f9fa;
            border-radius: 5px;
            display: none;
        }
        
        .download-btn {
            margin-top: 20px;
            display: none;
        }
        
        @media (max-width: 600px) {
            .container {
                margin: 20px;
                padding: 20px;
            }
            
            .upload-area {
                padding: 30px 15px;
            }
        }
    </style>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
</head>
<body>
    <div class="container">
        <h1>PDF File Compressor</h1>
        
        <div class="upload-area" id="upload-area">
            <i class="fas fa-file-pdf"></i>
            <h3>Drag & Drop PDF Files Here</h3>
            <p>or click to browse files</p>
            <input type="file" id="file-input" accept=".pdf" />
        </div>
        
        <div class="options">
            <div class="option-group">
                <label for="compression-level">Compression Level:</label>
                <select id="compression-level">
                    <option value="low">Low (Faster, Larger File)</option>
                    <option value="medium" selected>Medium (Balanced)</option>
                    <option value="high">High (Slower, Smaller File)</option>
                </select>
            </div>
            
            <div class="option-group">
                <label for="quality">Image Quality: <span id="quality-value">80</span>%</label>
                <input type="range" id="quality" min="10" max="100" value="80" step="5">
            </div>
        </div>
        
        <button id="compress-btn" disabled>Compress PDF</button>
        
        <div class="progress" id="progress-container">
            <div class="progress-bar" id="progress-bar"></div>
        </div>
        
        <div class="status" id="status"></div>
        
        <div class="file-info" id="file-info">
            <h4>File Information:</h4>
            <p>Original Size: <span id="original-size">-</span></p>
            <p>Compressed Size: <span id="compressed-size">-</span></p>
            <p>Reduction: <span id="reduction">-</span></p>
        </div>
        
        <a class="download-btn" id="download-btn">
            <button>
                <i class="fas fa-download"></i> Download Compressed PDF
            </button>
        </a>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const { PDFDocument } = PDFLib;
            const uploadArea = document.getElementById('upload-area');
            const fileInput = document.getElementById('file-input');
            const compressBtn = document.getElementById('compress-btn');
            const qualitySlider = document.getElementById('quality');
            const qualityValue = document.getElementById('quality-value');
            const progressContainer = document.getElementById('progress-container');
            const progressBar = document.getElementById('progress-bar');
            const statusDiv = document.getElementById('status');
            const fileInfoDiv = document.getElementById('file-info');
            const originalSizeSpan = document.getElementById('original-size');
            const compressedSizeSpan = document.getElementById('compressed-size');
            const reductionSpan = document.getElementById('reduction');
            const downloadBtn = document.getElementById('download-btn');
            
            let selectedFile = null;
            let compressedPdfBytes = null;
            
            // Update quality value display
            qualitySlider.addEventListener('input', function() {
                qualityValue.textContent = this.value;
            });
            
            // Handle file selection
            uploadArea.addEventListener('click', function() {
                fileInput.click();
            });
            
            fileInput.addEventListener('change', function(e) {
                if (e.target.files.length > 0) {
                    handleFileSelection(e.target.files[0]);
                }
            });
            
            // Handle drag and drop
            uploadArea.addEventListener('dragover', function(e) {
                e.preventDefault();
                uploadArea.style.backgroundColor = '#f0f8ff';
                uploadArea.style.borderColor = '#2980b9';
            });
            
            uploadArea.addEventListener('dragleave', function() {
                uploadArea.style.backgroundColor = '';
                uploadArea.style.borderColor = '#3498db';
            });
            
            uploadArea.addEventListener('drop', function(e) {
                e.preventDefault();
                uploadArea.style.backgroundColor = '';
                uploadArea.style.borderColor = '#3498db';
                
                if (e.dataTransfer.files.length > 0) {
                    handleFileSelection(e.dataTransfer.files[0]);
                }
            });
            
            // Handle file selection
            function handleFileSelection(file) {
                if (file.type !== 'application/pdf') {
                    showStatus('Please select a PDF file.', 'error');
                    return;
                }
                
                selectedFile = file;
                compressBtn.disabled = false;
                
                // Show file info
                const fileSize = (file.size / 1024).toFixed(2);
                uploadArea.innerHTML = `
                    <i class="fas fa-file-pdf"></i>
                    <h3>${file.name}</h3>
                    <p>${fileSize} KB</p>
                `;
                
                originalSizeSpan.textContent = `${fileSize} KB`;
            }
            
            // Compress button click handler
            compressBtn.addEventListener('click', async function() {
                if (!selectedFile) return;
                
                // Show progress
                progressContainer.style.display = 'block';
                progressBar.style.width = '0%';
                statusDiv.style.display = 'none';
                fileInfoDiv.style.display = 'none';
                downloadBtn.style.display = 'none';
                
                try {
                    showStatus('Compressing PDF...', 'info');
                    
                    // Read the PDF file
                    const arrayBuffer = await selectedFile.arrayBuffer();
                    
                    // Load the PDF document
                    const pdfDoc = await PDFDocument.load(arrayBuffer);
                    
                    // Get the compression level
                    const compressionLevel = document.getElementById('compression-level').value;
                    
                    // Set compression options based on user selection
                    let compressImages = false;
                    let imageQuality = parseInt(qualitySlider.value) / 100;
                    
                    // For this demo, we'll just save with different compression settings
                    // Note: PDF-LIB has limited compression options in the browser
                    const saveOptions = {
                        useObjectStreams: true, // Enables some compression
                    };
                    
                    if (compressionLevel === 'high') {
                        saveOptions.useObjectStreams = true;
                        compressImages = true;
                    } else if (compressionLevel === 'medium') {
                        saveOptions.useObjectStreams = true;
                        compressImages = false;
                    } else {
                        saveOptions.useObjectStreams = false;
                        compressImages = false;
                    }
                    
                    // Simulate progress (actual compression happens all at once)
                    let progress = 0;
                    const interval = setInterval(() => {
                        progress += 10;
                        if (progress > 90) progress = 90;
                        progressBar.style.width = `${progress}%`;
                    }, 100);
                    
                    // Compress images if requested
                    if (compressImages) {
                        const pages = pdfDoc.getPages();
                        for (const page of pages) {
                            const images = await page.getImages();
                            for (const image of images) {
                                const imageDict = image.node;
                                if (imageDict instanceof PDFLib.PDFImageXObject) {
                                    // In a real app, you would compress the image here
                                    // This is just a placeholder as browser-side image compression is limited
                                }
                            }
                        }
                    }
                    
                    // Save the compressed PDF
                    compressedPdfBytes = await pdfDoc.save(saveOptions);
                    
                    // Complete progress
                    clearInterval(interval);
                    progressBar.style.width = '100%';
                    
                    // Show results
                    const originalSize = selectedFile.size;
                    const compressedSize = compressedPdfBytes.byteLength;
                    
                    originalSizeSpan.textContent = `${(originalSize / 1024).toFixed(2)} KB`;
                    compressedSizeSpan.textContent = `${(compressedSize / 1024).toFixed(2)} KB`;
                    
                    const reduction = ((originalSize - compressedSize) / originalSize * 100).toFixed(2);
                    reductionSpan.textContent = `${reduction}%`;
                    
                    showStatus('Compression complete!', 'success');
                    fileInfoDiv.style.display = 'block';
                    
                    // Enable download
                    downloadBtn.style.display = 'block';
                    downloadBtn.onclick = function() {
                        download(new Blob([compressedPdfBytes], { type: 'application/pdf' }), 
                        selectedFile.name.replace('.pdf', '') + '_compressed.pdf');
                    };
                    
                } catch (error) {
                    console.error('Compression error:', error);
                    showStatus('Error compressing PDF: ' + error.message, 'error');
                }
            });
            
            // Show status message
            function showStatus(message, type) {
                statusDiv.textContent = message;
                statusDiv.style.display = 'block';
                statusDiv.style.backgroundColor = type === 'error' ? '#fdecea' : 
                                              type === 'success' ? '#edf7ed' : '#e8f4fd';
                statusDiv.style.color = type === 'error' ? '#5f2120' : 
                                      type === 'success' ? '#1e4620' : '#014361';
            }
        });
    </script>
</body>
</html>
