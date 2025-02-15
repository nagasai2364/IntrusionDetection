### **Code Description**

This Python script is designed to process images uploaded to an Azure Blob Storage container using an Azure Function. It performs image classification using a pre-trained machine learning model stored in Azure Blob Storage, and sends WhatsApp alerts via Twilio depending on the classification results. Below is an overview of its functionality and structure:

---

### **1. Key Libraries and Configuration**
- **TensorFlow**: Used for loading and running the pre-trained machine learning model to classify images.
- **NumPy**: Used for numerical operations on image data.
- **Azure Storage SDK**: Interacts with Azure Blob Storage to download the machine learning model and process image uploads.
- **Twilio**: Sends WhatsApp messages to a list of recipients, notifying them about the image classification results.
- **Azure Functions SDK**: Used to create an Azure Function, which triggers when a new image is uploaded to the specified Azure Blob Storage container.

**Configuration Settings** (stored securely in Azure Application Settings):
- **Azure Blob Storage Connection String**: For accessing the blob storage.
- **Twilio Credentials**: For sending WhatsApp messages through Twilio's API.
- **Model and Blob Container Details**: For accessing the pre-trained TensorFlow model.

---

### **2. Core Functions**

#### **send_whatsapp_message**
- **Purpose**: Sends WhatsApp messages to a list of recipients using the Twilio API. 
- **Parameters**: 
  - `body`: The message content.
  - `recipients`: A list of WhatsApp numbers to send the message to.
  - `media_url`: Optional URL for attaching an image (SAS URL of the image).
- **Logic**: It uses the `twilio.rest.Client` to send the message and handles any errors that may occur during message sending.

#### **download_model_from_blob**
- **Purpose**: Downloads the pre-trained machine learning model from Azure Blob Storage.
- **Logic**: It initializes a `BlobServiceClient`, accesses the container and blob containing the model file, downloads it locally, and loads it into TensorFlow for further use.
  
#### **generate_blob_sas_url**
- **Purpose**: Generates a SAS (Shared Access Signature) URL for the uploaded image in Azure Blob Storage, allowing secure access to the image for a limited time (1 hour in this case).
- **Logic**: Uses `generate_blob_sas` from Azure SDK to generate a SAS token, which is then appended to the blob URL, enabling access for WhatsApp media attachment.

#### **prepare_image**
- **Purpose**: Prepares an image file for prediction.
- **Logic**: The image is loaded, resized to 128x128 pixels, converted to a NumPy array, rescaled, and then expanded into a batch format suitable for TensorFlow.

#### **blob_trigger_iot**
- **Purpose**: Main function triggered by the upload of a new image to the specified Azure Blob Storage container.
- **Logic**:
  1. **Blob Handling**: When a new image is uploaded, the function logs the blob details (name and size) and downloads it to a temporary file.
  2. **Image Processing**: The image is prepared for prediction by resizing and rescaling.
  3. **Model Loading & Prediction**: The pre-trained TensorFlow model is loaded, and the image is classified (as either "Resident" or "Intruder").
  4. **Send WhatsApp Alert**: Based on the classification result:
     - **Intruder**: A security alert message is sent to a list of recipients via WhatsApp, including the confidence score and timestamp.
     - **Resident**: A confirmation message is sent that no action is needed.
  5. **Cleanup**: The temporary file is removed after processing.

---

### **3. Labels and Security Alerts**
- The machine learning model predicts whether the image represents a "Resident" or an "Intruder" based on the image content. However, the labels have been swapped:
  - **Predicted Class "1"** is treated as an **Intruder**, triggering a **Security Alert** message.
  - **Predicted Class "0"** is treated as a **Resident**, triggering a simple **No Action Needed** message.
- **Security Alert** is **only triggered** for "Intruder" predictions. If the model classifies the image as a "Resident," no security alert is sent.

---

### **4. WhatsApp Message Content**
- **Security Alert for Intruder**: 
  - The message body indicates a high-priority security issue with a "Confidence" score and the time the event was detected.
  - It includes the image's SAS URL, which allows the recipients to view the image.
- **Resident Confirmation**:
  - The message is more neutral and indicates that the person detected is a resident.
  - It includes a low priority message with no further action required.

---

### **5. Execution Flow**
1. **Image Upload**: The function is triggered when a new image is uploaded to the Azure Blob Storage container
