# AWS Rekognition Image Labels Generator

This project demonstrates how to use Amazon Rekognition to detect and label objects in an image stored in an Amazon S3 bucket. The project is written in Python and uses the Boto3 library to interact with AWS services and Matplotlib to display the labeled images.

## Features

- Detect objects and labels in images using Amazon Rekognition.
- Display the image with bounding boxes drawn around detected objects.
- Show the confidence score for each detected label.

## Prerequisites

Before running this project, ensure you have the following:

- **AWS Account**: You need an active AWS account with permissions to use Rekognition and S3.
- **Amazon S3 Bucket**: You'll need an S3 bucket to store images for analysis.
- **AWS CLI**: Ensure that the AWS Command Line Interface is installed and configured.
- **Python 3.x**: Make sure you have Python 3 installed on your machine.

## Setup

### Step 1: Configure S3 Bucket

1. Log in to your AWS account.
2. Navigate to the **S3 Dashboard** and create a new S3 bucket (e.g., `aws-rekognition-image-label-project`).
3. Upload the image you want to analyze to this S3 bucket (e.g., `dog-at-the-beach.jpg`).

### Step 2: Install and Configure AWS CLI

1. Install the **AWS CLI** by following the instructions from the official [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
2. Configure the AWS CLI with your credentials by running the following command:

    ```bash
    aws configure
    ```

    Enter your AWS access key, secret key, default region, and output format when prompted.

### Step 3: Install Required Python Libraries

You need to install the following Python libraries to interact with AWS services and handle image processing:

- **Boto3**: For interacting with AWS services like S3 and Rekognition.
- **Matplotlib**: For visualizing the image and drawing bounding boxes.
- **Pillow (PIL)**: For handling images in Python.

To install these libraries, run the following commands in your terminal:

```bash
pip install boto3
pip install matplotlib
pip install Pillow
```

### Step 4: Define Functions in Your Python Script

Now, define the functions required for your project, such as `detect_labels()`, to handle image processing, AWS Rekognition calls, and image visualization.

Here's a basic example of how to define your functions:

```python
import boto3
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from PIL import Image
from io import BytesIO

def detect_labels(photo, bucket):
    # Create a Rekognition client
    client = boto3.client('rekognition')

    # Detect labels in the photo
    response = client.detect_labels(
        Image={'S3Object': {'Bucket': bucket, 'Name': photo}},
        MaxLabels=10)

    # Print detected labels
    print('Detected labels for ' + photo)
    print()
    for label in response['Labels']:
        print("Label:", label['Name'])
        print("Confidence:", label['Confidence'])
        print()

    # Load the image from S3
    s3 = boto3.resource('s3')
    obj = s3.Object(bucket, photo)
    img_data = obj.get()['Body'].read()
    img = Image.open(BytesIO(img_data))

    # Display the image with bounding boxes
    plt.imshow(img)
    ax = plt.gca()
    for label in response['Labels']:
        for instance in label.get('Instances', []):
            bbox = instance['BoundingBox']
            left = bbox['Left'] * img.width
            top = bbox['Top'] * img.height
            width = bbox['Width'] * img.width
            height = bbox['Height'] * img.height
            rect = patches.Rectangle((left, top), width, height, linewidth=1, edgecolor='r', facecolor='none')
            ax.add_patch(rect)
            label_text = label['Name'] + ' (' + str(round(label['Confidence'], 2)) + '%)'
            plt.text(left, top - 2, label_text, color='r', fontsize=8, bbox=dict(facecolor='white', alpha=0.7))
    plt.show()

    return len(response['Labels'])

def main():
    photo = 'dog-at-the-beach.jpg'
    bucket = 'aws-rekognition-image-label-project'
    label_count = detect_labels(photo, bucket)
    print("Labels detected:", label_count)

if __name__ == "__main__":
    main()
```

### Step 5: Run the Python Script

Run the Python script from your terminal:

```bash
python3 main.py
```

This will:

- Fetch the image from your specified S3 bucket.
- Analyze the image using Amazon Rekognition to detect objects and labels.
- Display the image with bounding boxes and confidence scores.

## Example Output

The script will print detected labels and their confidence scores, and display the image with bounding boxes around detected objects.

Example output in the console:

```yaml
Detected labels for dog-at-the-beach.jpg

Label: Dog
Confidence: 99.2

Label: Beach
Confidence: 96.5

Labels detected: 2
```

Additionally, the image will be displayed with red bounding boxes around the detected objects, showing the confidence scores.

## Services Used

- **Amazon S3**: Stores the images.
- **Amazon Rekognition**: Detects labels and objects in the images.
- **Matplotlib**: Displays the image and bounding boxes.