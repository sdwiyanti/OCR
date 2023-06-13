import PyPDF2
import re
import os
import json
import zipfile
import shutil

def merge_numbering(file_name):
    with open(file_name, "rb") as file:
        reader = PyPDF2.PdfReader(file)
        num_pages = len(reader.pages)
        merged_data = []

        for page_number in range(num_pages):
            page = reader.pages[page_number]
            text = page.extract_text()

            page_content = {
                "page": page_number + 1,
                "text": {}
            }

            text = re.sub(r",(?=\n[A-Z])", ", \n", text)

            # Split line jika \n\n . ;
            pattern = r"\n \n|(?<!No)(?<!NO)(?<!Stbl)(?<!MOHD)(?<![0-9])\.|\;|(?<=Pasal [0-9]\.)|(?<=Pasal [0-9]{2}\.)|\s(?<=\(\d[0-9]\))|\s(?<!huruf )(?=\b[a-zA-Z]\.)"
            lines = re.split(pattern, text)            

            line_number = 1

            if len(lines) == 1:
                lines = re.split(r". \n|, \n", text)
                text = 'GANTIKOMADANTITIK'.join(lines)
                lines = text.split(r"GANTIKOMADANTITIK")

            for line in lines:
                line = line.strip()
                # Menghapus spasi berlebih dan tanda \n
                line = re.sub(r"\s+", " ", line.strip())

                if line:
                    page_content["text"][line_number] = line.strip()
                    line_number += 1

            merged_text = {}
            keys = list(page_content['text'].keys())
            i = 0

            while i < len(keys):
                line_number = keys[i]
                line = page_content['text'][line_number]
                merged_line = line

                words = line.split()
                last_word = words[-1]
                
                while len(last_word) == 1 and i + 1 < len(keys) and (last_word != "-" and last_word != ":"):
                    next_line_number = keys[i + 1]
                    next_line = page_content['text'][next_line_number]

                    merged_line += ". " + next_line
                    i += 1

                    words_after_merge = merged_line.split()
                    last_word_after_merge = words_after_merge[-1]
                    last_word = last_word_after_merge

                merged_text[line_number] = merged_line
                i += 1

            merged_entry = {'page': page_content['page'], 'text': merged_text}
            merged_data.append(merged_entry)

        # Mengatur ulang line_number pada akhir fungsi
        for entry in merged_data:
            text = entry['text']
            new_text = {}
            line_number = 1

            for key in sorted(text.keys()):
                new_text[line_number] = text[key]
                line_number += 1

            entry['text'] = new_text
    return merged_data


def extract_all_files(folder_path):
    file_names = os.listdir(folder_path)
    output = []

    for file_name in file_names:
        full_path = os.path.join(folder_path, file_name)

        if os.path.isdir(full_path):
            output.extend(extract_all_files(full_path))  # Rekursi: panggil fungsi untuk folder yang ada di dalamnya

        elif file_name.endswith(".zip"):
            zip_path = full_path
            extracted_files_folder = os.path.splitext(zip_path)[0]

            with zipfile.ZipFile(zip_path, 'r') as zip_ref:
                zip_ref.extractall(extracted_files_folder)

            zip_ref.close()
            os.remove(zip_path)
            print(f"File {file_name} extracted")

            output.extend(extract_all_files(extracted_files_folder))  # Rekursi: panggil fungsi untuk folder hasil ekstraksi

        elif file_name.endswith(".pdf"):
            file_content = merge_numbering(full_path)
            output.append({
                "file_name": file_name,
                "content": file_content
            })
            print(f"File {file_name} processed")

    return output

# Path folder ekstraksi yang berisi file ZIP dan PDF
folder_path = "file/tes"
initial_folder = folder_path  # Simpan folder awal
output = extract_all_files(folder_path)
print("Finished!!")

# Simpan output ke file JSON
with open('hasil/output_tes.json', 'w') as json_file:
    json.dump(output, json_file, indent=4)
