# Document Masking and Stamping Pipeline

This project provides a Python-based pipeline for processing documents (PDFs or images). The primary functions are to automatically correct the document's orientation, detect and mask the first 8 digits of a 12-digit Aadhaar number, and finally stamp the document with new, specified details.


##  Code Flow

The execution of the notebook follows a sequential pipeline, where data from one function is passed as input to the next. The main driver is the `read_pdf_from_folder` function.

1.  **Input Reading and Decoding**

      * The process starts in **`read_pdf_from_folder`**, which iterates through `.txt` files in a specified folder.
      * It extracts a Base64 string from the file's content and decodes it into bytes (`decoded_bytes`).
      * These bytes are passed to **`get_image_and_file_type`**. This function checks if the bytes represent a PDF or an image and converts them into a list of processable images. If it's a PDF, it calls **`pdf_bytes_to_images`** to handle the conversion.

2.  **Masking and Orientation Correction**

      * Each image from the list is passed to **`masking_file`** for the core processing.
      * Inside `masking_file`, the image is first sent to **`fix_image_orientation`**.
          * This function uses **`get_rotated_image`** and an OCR engine to detect the text angle and straighten the image.
      * The now correctly-oriented image is passed to **`aadhar_check`**.
          * This function uses OCR to find 12-digit numbers.
          * Each number is validated using **`Regex_Search`** and the Verhoeff algorithm in **`compute_checksum`**.
          * If a valid Aadhaar number is found, **`helper_mask_8digits`** is called to draw a black rectangle over the first 8 digits.

3.  **Stamping Process**

      * The masked image from the previous step is passed to the **`stamping`** function, along with the original `content` from the text file.
      * **`stamping`** first calls **`stamping_details`** to parse the `content` string and extract the text that needs to be stamped.
      * It then passes the masked image and the extracted text details to **`add_text_to_image`**. This function creates a new, larger blank image, pastes the masked image onto it, and draws the new text at the bottom.

4.  **Final Output**

      * The final, stamped image is returned up the call stack. The main loop in **`read_pdf_from_folder`** receives this image and can be configured to save it to an output directory.

##  Features

  - **Multi-Format Support**: Processes both PDF and standard image files (`.jpg`, `.png`).
  - **Base64 Decoding**: Handles documents provided as Base64 encoded strings within text files.
  - **Automatic Orientation Correction**: Uses OCR and layout analysis models (`PaddleOCR`, `PaddleX`) to detect and fix the rotation of a document.
  - **Aadhaar Number Detection**: Employs OCR to find 12-digit numbers within the document text.
  - **Sensitive Data Masking**: Validates potential Aadhaar numbers using the Verhoeff algorithm and masks the first 8 digits to protect sensitive information.
  - **Dynamic Text Stamping**: Adds custom text and details onto the processed document, creating a new, stamped image.
-----

##  Tech Stack

  - **Language**: Python 3
  - **Core Libraries**:
      - **OpenCV (`cv2`)**: For core image processing and manipulation.
      - **Pillow (`PIL`)**: Used for creating new images and drawing text/stamps.
      - **NumPy**: For efficient numerical operations, especially on image arrays.
      - **PyMuPDF (`fitz`)**: For robust PDF file parsing and rendering.
  - **AI / Machine Learning**:
      - **PaddleOCR Endpoint (3.1.0)**: For Optical Character Recognition (OCR) to detect text.
      - **PaddleX**: Used with a trained model for document orientation detection.


-----
