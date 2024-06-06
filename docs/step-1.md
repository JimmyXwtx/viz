# Step-1-pdf-images

Goal: try

## Code

```
from PIL import Image  # To analyze and filter images
import fitz  # PyMuPDF
import os
import json
import shutil

# main function to extract images from PDFs
def extract_images_from_pdfs(folder_name):
    # Get the folder path, the varibale folder_name is the name of the folder containing the PDFs
    folder_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), folder_name)

    # Check if the folder exists
    if not os.path.exists(folder_path):
        raise FileNotFoundError(f"The directory {folder_path} does not exist.")


    # List to store PDFs with no images
    no_image_pdfs = []

    # Function to recursively traverse folders
    def process_folder(folder_path):
        # Traverse the folder
        for root, dirs, files in os.walk(folder_path):
            # Process each file
            for filename in files:
                if filename.endswith(".pdf"):
                    pdf_path = os.path.join(root, filename)
                    pdf_document = fitz.open(pdf_path)
                    # Check if the PDF is encrypted
                    if pdf_document.is_encrypted:
                        if not pdf_document.authenticate(""):
                            print(f"Failed to decrypt {filename}. Skipping.")
                            continue
                    # Create a folder to store the images
                    output_folder = os.path.join(root, "figures")
                    os.makedirs(output_folder, exist_ok=True)

                    print(f"Processing {filename}...")
                    any_images_extracted = False
                    # Extract images from each page
                    for page_number, page in enumerate(pdf_document, start=1):
                        image_list = page.get_images(full=True)
                        # Check if there are any images on the page
                        for image_index, img in enumerate(image_list, start=1):
                            # Extract the image from the PDF document and save it as a PNG file
                            xref = img[0]
                            base_image = pdf_document.extract_image(xref)
                            image_bytes = base_image["image"]
                            image_filename = f"page_{page_number}_extracted_image_{image_index}.png"
                            image_path = os.path.join(output_folder, image_filename)

                            with open(image_path, "wb") as img_file:
                                img_file.write(image_bytes)
                            # Check if the image can be processed by getcolors() and is large enough
                            with Image.open(image_path) as img:
                                # Ensure getcolors can process the image by setting a large enough limit
                                color_count = img.getcolors(maxcolors=10000)
                                if img.size[0] > 200 and img.size[1] > 200 and color_count and len(
                                        color_count) > 1:
                                    print(f"Saved {image_filename} to {output_folder}.")
                                    any_images_extracted = True
                                else:
                                    os.remove(image_path)
                    # Check if no images were extracted from the page, put them in a separate folder
                    if not any_images_extracted:
                        no_image_folder = os.path.join(root, "no-image")
                        os.makedirs(no_image_folder, exist_ok=True)
                        new_pdf_path = os.path.join(no_image_folder, filename)
                        shutil.move(pdf_path, new_pdf_path)
                        no_image_pdfs.append({"order": len(no_image_pdfs) + 1, "key": filename[:-4], "path": new_pdf_path})
                        print(f"No images extracted. Moved {filename} to {no_image_folder}.")

                    pdf_document.close()

    process_folder(folder_path)
    
    # Save the list of PDFs with no images to a JSON file

    with open(os.path.join(folder_path, "no_image_pdfs.json"), "w") as json_file:
        json.dump(no_image_pdfs, json_file, indent=4)



folder_name = 'ten-pdfs'
extract_images_from_pdfs(folder_name)


```


