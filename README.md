# Image to PDF Console Script

This is a browser console script that finds all images on a webpage with a `blob:` source, compiles them into a single PDF document, and initiates a download.

## Overview

The script dynamically loads the latest version of the jsPDF library to perform its functions. It is designed to be run directly in a web browser's developer console on a page where you want to archive blob images.

Key features include:
- Securely loads a modern, up-to-date version of jsPDF.
- Automatically finds all `<img>` elements sourced from a blob.
- Creates a new page for each image found.
- Resizes each image to fit an A4 page while maintaining its aspect ratio.
- Centers the image on the page.
- Saves the final document as `download.pdf`.

## How to Use

1.  **Navigate to the Target Webpage**
    Open the webpage containing the blob images you wish to save in your browser.

2.  **Copy the Script**
    Copy the entire code block below:

    ```javascript
    (function() {
        // This script will find all <img> tags with a 'blob:' src, create a PDF from them, and start a download.
        let jspdf = document.createElement("script");
        jspdf.onload = function() {
            const {
                jsPDF
            } = window.jspdf;
            const pdf = new jsPDF({
                orientation: 'p',
                unit: 'mm',
                format: 'a4'
            });

            // Get all <img> elements and filter for blob sources
            const elements = Array.from(document.getElementsByTagName("img"));
            const blobImages = elements.filter(img => /^blob:/.test(img.src));

            if (blobImages.length === 0) {
                console.log("No images with a 'blob:' source were found to create a PDF.");
                return;
            }

            blobImages.forEach((img, index) => {
                // Add a new page for each image after the first one
                if (index > 0) {
                    pdf.addPage();
                }

                try {
                    // Use a canvas to convert the image to a data URL
                    const canvasElement = document.createElement('canvas');
                    const con = canvasElement.getContext("2d");
                    
                    // Use naturalWidth/Height to get the original image dimensions
                    canvasElement.width = img.naturalWidth;
                    canvasElement.height = img.naturalHeight;
                    con.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight);
                    
                    const imgData = canvasElement.toDataURL("image/jpeg", 1.0);

                    // Calculate dimensions to fit the image on the PDF page while maintaining aspect ratio
                    const page_width = pdf.internal.pageSize.getWidth();
                    const page_height = pdf.internal.pageSize.getHeight();
                    const page_ratio = page_width / page_height;
                    const img_ratio = canvasElement.width / canvasElement.height;
                    
                    let pdf_img_width, pdf_img_height, x_offset, y_offset;

                    if (img_ratio > page_ratio) {
                        // Image is wider than the page
                        pdf_img_width = page_width;
                        pdf_img_height = page_width / img_ratio;
                    } else {
                        // Image is taller than or has the same ratio as the page
                        pdf_img_height = page_height;
                        pdf_img_width = page_height * img_ratio;
                    }
                    
                    // Center the image on the page
                    x_offset = (page_width - pdf_img_width) / 2;
                    y_offset = (page_height - pdf_img_height) / 2;

                    pdf.addImage(imgData, 'JPEG', x_offset, y_offset, pdf_img_width, pdf_img_height);

                } catch (e) {
                    console.error("Could not process image:", img.src, e);
                }
            });

            pdf.save("download.pdf");
        };

        // Use a modern, secure version of jsPDF
        jspdf.src = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js';
        document.body.appendChild(jspdf);
    })();
    ```

3.  **Open the Developer Console**
    Press `F12`, or `Ctrl+Shift+I` (Windows/Linux), or `Cmd+Opt+I` (Mac) to open the developer tools. Make sure you are on the **Console** tab.

4.  **Paste and Run**
    Paste the copied script into the console and press `Enter`. The script will run, and a download prompt for `download.pdf` should appear shortly.

## Precautions and Disclaimers

### Browser Console Security
The browser's developer console is a powerful tool intended for developers. **Never run code in your console from a source you do not fully trust.** This script is provided for a specific purpose, but you should always exercise caution when executing code that can interact with your web session.

### Website Terms of Service
Be aware that automating actions or scraping data from websites may be against their Terms of Service. Use this script responsibly and at your own risk.

### External Library
This script works by dynamically loading the `jsPDF` library from `cdnjs.cloudflare.com`, which is a reputable and widely-used CDN.

### No Warranty
This script is provided "as is", without warranty of any kind. The author is not responsible for any misuse or damage resulting from its use.
