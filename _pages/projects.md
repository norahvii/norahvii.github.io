---
title: Projects
permalink: /projects/
---

## [AT2_slide_register](https://github.com/norahvii/AT2_slide_register)

Used PaddleOCR and Lebenshtein distance to name histology slides and create spreadsheets.

```python
try:
    # Phase 2: Gather the Stain ID:
    logging.info("Starting Phase 2")
    predefined_choices = ['LB509', 'HE', 'PHF-1', 'TDP-43', '10D5','pSYN'] # Stain Choices
    stain_id = None
    best_similarity = 0
    for i in text_list:
        for choice in predefined_choices:
            similarity = fuzz.ratio(i, choice)  # Calculate Lebenshtein distance
            if similarity > best_similarity:
                best_similarity = similarity
                stain_id = choice
    print(f"Stain ID: {stain_id}")
except Exception as e:
    logging.critical("Error in phase 2")
    logging.error(f"An exception occurred while processing file: {file}")
    logging.error(f"Exception details: {e} (Stain ID)")
    good = False
    move_to_failed_folder(file, failed_folder); failed_count += 1
    continue
```

## [ocr_photolab](https://github.com/norahvii/ocr_photolab)

Used Pillow to create a custom tool for cleaning up muddied documents.

```python
# apply levels adjustment to all files in the collection
def batch_levels(threshold, darkness):
    # Open resulted cropped image
    cropped_image = Image.open(target)
    # Gather cropped image's size
    width, height = cropped_image.size
    # Load all pixels from the image
    orig_pixel_map = cropped_image.load()
    # Gather mode
    mode = cropped_image.mode
    # Create a new image matching the original image's color mode, and size
    new_image = Image.new(mode, (width, height))
    # Load all the pixels from this new image as well
    new_pixel_map = new_image.load()

    new_image = Image.new(mode, (width,height))
    new_pixel_map = new_image.load()

    for x in range(width):
        for y in range(height):
            r,g,b = orig_pixel_map[x,y]
            if sum((r,g,b)) > threshold:
                new_pixel_map[x,y] = (255,255,255)
            else:
                new_pixel_map[x,y] = (r*darkness,g*darkness,b*darkness)
    # must run inside function
    new_image.save(target) # Save adjusted verison to tmp folder
```

## [xnat_tools](https://github.com/norahvii/xnat_tools)

Extracted MRI and PET data to generate timing files and uplaod those to XNAT REST API.

```python
class FileProcessor:
    def __init__(self, xnat_client, project_id, subject_group):
        self.xnat_client = xnat_client
        self.project_id = project_id
        self.subject_group = subject_group
        self.adrc_demographics = pd.read_csv('./demographic_data/all_adrc_demographics.csv', dtype={'subject': str})
        self.wrap_demographics = pd.read_csv('./demographic_data/all_wrap_demographics.csv', dtype={'subject': str})

    def process_file(self, csv_row):
        id_string = csv_row['id_string']
        full_path = csv_row['full_path']

        if full_path.lower().endswith('.json'):
            try:
                with open(full_path, 'r') as f:
                    json_content = json.load(f)

                content = self.extract_content_from_json(json_content, id_string)
                subject_label = self.get_subject_label(id_string)
                session_label = self.get_session_label(content, id_string)
                scan_id = content['scan_id']

                self.create_or_update_subject(subject_label, id_string)
                self.create_or_update_session(subject_label, session_label, content)
                self.create_or_update_scan(subject_label, session_label, scan_id, content)

                json_file_path = full_path
                nii_file_path = full_path.replace('.json', '.nii.gz')
                timing_file_path = None

                if os.path.exists(nii_file_path):
                    time_frame = [str(i + 1) for i in range(len(json_content.get('FrameDuration', '')))]
                    timing_generator = TimingFileGenerator(time_frame, json_content.get('FrameDuration', ''), json_content.get('FrameTimesEnd', ''), os.path.dirname(nii_file_path), os.path.basename(nii_file_path).split('.')[0])
                    timing_file_path = timing_generator.generate()

                self.upload_files(subject_label, session_label, scan_id, json_file_path, nii_file_path, timing_file_path)
            except Exception as e:
                logging.error(f"Error processing file {full_path}: {e}")
```

## [systematic_review](https://github.com/norahvii/systematic_review/blob/main/systematic_review.py)

Used pymed to scan PubMed to perform a systematic review of existing literature.

```python
def combine_and_save():
    # Get the current working directory
    csv_directory = os.getcwd()
    # Get a list of all CSV files in the current working directory
    csv_files = [file for file in os.listdir(csv_directory) if file.endswith('.csv')]
    # Combine all DataFrames into one
    combined_data = pd.concat([pd.read_csv(os.path.join(csv_directory, file)) for file in csv_files], ignore_index=True)
    # Remove duplicates based on all columns
    combined_data = combined_data.drop_duplicates()
    # Sort the DataFrame alphabetically by the 'title' column (adjust column name if needed)
    combined_data = combined_data.sort_values(by='title').reset_index(drop=True).iloc[:, 1:]
    # Save the combined and sorted data to a new CSV file
    combined_data.to_csv('combined_data.csv', index=False)
    print(f'Length of combined_data: {len(combined_data)}')
    # Read the combined data and remove duplicates based on 'pubmed_id'
    unduped_data = combined_data.drop_duplicates(subset='pubmed_id', keep='first')
    unduped_data = unduped_data.iloc[:, 1:]
    # Save the unduplicated data to a new CSV file
    unduped_data.to_csv('combined.csv', index=False)
    print(f'Length of unduped_data: {len(unduped_data)}')
    # Print titles from unduplicated data
    print('Titles from unduped_data:')
    for title in unduped_data['title']:
        print(title)
```

## [pet_helper](https://github.com/norahvii/pet_helper/blob/main/pet_helper.py)

Wrote a command line tool that helps calculate if an injection time was within acceptable limits.

```python
# Colors
yellow,red,green,cl = '\033[33m','\033[31m','\033[32m','\033[0m'

try:
    # Print values to console:
    print(f"MST: {mst}")
    print(f"MDT: {mdt}")
    print(f"Optimal static delay: {desired_static_delay}")
    print(f"Optimal dynamic delay: {desired_dynamic_delay}")
    # Print green if optimal
    optimal = None
    # Optimal dynamic
    if delta == int(desired_dynamic_delay):
        print(f"{delta} is{green} optimal{cl} for {selected_tracer} (dynamic: {model_state_if_dynamic}).")
        optimal = True
    # Optimal static
    if delta == int(desired_static_delay):
        print(f"{delta} is{green} optimal{cl} for {selected_tracer} (static: {model_state_if_static}).")
        optimal = True
    # Print yellow if sub-optimal
    if not optimal:
        # Sub-optimal dynamic
        if delta in range(int(dynamic_minimum), int(dynamic_maximum)+1):
            print(f"{delta} is{yellow} sub-optimal{cl} for {selected_tracer} (dynamic: {model_state_if_dynamic}).")
            if delta > int(mst):
                print(f"MST exceeded!{red} Adjust MST{cl} ({mst}), always below {dynamic_maximum}:")
            else:
                print("Requires no adjustment:")
            print(f"Dynamic minimum: {dynamic_minimum}")
            print(f"Dynamic maximum: {dynamic_maximum}")
```

Last updated: {{ "now" | date: "%B %d, %Y" }}  <!-- You can replace "now" with actual update info if desired -->
