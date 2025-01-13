---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
---


<br />

# Automating Translation File Uploads to Lokalise with GitHub Actions  

Recently, I faced the intricate challenge of automating the upload of translation files to Lokalise using GitHub Actions. This journey involved multiple trials with different base64 encoding methods and countless refinements to achieve a seamless solution. Here’s a detailed account of the problem, my efforts, and the ultimate solution.  

#### The Challenge  

Automating the upload of a translations file to Lokalise via GitHub Actions required encoding the file in base64 and sending it through a POST request. The complexity lay in correctly formatting the data while managing GitHub secrets securely.  

#### Initial Attempts  

I tried directly encoding the file content and embedding it in the JSON payload within the GitHub Actions script. Here’s a snippet of the command I used:  

```sh
 FILE_CONTENT=$(base64 file.json | tr -d '\n')

curl --request POST \
     --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/files/upload" \
     --header 'accept: application/json' \
     --header 'content-type: application/json' \
     --data '
{
  "data": '"$FILE_CONTENT"'
}
```

**<span style="color:red;">Error</span>**


```sh
Argument list too long." This error occurs when the encoded content is too large for the shell to handle directly
```


Inline Encoding in curl:
```sh
curl --request POST \
     --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/files/upload" \
     --header 'accept: application/json' \
     --header 'content-type: application/json' \
     --data '
{
  "data": '"$(base64 file.json | tr -d '\n')"'
}
```
**<span style="color:red;">Error</span>**

_Similar issue with argument length limitations and improper JSON formatting_

I tried directly encoding the file content and embedding it in the JSON payload within the GitHub Actions script. Here’s a snippet of the command I used:

```sh
BASE64_CONTENT=$(base64 -i file.json -o encoded_file.txt)

curl --request POST \
     --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/files/upload" \
     --header 'accept: application/json' \
     --header 'content-type: application/json' \
     --data '
{
  "data": '"$BASE64_CONTENT"'
}
```

However, this approach caused issues due to newline characters and incorrect escaping, leading to invalid JSON payloads




#### The Solution  

The breakthrough came by writing the base64 output directly to a file and reading it back into the script.  

I also switched from `base64` to `openssl base64`, which gave me more flexibility when working with GitHub Actions.  

Here’s the final approach that solved the issue:  

* Encode the file and set the variable:  

```sh
openssl base64 -A -in file.json -out encoded_file.json
BASE64_CONTENT=$(cat encoded_file.json)
```

* Create JSON Payload:  

```sh
json_content=$(cat EOF
{
  "data": "$BASE64_CONTENT",
  "lang_iso": "en_US",
  "filename": "index.json",
  "use_automations": true,
  "cleanup_mode": true,
  "slashn_to_linebreak": true,
  "replace_modified": true
}
EOF
)

echo "$json_content" > data.json
```


* Upload to Lokalise:  

```sh

upload_response=$(curl --request POST \
  --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/files/upload" \
  --header "X-Api-Token: ${{ secrets.LOKALISE_TOKEN }}" \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --data '@data.json')
      
```
Finally, success.

With this now working and correctly uploading the payload, I was able to add some checks to ensure I download the correct translation job.

* Checking Upload Status:  

```sh

process_id=$(echo $upload_response | jq -r '.process.process_id')

check_status() {
  status_response=$(curl --request GET \
    --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/processes/$process_id" \
    --header "X-Api-Token: ${{ secrets.LOKALISE_TOKEN }}" \
    --header "accept: application/json")

  process_status=$(echo $status_response | jq -r '.process.status')
  if [ "$process_status" = "finished" ]; then
    return 0
  else
    return 1
  fi
}

while ! check_status; do
  sleep 2
done
```

#### Conclusion  

Through persistence and numerous trials, I managed to automate the translation file uploads to Lokalise.  

The process of base64 encoding in a CI/CD pipeline revealed the importance of handling large data and JSON formatting correctly. By outputting the encoded content to a file and reading it back, I bypassed the argument length limitations and achieved a successful upload.  

For reference, this is the full GitHub Action I used:

```sh

name: Update Translations

on:
  pull_request:
    branches:
      - main


jobs:
  update-translations:
    runs-on: ubuntu-latest
    permissions:
      contents: 'write'
      id-token: 'write'
    if: ${{ startsWith(github.head_ref, 'feat') || startsWith(github.head_ref, 'Feat') }}


    steps:
      - name: Check Branch
        run: echo ${{ github.head_ref }}

      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}

      - name: Install jq
        run: sudo apt-get install -y jq
      
      - name: Upload file to Lokalise
        run: |
          openssl base64 -A -in file.json -out encoded_file.json

          echo "Set Variable"
          BASE64_CONTENT=$(cat encoded_file.json)
          echo $BASE64_CONTENT

          echo "Set JSON Content"
          json_content=$(cat EOF
          {
            "data": "$BASE64_CONTENT",
            "lang_iso": "en_US",
            "filename": "index.json",
            "use_automations": true,
            "cleanup_mode": true,
            "slashn_to_linebreak": true,
            "replace_modified": true
          }
          EOF
          )

          echo "$json_content" > data.json

          cat data.json

          upload_response=$(curl --request POST \
            --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/files/upload" \
            --header "X-Api-Token: ${{ secrets.LOCALISE_TOKEN }}" \
            --header 'accept: application/json' \
            --header 'content-type: application/json' \
            --data '@data.json')

          process_id=$(echo $upload_response | jq -r '.process.process_id')
          echo "Process ID: $process_id"

          # Function to check process status
          check_status() {
            status_response=$(curl --request GET \
              --url "https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/processes/$process_id" \
              --header "X-Api-Token: ${{ secrets.LOCALISE_TOKEN }}" \
              --header "accept: application/json")

            process_status=$(echo $status_response | jq -r '.process.status')
            echo "Current status: $process_status"

            if [ "$process_status" = "finished" ]; then
              return 0
            else
              return 1
            fi
          }

          # Loop to check status every 10 seconds until it's finished
          while ! check_status; do
            echo "Waiting for process to finish..."
            sleep 2
          done

          echo "Process finished."

      - name: Run translation update
        run: |
          DOWNLOAD_URL="https://api.lokalise.com/api2/projects/${{ secrets.LOKALISE_PROJECT_ID }}/files/download"

          # Create a temporary directory to store the download link response
          TEMP_DIR=$(mktemp -d)
          DOWNLOAD_LINK_FILE="${TEMP_DIR}/download_link.json"

          # Request the download link from Lokalise
          curl --request POST \
              --url $DOWNLOAD_URL \
              --header "X-Api-Token: ${{ secrets.LOCALISE_TOKEN }}" \
              --header 'accept: application/json' \
              --header 'content-type: application/json' \
              --data '
          {
            "filter_langs": [
              "en_GB",
              "fr",
              "de",
              "it",
              "tlh_KL",
              "pl"
            ],
            "format": "json",
            "export_empty_as": "base",
            "export_sort": "first_added",
            "plural_format": "i18next_v4",
            "indentation": "4sp",
            "original_filenames": false,
            "escape_percent": true
          }
          ' > $DOWNLOAD_LINK_FILE

          # Extract the download URL from the response
          DOWNLOAD_URL=$(jq -r '.bundle_url' $DOWNLOAD_LINK_FILE)

          # Download the translations file
          curl --output "${TEMP_DIR}/translations.zip" $DOWNLOAD_URL

          # Unzip the translations to the temporary directory
          unzip -o "${TEMP_DIR}/translations.zip" -d "${TEMP_DIR}/translations"

          # Copy the contents of the locale directory to the target translations directory
          cp -r "${TEMP_DIR}/translations/locale/." ./translations

          # Clean up the temporary directory
          rm -rf $TEMP_DIR
          rm data.json
          rm encoded_file.json
          
          # Commit changes
          git config --global user.email "platform+benefexbot@benefex.co"
          git config --global user.name "Benefex Bot"
          git pull -q
          git add ./translations
          git commit -m "Update translations from Lokalise"
          git push
```